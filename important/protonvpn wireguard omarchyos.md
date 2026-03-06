# ProtonVPN WireGuard Setup on OmarchyOS (Arch + Hyprland)

A complete guide to setting up ProtonVPN via WireGuard protocol on OmarchyOS with custom connect/disconnect scripts and an optional Waybar status indicator. Includes troubleshooting for common and rare failure cases.

---

## System Context

- **OS:** OmarchyOS (Arch Linux + Hyprland)
- **Network manager:** `systemd-networkd` + `systemd-resolved`
- **DNS:** `/etc/resolv.conf` is a symlink → `/run/systemd/resolve/stub-resolv.conf`
- **Protocol:** WireGuard (kernel-native since Linux 5.6, faster and simpler than OpenVPN)
- **VPN provider:** ProtonVPN (free tier — Netherlands, Norway, Japan, Singapore, United States)

> **Config expiry:** Currently downloaded configs are valid until **Mar 6, 2027**. Re-generate and re-download from account.protonvpn.com after that date.

---

## Prerequisites

### 1. Install required packages

```bash
sudo pacman -S wireguard-tools openresolv systemd-resolvconf
```

**Why each package:**
- `wireguard-tools` — provides `wg` and `wg-quick` CLI tools
- `openresolv` — provides the `resolvconf` binary that `wg-quick` calls when setting DNS
- `systemd-resolvconf` — compatibility shim that routes `resolvconf` commands through `systemd-resolved` instead of fighting it (critical on OmarchyOS — without this, you get `signature mismatch` errors every connection)

### 2. Create a ProtonVPN account

