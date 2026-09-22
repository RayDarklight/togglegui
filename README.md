# togglegui

A simple bash script to switch your Linux desktop between GUI and headless mode on demand — useful for saving power or enabling remote-only workflows via SSH.

When switched to headless mode, the display manager is stopped and the screen backlight is turned off, dropping CPU/power usage significantly. Docker is kept running in the background in both modes.

---

## Features

- Toggle GUI on/off with a single command
- Saves and restores backlight brightness automatically
- Keeps Docker running in both modes
- Logs all actions to `/var/log/togglegui.log`
- Tracks current state for quick status checks

---

## Requirements

| Requirement | Notes |
|---|---|
| `bash` | Script interpreter |
| `systemd` | Used to manage the display manager and Docker |
| `ethtool` | Optional — only needed if using alongside WOL setup |
| `sudo` | Required for display manager and backlight control |
| Intel backlight | Expects `/sys/class/backlight/intel_backlight` — AMD systems may need to adjust this path |
| Display manager | Any systemd-managed display manager (SDDM, GDM, LightDM, etc.) |
| Docker (optional) | If not installed, Docker-related lines fail silently |

---

## Installation

```bash
# Clone the repo
git clone https://github.com/RayDarklight/togglegui.git
cd togglegui

# Make the script executable
chmod +x togglegui

# Move to PATH
sudo mv togglegui /usr/local/bin/togglegui
```

---

## Usage

```bash
# Toggle (auto-detects current state)
togglegui

# Explicitly turn GUI off (enter headless mode)
togglegui off

# Explicitly turn GUI on
togglegui on

# Check current mode
togglegui status
```

### Output

```
togglegui status
# GUI
# or
# HEADLESS
# or
# Unknown   (before first toggle after reboot)
```

---

## How It Works

**GUI Off (Headless mode)**
1. Saves current backlight brightness to `/tmp/.backlight_brightness`
2. Sets backlight to 0 (display off)
3. Stops the display manager via `systemctl`
4. Restarts Docker to ensure it stays healthy
5. Writes `HEADLESS` to `/tmp/.gui_state`

**GUI On**
1. Starts the display manager via `systemctl`
2. Restarts Docker
3. Restores saved backlight brightness
4. Writes `GUI` to `/tmp/.gui_state`

**Auto-toggle**
If no argument is passed, the script checks whether the display manager is currently active and switches to the opposite state.

---

## Logs

All actions are logged with timestamps to:

```
/var/log/togglegui.log
```

---

## Notes

- The state file `/tmp/.gui_state` is cleared on reboot (lives in `/tmp`), but the toggle logic uses `systemctl is-active` to detect state, so the script works correctly regardless.
- AMD GPU users should replace `intel_backlight` with the appropriate path under `/sys/class/backlight/` for their system.

---

## Revision History

| Version | Notes |
|---|---|
| r1 | Initial implementation — GUI toggle, backlight control, Docker exception, status check |

---

## Author

[RayDarklight](https://github.com/RayDarklight)
