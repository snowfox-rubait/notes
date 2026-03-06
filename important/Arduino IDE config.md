### Arduino IDE Setup

> Do this after installing `arduino-ide-bin` from AUR and `arduino-cli` from normal packages.

**Step 1 — Fix the .desktop exec flags (required for Wayland):**

```bash
sudo nano /usr/share/applications/arduino-ide-v2.desktop
```

If that path doesn't exist, check `/usr/share/applications/` for the correct filename.

Change:
```
Exec=arduino-ide
```
To:
```
Exec=arduino-ide --ozone-platform=x11 --force-device-scale-factor=1
```

**Step 2 — Add your user to required groups (for serial port access):**

```bash
sudo usermod -aG uucp $USER
sudo usermod -aG lock $USER
```

> Log out and back in after this — group changes don't take effect until next login.

**Step 3 — Add board manager URLs:**

Open Arduino IDE → Preferences → paste into the "Additional boards manager URLs" field:

```
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json, http://arduino.esp8266.com/stable/package_esp8266com_index.json
```

**Step 4 — Install boards and libraries** from the Boards Manager (e.g. ESP32, ESP8266).

---