Go to [proton.me](https://proton.me) — free tier, no credit card required.

---

## Step 1: Download WireGuard Config Files

1. Log into [account.protonvpn.com](https://account.protonvpn.com)
2. Go to **Downloads** → **WireGuard configuration**
3. Settings to select:
   - Platform: GNU/Linux
   - **VPN Accelerator: ON** (improves throughput on high-latency connections — important from Bangladesh)
   - **NAT-PMP: ON** (only needed if hosting services through VPN — enable based on your needs)
4. Download one `.conf` file per country you want. Free tier countries: NL, NO, JP, SG, US
5. Files will be named like: `wg-NL-FREE-43.conf`, `wg-JP-FREE-7.conf`, etc.

**Note on OpenVPN credentials:** ProtonVPN shows "OpenVPN username" separately from your main login. These are only for non-app connections. For WireGuard config files, you don't need these credentials — the config file contains its own keys.

---

## Step 2: Install Config Files

Save this as `vpnSetup.sh` and update the filenames to match what you downloaded:

```bash
#!/bin/bash
# Run once to install all Proton WireGuard configs

DOWNLOADS="$HOME/Downloads"

declare -A CONFIGS=(
  ["jp"]="wg-JP-FREE-7.conf"
  ["nl"]="wg-NL-FREE-43.conf"
  ["no"]="wg-NO-FREE-1.conf"
  ["sg"]="wg-SG-FREE-9.conf"
  ["us"]="wg-US-FREE-64.conf"
)

for country in "${!CONFIGS[@]}"; do
  src="$DOWNLOADS/${CONFIGS[$country]}"
  dst="/etc/wireguard/proton-$country.conf"
  if [[ -f "$src" ]]; then
    sudo cp "$src" "$dst"
    sudo chmod 600 "$dst"
    echo "✓ Installed proton-$country"
  else
    echo "✗ Not found: $src"
  fi
done

echo ""
echo "Done. Run vpnUp to connect."
```

```bash
chmod +x vpnSetup.sh
./vpnSetup.sh
```

**Why `/etc/wireguard/`:** `wg-quick` looks for configs here by default. The directory is `chmod 700` (root-only) for security — the configs contain private keys.

---

## Step 3: The vpnUp Script

This script handles connect, disconnect, country switching, status checking, and all error cases. It self-elevates to root automatically — no need to prefix with `sudo`.

```bash
#!/bin/bash
# vpnUp — Proton WireGuard VPN manager

VERSION="1.0.0"

# ─── Self-elevate (skip for --help and --version) ─────────────────────────────
if [[ "$1" != "--help" && "$1" != "-h" && "$1" != "--version" && "$1" != "-V" ]]; then
  if [[ $EUID -ne 0 ]]; then
    exec sudo "$0" "$@"
  fi
fi

# ─── Config ───────────────────────────────────────────────────────────────────

declare -A COUNTRIES=(
  ["jp"]="Japan"
  ["nl"]="Netherlands"
  ["no"]="Norway"
  ["sg"]="Singapore"
  ["us"]="United States"
)

WG_DIR="/etc/wireguard"
IP_CHECK_URL="https://ifconfig.me"
IP_CHECK_TIMEOUT=15
HANDSHAKE_TIMEOUT=10

# ─── Helpers ──────────────────────────────────────────────────────────────────

red()    { echo -e "\e[31m$*\e[0m"; }
green()  { echo -e "\e[32m$*\e[0m"; }
yellow() { echo -e "\e[33m$*\e[0m"; }
dim()    { echo -e "\e[2m$*\e[0m"; }
die()    { red "Error: $*"; exit 1; }

usage() {
  cat <<USAGE
vpnUp $VERSION — Proton WireGuard VPN manager

Usage:
  vpnUp [COUNTRY]        Connect to a country (interactive if omitted)
  vpnUp off              Disconnect from VPN
  vpnUp --status         Show current connection status
  vpnUp --list           List available countries
  vpnUp --help           Show this help text
  vpnUp --version        Show version

Country codes:
$(for c in $(echo "${!COUNTRIES[@]}" | tr ' ' '\n' | sort); do
    printf "  %-6s %s\n" "$c" "${COUNTRIES[$c]}"
  done)

Examples:
  vpnUp nl               Connect to Netherlands
  vpnUp off              Disconnect
  vpnUp --status         Check if connected and show IP
USAGE
}

check_deps() {
  local missing=()
  for cmd in wg wg-quick curl resolvconf; do
    command -v "$cmd" &>/dev/null || missing+=("$cmd")
  done
  [[ ${#missing[@]} -gt 0 ]] && die "Missing required tools: ${missing[*]}"
}

get_active_iface() {
  wg show interfaces 2>/dev/null | tr ' ' '\n' | grep '^proton-' | head -1
}

force_cleanup_iface() {
  local iface="$1"
  yellow "Cleaning up dangling interface: $iface"
  wg-quick down "$iface" 2>/dev/null
  ip link delete dev "$iface" 2>/dev/null
}

verify_handshake() {
  local iface="$1"
  local elapsed=0
  echo -n "Waiting for handshake"
  while [[ $elapsed -lt $HANDSHAKE_TIMEOUT ]]; do
    local hs
    hs=$(wg show "$iface" latest-handshakes 2>/dev/null | awk '{print $2}')
    if [[ -n "$hs" && "$hs" -gt 0 ]]; then
      echo ""; return 0
    fi
    echo -n "."; sleep 1; ((elapsed++))
  done
  echo ""; return 1
}

fix_resolv() {
  resolvconf -u 2>/dev/null || yellow "Warning: resolvconf -u failed — DNS may be stale"
}

show_current_ip() {
  echo -n "Your IP: "
  local ip
  ip=$(curl -s --max-time $IP_CHECK_TIMEOUT -4 "$IP_CHECK_URL" 2>/dev/null)
  [[ -n "$ip" ]] && green "$ip" || yellow "(IP check timed out — tunnel may still be working)"
}

status() {
  local active
  active=$(get_active_iface)
  if [[ -n "$active" ]]; then
    local code="${active#proton-}"
    local country="${COUNTRIES[$code]:-$code}"
    green "Connected → $country ($active)"
    show_current_ip
  else
    yellow "Disconnected"
  fi
}

# ─── Disconnect ───────────────────────────────────────────────────────────────

disconnect_all() {
  local active
  active=$(get_active_iface)
  if [[ -n "$active" ]]; then
    echo "Disconnecting $active..."
    wg-quick down "$active" 2>/dev/null || force_cleanup_iface "$active"
    ip link show "$active" &>/dev/null && ip link delete dev "$active" 2>/dev/null
  fi
  fix_resolv
}

# ─── Connect ──────────────────────────────────────────────────────────────────

connect_vpn() {
  local code="$1"
  local conf="$WG_DIR/proton-$code.conf"
  local iface="proton-$code"

  [[ -z "${COUNTRIES[$code]}" ]] && die "Unknown country code '$code'. Run 'vpnUp --list' for valid options."
  [[ ! -f "$conf" ]] && die "Config not found: $conf\nRun vpnSetup.sh first."

  local active
  active=$(get_active_iface)
  if [[ "$active" == "$iface" ]]; then
    yellow "Already connected to ${COUNTRIES[$code]}."
    show_current_ip; exit 0
  fi

  fix_resolv
  echo "Connecting to ${COUNTRIES[$code]}..."
  if ! wg-quick up "$iface" 2>&1; then
    red "wg-quick failed."
    force_cleanup_iface "$iface"
    fix_resolv
    die "Connection failed. Server may be down or config invalid."
  fi

  if ! verify_handshake "$iface"; then
    red "No handshake from ${COUNTRIES[$code]} within ${HANDSHAKE_TIMEOUT}s."
    force_cleanup_iface "$iface"
    fix_resolv
    die "Connection aborted — tunnel not established."
  fi

  green "Connected to ${COUNTRIES[$code]}."
  show_current_ip
}

# ─── Trap Ctrl+C ──────────────────────────────────────────────────────────────

cleanup_on_interrupt() {
  echo ""
  yellow "Interrupted. Cleaning up..."
  local active
  active=$(get_active_iface)
  [[ -n "$active" ]] && force_cleanup_iface "$active"
  fix_resolv
  exit 1
}
trap cleanup_on_interrupt INT TERM

# ─── Argument parsing ─────────────────────────────────────────────────────────

check_deps

case "${1,,}" in
  -h|--help)
    usage; exit 0 ;;
  -V|--version)
    echo "vpnUp $VERSION"; exit 0 ;;
  --status|-s)
    status; exit 0 ;;
  --list|-l)
    echo "Available servers:"
    for c in $(echo "${!COUNTRIES[@]}" | tr ' ' '\n' | sort); do
      if [[ -f "$WG_DIR/proton-$c.conf" ]]; then
        printf "  %-6s %s\n" "$c" "${COUNTRIES[$c]}"
      else
        dim "  $(printf '%-6s' $c) ${COUNTRIES[$c]} (config missing)"
      fi
    done
    exit 0 ;;
  off|down)
    disconnect_all
    green "VPN disconnected."
    exit 0 ;;
  "")
    echo ""
    echo "Available servers:"
    for c in $(echo "${!COUNTRIES[@]}" | tr ' ' '\n' | sort); do
      if [[ -f "$WG_DIR/proton-$c.conf" ]]; then
        printf "  %-6s %s\n" "$c" "${COUNTRIES[$c]}"
      else
        dim "  $(printf '%-6s' $c) ${COUNTRIES[$c]} (config missing)"
      fi
    done
    echo ""
    read -rp "Enter country code (or 'off' to disconnect): " code
    code="${code,,}"
    if [[ "$code" == "off" ]]; then
      disconnect_all
      green "VPN disconnected."
      exit 0
    fi
    disconnect_all
    connect_vpn "$code" ;;
  *)
    disconnect_all
    connect_vpn "${1,,}" ;;
esac
```

### Install and usage

```bash
# Install globally
chmod +x vpnUp.sh
sudo cp vpnUp.sh /usr/local/bin/vpnUp

# Commands
vpnUp --help           # full usage reference
vpnUp --version        # print version
vpnUp --status         # show connected country + current IP
vpnUp --list           # list available countries
vpnUp nl               # connect to Netherlands
vpnUp off              # disconnect (also: vpnUp down)
vpnUp                  # interactive menu
```

---

## Step 4: Waybar VPN Indicator (Optional)

`vpnUp --status` is sufficient for quick checks from the terminal. The Waybar indicator is useful if you want a persistent visual in your bar without typing anything.

### Status script

Save as `~/.config/waybar/scripts/vpn-status.sh`:

```bash
#!/bin/bash
# Outputs JSON for Waybar custom module

declare -A COUNTRY_NAMES=(
  ["jp"]="Japan"
  ["nl"]="Netherlands"
  ["no"]="Norway"
  ["sg"]="Singapore"
  ["us"]="United States"
)

active=""
for iface in $(wg show interfaces 2>/dev/null); do
  if [[ "$iface" == proton-* ]]; then
    active="$iface"
    break
  fi
done

if [[ -n "$active" ]]; then
  code="${active#proton-}"
  country="${COUNTRY_NAMES[$code]:-$code}"
  echo "{\"text\":\"󰦝\", \"tooltip\":\"VPN: $country\", \"class\":\"connected\"}"
else
  echo "{\"text\":\"󰦞\", \"tooltip\":\"VPN: Off\", \"class\":\"disconnected\"}"
fi
```

```bash
mkdir -p ~/.config/waybar/scripts
chmod +x ~/.config/waybar/scripts/vpn-status.sh
```

### Waybar config.jsonc

Add `"custom/vpn"` to your `modules-right` array, then add the module definition:

```json
"custom/vpn": {
    "exec": "~/.config/waybar/scripts/vpn-status.sh",
    "return-type": "json",
    "interval": 5,
    "tooltip": true
}
```

### Waybar style.css

```css
#custom-vpn {
    padding: 0 8px;
    font-size: 16px;
}

#custom-vpn.connected {
    color: #a6e3a1;
}

#custom-vpn.disconnected {
    color: #6c7086;
}
```

Restart Waybar:

```bash
killall waybar && waybar &
```

**Note:** Icons `󰦝` and `󰦞` are Nerd Font shield glyphs. OmarchyOS ships with Nerd Fonts so they render correctly.

---

## Verifying the Connection

```bash
# Quickest — uses vpnUp's built-in check
vpnUp --status

# Manual IP checks
curl -4 ifconfig.me    # IPv4
curl -6 ifconfig.me    # IPv6 (only works if the server supports it — not all free servers do)

# Inspect the active WireGuard tunnel directly
sudo wg show
```

**Note on IPv6:** Some Proton free servers (e.g. Singapore) don't assign an IPv6 address. This is normal — your connection still works over IPv4. Services that don't support IPv6 will automatically fall back to IPv4 (Happy Eyeballs algorithm).

---

## Troubleshooting

### `resolvconf: signature mismatch`

**Cause:** OmarchyOS uses `systemd-resolved` which owns `/etc/resolv.conf` as a symlink. The standalone `resolvconf` tool doesn't know about this and corrupts the signature when it tries to write DNS.

**Fix:**
```bash
sudo pacman -S systemd-resolvconf
```

This installs a shim that routes all `resolvconf` calls through `systemd-resolved`. Permanent fix, survives reboots.

**If it still happens once after installing:** Run `sudo resolvconf -u` manually once to reset state, then it should work cleanly going forward.

---

### `config missing` shown for all countries

**Cause:** `/etc/wireguard/` is `chmod 700` — only root can see inside it. The script self-elevates via `exec sudo "$0" "$@"` so this should be transparent, but if you're running an old version of the script or running it in an unusual way, file checks may silently fail.

**Fix:** Make sure you're using the latest `vpnUp` installed at `/usr/local/bin/vpnUp`.

**Verify configs are actually there:**
```bash
sudo ls /etc/wireguard/
```

---

### Connection fails first attempt, works on second

**Cause:** Race condition between `wg-quick down` running `resolvconf -d` on disconnect, and systemd writing `/etc/resolv.conf` again before the next connect.

**Fix:** The script calls `fix_resolv` (which runs `resolvconf -u`) both after disconnect AND immediately before `wg-quick up`. This double-clean ensures the state is always fresh. Make sure you're using the latest version.

---

### `wg-quick failed` — interface created but no tunnel

**Cause:** `wg-quick up` created the network interface but then bailed (usually DNS-related). The interface is left dangling.

**Fix:** The script handles this automatically — `force_cleanup_iface` removes the ghost interface and calls `fix_resolv`. If it persists manually:

```bash
sudo ip link delete dev proton-nl
sudo resolvconf -u
```

---

### Handshake timeout — tunnel not established

**Cause:** The Proton server is unreachable or overloaded. The tunnel interface comes up but no cryptographic handshake completes within 10 seconds.

**What to try:**
1. Try a different country: `vpnUp sg` or `vpnUp us`
2. Check if Proton is having an outage (check r/ProtonVPN)
3. Check your base internet: `curl -4 ifconfig.me` without VPN should return your real IP

---

### IP check times out after connecting

**Cause:** `ifconfig.me` can be slow to respond right after a VPN tunnel comes up, especially from high-latency locations like Bangladesh.

**Fix:** Already handled in the script with a 15-second timeout. If it still shows `(IP check timed out)`, manually verify:
```bash
vpnUp --status
# or
curl -4 ifconfig.me
```

---

### No internet after disconnecting

**Cause:** DNS wasn't restored properly after `wg-quick down`.

**Fix:**
```bash
sudo resolvconf -u
```

If that doesn't work:
```bash
sudo systemctl restart systemd-resolved
```

---

### `resolvconf: Cannot write to /run/resolvconf/lock`

**Cause:** You ran `resolvconf -u` without sudo. It needs root.

**Fix:** Always use `sudo resolvconf -u`.

---

## Rare / Future-Proofing Cases

### OmarchyOS changes DNS management

OmarchyOS currently uses `systemd-networkd` + `systemd-resolved`. If a future version switches to NetworkManager:

**How to detect:**
```bash
systemctl status NetworkManager systemd-networkd 2>/dev/null | grep -E "●|Active"
ls -la /etc/resolv.conf
```

**If NetworkManager is now running:**
- Install `networkmanager-openvpn` or use `nmcli` to import WireGuard configs instead of `wg-quick`
- Or keep using `wg-quick` but install `networkmanager` integration: `sudo pacman -S networkmanager`
- Uninstall `openresolv` and `systemd-resolvconf`, install `networkmanager` resolvconf integration

**If `/etc/resolv.conf` is no longer a symlink:**
```bash
ls -la /etc/resolv.conf
# If it's a real file, resolvconf conflicts are gone — remove fix_resolv calls from vpnUp
```

---

### Proton changes their config file format

Proton's WireGuard configs follow the standard WireGuard INI format. If future configs look different, check:

```bash
sudo cat /etc/wireguard/proton-nl.conf
```

Expected structure:
```ini
[Interface]
PrivateKey = <key>
Address = 10.x.x.x/32, 2a07:xxxx::x/128
DNS = 10.x.x.x, 2a07:xxxx::x

[Peer]
PublicKey = <key>
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = <ip>:51820
PersistentKeepalive = 25
```

If the format changes significantly, re-download fresh configs from account.protonvpn.com and re-run `vpnSetup.sh`.

---

### WireGuard kernel module missing after kernel update

Arch updates the kernel frequently. After a kernel update, WireGuard (built-in since 5.6) should still work, but if it doesn't:

```bash
uname -r                  # check kernel version
modinfo wireguard         # check if module is available
sudo modprobe wireguard   # load it manually if needed
lsmod | grep wireguard    # confirm it loaded
```

If `modinfo wireguard` fails, the kernel headers may be mismatched. Fix:
```bash
sudo pacman -S linux linux-headers
sudo reboot
```

---

### Proton free server IPs change

Proton rotates server IPs periodically. If a config suddenly stops connecting (handshake timeout every time):

1. Re-download fresh config files from account.protonvpn.com
2. Re-run `vpnSetup.sh` to overwrite old configs

---

### Private key compromised or config corrupted

Generate a fresh config:
1. Log into account.protonvpn.com
2. Delete the old WireGuard key under **Account → WireGuard keys**
3. Download fresh config files
4. Re-run `vpnSetup.sh`

---

## Notes on Routing and Latency

Latency varies significantly by your location:
- Pick the geographically closest server for lowest latency
- EU servers (NL, NO) will have higher latency from Asia
- Asia servers (JP, SG) will be faster from South/Southeast Asia
- Use `vpnUp --status` to verify your IP after connecting

---

## File Locations Reference

| File | Location | Purpose |
|------|----------|---------|
| WireGuard configs | `/etc/wireguard/proton-*.conf` | Per-country VPN configs |
| vpnUp script | `/usr/local/bin/vpnUp` | Main VPN control script |
| vpnSetup script | `~/Downloads/vpnSetup.sh` | One-time install script |
| Waybar script (optional) | `~/.config/waybar/scripts/vpn-status.sh` | Waybar status indicator backend |
| Waybar config (optional) | `~/.config/waybar/config.jsonc` | Add `custom/vpn` module here |
| Waybar CSS (optional) | `~/.config/waybar/style.css` | Add `#custom-vpn` styles here |
