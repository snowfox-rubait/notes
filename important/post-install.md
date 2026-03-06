# Post-Install Checklist (Omarchy / Arch + Hyprland)

> This is the canonical single-file version. Track all future changes with git.

---

## Changelog

### v5 — Philosophy update
**Updated:**
- Default editor changed from Sublime Text to Neovim (`nvim`)
- Caps Lock config corrected to `compose:ralt,caps:swapescape` (reflects actual setup)
- Chromium account sync kept with migration note to Firefox/LibreWolf
- Browser section updated with honest notes on each option's cons
- JetBrains IDEs kept but flagged — only use if forced by client/project
- VSCode kept with note: remove after fully mastering Neovim
- AI tools trimmed to Gemini and Claude only — local LLM tools removed (hardware too weak)
- Ollama, LM Studio, Cursor CLI, Copilot CLI removed
- Kaspersky Password Manager kept until expiry — migration to Vaultwarden planned
- `dotslash` and `rsync` TODO note removed (was useless, no context)

### v4 — Merged canonical (from v1 + v2)
**Added:**
- `firefox / librefox / waterfox` browser option (from v2)
- Descriptions for every package explaining what it does and why
- Arduino IDE setup section fully restored (was dropped in v3)
- `arduino-cli` and `arduino-ide-bin` restored (from v1)
- `freetube` kept in AUR (from v1)
- Logout reminder added after `usermod` commands
- Wallpaper section split into easy vs manual method
- Shortcuts converted to a table
- Git setup section added
- Section numbers added for easier navigation

**Removed:**
- v3 discarded entirely — was missing Arduino and freetube with no documented reason
- Redundant phrasing and inconsistent formatting cleaned up throughout

---

## 1. Set Theme

Change theme to **Osaka Jade**.

---

## 2. Remove Packages

### Normal packages
`1password-beta`, `1password-cli`, `1password`, `alacritty`, `typora`, `xournalpp`, `spotify`, `signal-desktop`, `whois`

### Web Apps
Basecamp, Fizzy, Google ecosystem, HEY, X, Zoom, YouTube

> Remove related keybindings after uninstalling.

---

## 3. Install Packages

### Normal packages
- `ddcutil` — monitor brightness/DDC control (desktop PC only)
- `ntfs-3g` — NTFS read/write support for Windows drives
- `nano` — simple fallback terminal text editor
- `vlc` — media player
- `arduino-cli` — Arduino command-line tooling

### AUR packages
- `clion` — C/C++ IDE *(only use if client requires it or you are forced to)*
- `arduino-ide-bin` — Arduino GUI IDE
- `freetube` — FOSS privacy-respecting YouTube frontend

### IDEs
> Only use these if your client requires it or you are forced to. Prefer Neovim for everything else.
- `pycharm` — Python IDE
- `intellij-idea` — Java/Kotlin IDE

### Browser
Pick one. Honest notes on each:

- `firefox` — good default, but Mozilla has been making questionable decisions lately (Mr. Tech partnership, ad-measurement API). Still one of the better options.
- `librewolf` — Firefox fork with tracking protection and privacy settings hardened out of the box. Best choice if you want FOSS-aligned browsing without manual config.
- `waterfox` — had ownership controversy (was sold to System1, an ad-tech company, then bought back). Not recommended.

### Web Apps
- Kaspersky Password Manager *(currently paid up — migrate to Vaultwarden after expiry)*

