# Monitor Brightness & Contrast Control on OmarchyOS

A Waybar indicator for DDC/CI monitor control via `ddcutil`. Click the icon to open a floating terminal prompt for setting brightness and contrast. Hover to see current values.

---

## System Context

- **OS:** OmarchyOS (Arch Linux + Hyprland)
- **Tool:** `ddcutil` — communicates with monitor over DDC/CI (the I2C bus built into every modern monitor)
- **Interface:** Waybar `custom/monitor` module inside the tray drawer
- **Signal:** `SIGRTMIN+11` (signals 7–10 are taken by other OmarchyOS modules)

---

## Prerequisites

### 1. Install ddcutil

```bash
sudo pacman -S ddcutil
```

### 2. Verify your monitor is detected

```bash
sudo ddcutil detect
```

Should show your monitor model. If it returns nothing, DDC/CI may be disabled in your monitor's OSD menu — enable it there first.

### 3. Add your user to the i2c group

```bash
sudo usermod -aG i2c $USER
```

Log out and back in. After this, `ddcutil` works without `sudo`.

---

## VCP Codes Used

| Code | Feature |
|------|---------|
| `10` | Brightness |
| `12` | Contrast |

Verify these work on your monitor:
```bash
ddcutil getvcp 10
ddcutil getvcp 12
```

---

## Scripts

Place both scripts in `~/.config/waybar/scripts/` and make them executable:

```bash
chmod +x ~/.config/waybar/scripts/monitor-status.sh
chmod +x ~/.config/waybar/scripts/monitor-set.sh
```

### monitor-status.sh

Outputs JSON for the Waybar module. Called once on Waybar start, then only refreshed via signal after values are set.

```bash
#!/bin/bash
# ~/.config/waybar/scripts/monitor-status.sh

brightness=$(ddcutil getvcp 10 2>/dev/null | grep -oP 'current value =\s*\K[0-9]+')
contrast=$(ddcutil getvcp 12 2>/dev/null | grep -oP 'current value =\s*\K[0-9]+')

if [[ -z "$brightness" || -z "$contrast" ]]; then
  echo '{"text": "󰍹", "tooltip": "Monitor: unavailable", "class": "unavailable"}'
else
  echo "{\"text\": \"󰍹\", \"tooltip\": \"Brightness: ${brightness}%\nContrast: ${contrast}%\", \"class\": \"active\"}"
fi
```

### monitor-set.sh

Runs in a floating terminal. Shows current values, prompts for new ones. Empty Enter skips that value. Fires signal to refresh the tooltip after applying.

```bash
#!/bin/bash
# ~/.config/waybar/scripts/monitor-set.sh

current_brightness=$(ddcutil getvcp 10 2>/dev/null | grep -oP 'current value =\s*\K[0-9]+')
current_contrast=$(ddcutil getvcp 12 2>/dev/null | grep -oP 'current value =\s*\K[0-9]+')

read -rp "Brightness (current: ${current_brightness:-?}, 0-100, Enter to skip): " brightness
read -rp "Contrast   (current: ${current_contrast:-?}, 0-100, Enter to skip): " contrast

if [[ -n "$brightness" && "$brightness" =~ ^[0-9]+$ && "$brightness" -le 100 ]]; then
  ddcutil setvcp 10 "$brightness"
fi

if [[ -n "$contrast" && "$contrast" =~ ^[0-9]+$ && "$contrast" -le 100 ]]; then
  ddcutil setvcp 12 "$contrast"
fi

pkill -SIGRTMIN+11 waybar
```

---

## Waybar Integration

### config.jsonc — module definition

```json
"custom/monitor": {
    "exec": "$HOME/.config/waybar/scripts/monitor-status.sh",
    "return-type": "json",
    "interval": 3600,
    "signal": 11,
    "on-click": "omarchy-launch-floating-terminal-with-presentation $HOME/.config/waybar/scripts/monitor-set.sh",
    "tooltip": true
}
```

### config.jsonc — add to tray drawer

```json
"group/tray-expander": {
    "modules": ["custom/expand-icon", "tray", "custom/monitor", "custom/vpn"]
}
```

**Note:** Use `$HOME` not `~` in Waybar config — Waybar does not expand `~`.

### style.css — add to bottom

```css
#custom-monitor,
#custom-vpn {
  min-width: 12px;
  margin: 0 7.5px;
}

#custom-monitor.unavailable {
  color: #6c7086;
}
```

Restart Waybar after any config changes:
```bash
killall waybar && waybar &
```

---

## How It Works

- **Hover** → tooltip shows current brightness and contrast (fetched at Waybar startup, refreshed after every set)
- **Click** → OmarchyOS floating terminal opens running `monitor-set.sh`
- **Enter value or skip** → empty Enter keeps the current value, valid 0–100 applies it
- **After setting** → `pkill -SIGRTMIN+11 waybar` fires, Waybar re-runs `monitor-status.sh` immediately and updates the tooltip

**Why `interval: 3600`:** `ddcutil getvcp` takes ~1 second per call over I2C. Polling every few seconds would be wasteful. Instead the tooltip only updates on demand via the signal, which fires automatically after every set.

---

## Troubleshooting

### `Monitor: unavailable` in tooltip

**Cause:** `ddcutil` can't communicate with the monitor.

**Check:**
```bash
ddcutil detect
ddcutil getvcp 10
```

**Common fixes:**
- Enable DDC/CI in your monitor's OSD menu
- Make sure your user is in the `i2c` group: `groups $USER`
- If running in a VM with GPU passthrough, DDC/CI may not be passed through — test from the host directly

---

### Click opens terminal but ddcutil is slow

`ddcutil` queries the monitor over I2C which takes ~1s per call. This is normal. The script fetches current values at open time so there's a ~2 second delay before the prompts appear.

---

### Tooltip doesn't update after setting values

`pkill -SIGRTMIN+11 waybar` at the end of `monitor-set.sh` triggers the refresh. If it's not updating:

```bash
# Test the signal manually
pkill -SIGRTMIN+11 waybar
```

If that doesn't work, check that signal 11 isn't already used by another module in your config.

---

### VCP codes 10/12 don't work on your monitor

Some monitors use non-standard VCP codes. Find the right ones:

```bash
ddcutil capabilities | grep -A2 "Feature: 10\|Feature: 12"
# Or dump all features
ddcutil getvcp known
```

Update the codes in both scripts accordingly.

---

## File Locations Reference

| File | Location |
|------|----------|
| Status script | `~/.config/waybar/scripts/monitor-status.sh` |
| Set script | `~/.config/waybar/scripts/monitor-set.sh` |
| Waybar config | `~/.config/waybar/config.jsonc` |
| Waybar styles | `~/.config/waybar/style.css` |