### Services
- Tailscale
- Chromium account sync *(switch to Firefox/LibreWolf sync as soon as possible — you've used Chrome for 5 years and profiles have everything configured, so this transition will take time, but it should be done)*

### Editor
- Neovim with LazyVim *(primary editor)*
- VSCode *(keep until you fully master Neovim to the point you can build your own config — remove after that)*

### AI Tools
> Local LLM tools removed — hardware is too weak to run them usefully. Cloud only for now.
- Gemini
- Claude

### Gaming
- Steam *(special install)*

> Set keybindings for daily-use apps after installing everything.


---

## 4. Config Changes

> Read the Omarchy manual before starting. Many of these changes have relevant context there.

---

### Set Default Editor to Neovim

File: `~/.config/uwsm/default`

Find the `EDITOR` line and change it to:

```sh
export EDITOR=nvim  # neovim
```

---

### Move Waybar to Bottom

File: `~/.config/waybar/config.jsonc`

Replace:
```json
"layer": "top",
"position": "top",
```
With:
```json
"layer": "top",
"position": "bottom",
```

---

### Change Clock to 12-Hour Format

File: `~/.config/waybar/config.jsonc`

Replace:
```json
"clock": {
  "format": "{:%A %H:%M}",
```
With:
```json
"clock": {
  "format": "{:%A %I:%M %p}",
```

---

### VPN (ProtonVPN + WireGuard)

See `protonvpn-wireguard-omarchyos.md` for full setup.

- Configs already downloaded — valid until **Mar 6, 2027**
- Run `vpnSetup.sh` once to install them into `/etc/wireguard/`
- Then use `vpnUp.sh` to connect
- Add it to `$PATH` so you don't have to specify the whole directory

---

### Monitor Brightness & Contrast Control (Waybar)

See `monitor-control-omarchyos.md` for full setup.

- Requires `ddcutil` (already installed above)
- Add your user to the i2c group: `sudo usermod -aG i2c $USER` then log out and back in
- Place `monitor-status.sh` and `monitor-set.sh` in `~/.config/waybar/scripts/`
- Add `custom/monitor` module to `config.jsonc` and styles to `style.css`
- Click the icon in the tray drawer to set brightness/contrast via floating terminal

---

### Fix Caps Lock Key

File: `~/.config/hypr/input.conf`

Find the `kb_options` line and change it to:

```
kb_options = compose:ralt,caps:swapescape
```

This remaps Caps Lock to Escape (essential for Vim), and sets Right Alt as the Compose key.

---

### Remove Window Gaps and Round Corners

File: `~/.config/hypr/looknfeel.conf`

```
general {
    gaps_in = 0
    gaps_out = 0
    border_size = 0
}

decoration {
    rounding = 8
}
```

---

### Change Wallpaper

**Easy method:** Open Omarchy menu (`Super + Alt + Space`) → Install → Style → Background → Enter, then copy your wallpaper image into that folder.

**Manual method:** Copy your image directly to `~/.config/omarchy/[theme-name]/backgrounds/`

Toggle between wallpapers with `Super + Ctrl + Space`.

---

### Disable Auto Sleep and Screen Lock

File: `~/.config/hypr/hypridle.conf`

Replace the entire file content with:

```
general {
    lock_cmd = ""              # disables automatic lock
    before_sleep_cmd = ""      # disables pre-suspend lock
    after_sleep_cmd = ""       # disables DPMS-on after sleep
    inhibit_sleep = 0          # no inhibit wait
}

# Uncomment below to re-enable sleep/lock behavior:

# listener {
#     timeout = 150
#     on-timeout = pidof hyprlock || omarchy-launch-screensaver
# }

# listener {
#     timeout = 151
#     on-timeout = loginctl lock-session
# }

# listener {
#     timeout = 330
#     on-timeout = hyprctl dispatch dpms off
#     on-resume = hyprctl dispatch dpms on && brightnessctl -r
# }
```

---

### Monitor Configuration

File: `~/.config/hypr/monitors.conf`

Edit as needed for your specific display setup.

---

### Arduino IDE config

- `arduino-ide-bin` — Arduino GUI IDE *(see `arduino-ide-setup.md` for post-install configuration)*

---

### Add LocalSend Keybinding

File: `~/.config/hypr/bindings.conf`

Add this line at the bottom of the `# Application bindings` section:

```
bindd = SUPER SHIFT, L, LocalSend, exec, localsend
```

---

## 5. Useful Shortcuts and Tips

> Also check the Omarchy Manual sections: **Shell Tools**, **Terminal**, and **HotKeys**.

| Action | Shortcut / Command |
|---|---|
| Close all windows | `Ctrl + Alt + Del` |
| Toggle wallpaper | `Super + Ctrl + Space` |
| Open Omarchy menu | `Super + Alt + Space` |
| Navigate file browser | `Alt + Arrow` (instead of Backspace) |
| List all installed packages | `sudo pacman -Qq` |

---

## 6. Git Setup for This File

Run this once after saving the file:

```bash
mkdir -p ~/notes
mv post-install.md ~/notes/
cd ~/notes
git init
git add post-install.md
git commit -m "init: add post-install checklist (v5 canonical)"
```

After every meaningful edit:

```bash
git add post-install.md
git commit -m "update: removed vscode after nvim mastery"
```

Commit message conventions used in this repo:
- `init:` — first commit
- `add:` — new content added
- `remove:` — content removed
- `update:` — existing content changed
- `fix:` — corrected something wrong
