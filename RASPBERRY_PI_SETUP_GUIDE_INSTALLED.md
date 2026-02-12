# Raspberry Pi Smart Calendar Kiosk Setup Guide

A complete guide to building a DIY Skylight-style family calendar using a Raspberry Pi 4, external touchscreen, and Home Assistant.

---

## Time Estimate

| Phase | Task | Estimated Time |
|-------|------|----------------|
| **Phase 1** | Download Raspberry Pi Imager & flash SD card | 15-20 min |
| | First boot & system updates | 20-30 min |
| **Phase 2** | Touchscreen hardware connection | 10-15 min |
| **Phase 3** | Install packages & configure kiosk mode | 20-30 min |
| **Phase 4** | HACS installation | 10-15 min |
| | Install HACS frontend components (7 required + 2 recommended) | 15-20 min |
| | Configure packages, calendars, dashboard | 30-45 min |
| | Theme setup | 5-10 min |
| **Phase 5** | Final configuration & testing | 10-15 min |
| | **TOTAL** | **2.5 - 3.5 hours** |

### Factors That Affect Timing

**Faster if:**
- You've worked with Raspberry Pi before
- Using local calendars instead of Google Calendar
- Fast internet connection (downloads, updates)

**Slower if:**
- First time with Raspberry Pi/Linux
- Setting up Google Calendar integration (OAuth setup)
- Troubleshooting display/touch issues
- Customizing the dashboard heavily for your family
- Using a non-standard display that needs driver work

**Recommendation:** Set aside **half a day** for a comfortable pace with breaks for troubleshooting.

---

## Parts List

### Required Components

| Component | Notes | Approx. Cost |
|-----------|-------|--------------|
| **Raspberry Pi 4 Model B (2GB+)** | Smooth, responsive performance | $45-75 |
| **MicroSD Card** | 32GB+ Class 10 recommended | $10-15 |
| **Power Supply** | 5V 3A **USB-C** (official recommended) | $10-15 |
| **Touchscreen Display** | See options below | $50-150 |
| **Micro-HDMI to HDMI Cable** | Pi 4 uses micro-HDMI (2 ports available) | $8-12 |
| **MicroSD Card Reader** | For flashing OS | $5-10 |

### Recommended Display

**15" HDMI Touchscreen**
- For countertop display similar to original Skylight
- USB for touch, HDMI for display
- Requires external power

### Optional Components

| Component | Purpose |
|-----------|---------|
| **Case/Stand** | Countertop mounting |
| **Heatsinks** | Thermal management (recommended) |
| **Fan/Active Cooling** | Recommended for enclosed case |
| **USB Keyboard/Mouse** | Initial setup only |

---

## Phase 1: Raspberry Pi OS Setup
**Estimated time: 35-50 minutes**

### Step 1: Download Raspberry Pi Imager
**5 minutes**

1. Download from [raspberrypi.com/software](https://www.raspberrypi.com/software/)
2. Install on your computer

### Step 2: Flash the SD Card
**10-15 minutes**

1. Insert MicroSD card into your computer
2. Open Raspberry Pi Imager
3. Click **Choose Device** → Select **Raspberry Pi 4**
4. Click **Choose OS** → **Raspberry Pi OS (64-bit)** with Desktop
5. Click **Choose Storage** → Select your SD card
6. Click **Next**, then **Edit Settings**:

   **General tab:**
   - Set hostname: `calendar-kiosk`
   - Set username/password
   - Configure Wi-Fi (SSID and password)

   **Services tab:**
   - Enable SSH
   - Choose authentication method:
     - **Password authentication** (easier): Use the password you set in General tab
     - **Public-key authentication** (more secure): See [Optional: SSH Public-Key Setup](#optional-ssh-public-key-setup) below
   - Enable **Raspberry Pi Connect** (allows remote access from anywhere via Raspberry Pi's cloud service)

7. Click **Save** → **Yes** to apply settings → **Yes** to flash

> **Note on Raspberry Pi Connect:** After first boot, you'll need to sign in with a Raspberry Pi ID to complete Connect setup. Visit [connect.raspberrypi.com](https://connect.raspberrypi.com) to access your Pi remotely.

### Step 3: First Boot
**2-3 minutes**

1. Insert SD card into Raspberry Pi
2. Connect keyboard, mouse, and monitor temporarily
3. Connect USB-C power supply
4. Wait for initial boot and setup (2-3 minutes)

### Step 4: Complete Raspberry Pi Connect Setup

After first boot, link your Pi to your [Raspberry Pi ID](https://id.raspberrypi.com/) to enable remote access. If you don't have a Raspberry Pi ID yet, [create one here](https://id.raspberrypi.com/sign-up).

The sign-in command generates a URL with a short-lived authorization code. If the link expires before you complete authorization, re-run the command to get a fresh one.

```bash
# Install rpi-connect if not already present
sudo apt install rpi-connect -y

# Start the sign-in process (generates a one-time auth URL)
rpi-connect signin
```

Open the printed URL in a browser on any device, sign in with your Raspberry Pi ID, and authorize the device.

```bash
# Verify Connect is running
systemctl --user status rpi-connect

# Ensure it starts automatically on boot
systemctl --user enable rpi-connect
```

Once linked, you can access your Pi remotely at [connect.raspberrypi.com](https://connect.raspberrypi.com).

> **Reference:** See the official [Raspberry Pi Connect documentation](https://www.raspberrypi.com/documentation/services/connect.html) for more details on setup and troubleshooting.

### Step 5: Update the System
**15-30 minutes** (depends on internet speed)

Open Terminal and run:
```bash
sudo apt update
sudo apt upgrade -y
```

---

## Phase 2: Touchscreen Hardware Setup
**Estimated time: 10-15 minutes**

1. Connect **Micro-HDMI to HDMI cable** from display to Pi 4's **HDMI0** port (closest to USB-C power)
2. Connect **USB cable** from display to Pi (for touch) - use USB 2.0 port
3. Power on the display and Pi
4. Touch should work automatically on modern Raspberry Pi OS

> **Note:** The Pi 4 has two micro-HDMI ports. Use **HDMI0** (the one closest to the power connector) for primary display.

### Rotate Display (if needed)
```bash
# For desktop environment
# Menu → Preferences → Screen Configuration
# Right-click display → Orientation → Choose rotation
```

---

## Phase 3: Kiosk Mode Setup
**Estimated time: 20-30 minutes**

### Step 1: Install Required Packages
**5-10 minutes**

```bash
sudo apt install xserver-xorg lightdm onboard chromium sed -y
```

**What each package does:**
| Package | Purpose |
|---------|---------|
| `xserver-xorg` | X Window System server — provides the graphical display layer that renders the desktop |
| `lightdm` | Display manager — handles the login screen and enables auto-login on boot |
| `onboard` | On-screen virtual keyboard for touchscreen text input |
| `chromium` | Web browser that displays the Home Assistant dashboard in kiosk mode |
| `sed` | Stream editor — used here to modify LightDM config files from the command line |

### Step 2: Configure Auto-Login
**2 minutes**

This configures LightDM to boot directly to the desktop without showing a login prompt — essential for a kiosk that should start unattended.

> **`YOUR_USERNAME`** is the username you set in **Phase 1, Step 2** (the username configured in Raspberry Pi Imager's General tab when flashing the SD card).

```bash
# Enable graphical boot
sudo systemctl --quiet set-default graphical.target

# Set auto-login (replace YOUR_USERNAME with your actual username)
sudo sed /etc/lightdm/lightdm.conf -i -e "s/^\(#\|\)autologin-user=.*/autologin-user=YOUR_USERNAME/"
```

### Step 3: Set Screen Resolution
**2 minutes**

This tells LightDM to run `xrandr -s <resolution>` at login time, which sets the display resolution before Chromium launches. This ensures the display matches the dashboard's designed resolution (the `--window-size` in the kiosk script should match this value).

```bash
# Set your display resolution (e.g., 1920x1080):
RESOLUTION=1920x1080
sudo sed /etc/lightdm/lightdm.conf -i -e "s/^\(#\|\)display-setup-script=.*/display-setup-script=xrandr -s $RESOLUTION/"
```

### Step 4: Disable Screen Blanking
**1 minute**

This prevents the display from going blank or entering sleep mode — critical for a wall-mounted calendar that should always be visible.

- `-s 0` — Disables the screensaver timeout (screen never blanks due to inactivity)
- `dpms` — Disables DPMS (Display Power Management Signaling), preventing the display from entering standby/suspend/off states

```bash
sudo sed /etc/lightdm/lightdm.conf -i -e "s/^\(#\|\)xserver-command=.*/xserver-command=X -s 0 dpms/"
```

### Step 5: Create Kiosk Launch Script
**5 minutes**

```bash
sudo mkdir -p /opt/kiosk/
sudo nano /opt/kiosk/kiosk.sh
```

Add this content (replace `YOUR_HOME_ASSISTANT_URL` with your Home Assistant server's IP or hostname, e.g., `192.168.1.100` or `homeassistant.local`):

> **What URL to use:** The path after the port (`/family-calendar/family-calendar`) is your **dashboard path** + **view path** from the [Home Assistant Setup Guide](HOME_ASSISTANT_SETUP_GUIDE.md) (Step 8). If you chose different paths when creating your dashboard, substitute them here.

```bash
#!/bin/bash

# Start a D-Bus session and launch the on-screen keyboard.
# dbus-run-session provides a message bus that onboard needs to
# communicate with other applications (e.g., detecting text fields).
/usr/bin/dbus-run-session /usr/bin/onboard &

# Launch Chromium in kiosk mode
/usr/bin/chromium \
  --app=http://YOUR_HOME_ASSISTANT_URL:8123/family-calendar/family-calendar \
  --noerrdialogs \
  --disable-infobars \
  --kiosk \
  --disable-session-crashed-bubble \
  --disable-restore-session-state \
  --window-position=0,0 \
  --window-size=1920,1080 \
  --check-for-update-interval=31536000
```

**Chromium flags explained:**
| Flag | Purpose |
|------|---------|
| `--app=URL` | Opens in "app mode" — no address bar, tabs, or browser UI; just the web page. The URL should point to your HA dashboard (see [Home Assistant Setup Guide](HOME_ASSISTANT_SETUP_GUIDE.md), Step 8) |
| `--kiosk` | Full-screen with no window decorations or exit controls |
| `--noerrdialogs` | Suppresses crash/error pop-up dialogs |
| `--disable-infobars` | Hides notification bars (e.g., "Chrome is not your default browser") |
| `--disable-session-crashed-bubble` | Prevents the "restore pages" prompt after a crash |
| `--disable-restore-session-state` | Starts fresh rather than restoring previous tabs |
| `--window-position=0,0` | Positions window at top-left corner of the display |
| `--window-size=1920,1080` | Sets window dimensions to match your display resolution |
| `--check-for-update-interval=31536000` | Suppresses update nags (checks once per year in seconds) |

Make it executable:
```bash
sudo chmod +x /opt/kiosk/kiosk.sh
```

### Step 6: Create Systemd Service
**3 minutes**

```bash
sudo nano /lib/systemd/system/kiosk.service
```

Add this content (replace `YOUR_USERNAME` with the same username from Phase 1, Step 2):
```ini
[Unit]
Description=Home Assistant Calendar Kiosk
Wants=graphical.target
After=graphical.target

[Service]
Environment=DISPLAY=:0
ExecStart=/usr/bin/bash /opt/kiosk/kiosk.sh
Restart=always
User=YOUR_USERNAME
Group=YOUR_USERNAME

[Install]
WantedBy=graphical.target
```

**Service fields explained:**
| Field | Purpose |
|-------|---------|
| `Wants=graphical.target` | Declares a dependency on the graphical desktop environment |
| `After=graphical.target` | Waits for the graphical desktop to be fully ready before starting |
| `Environment=DISPLAY=:0` | Tells the service which X display to render on (`:0` is the primary display) |
| `ExecStart=...` | The command to run when the service starts |
| `Restart=always` | Auto-restarts the kiosk if Chromium crashes |
| `User` / `Group` | Runs the service as your user (not root) — same username from Phase 1, Step 2 |
| `WantedBy=graphical.target` | Starts this service whenever the system boots to graphical mode |

### Step 7: Enable and Start Service
**2 minutes**

```bash
sudo systemctl daemon-reload
sudo systemctl enable kiosk
sudo reboot
```

---

## Phase 4: Home Assistant Dashboard Setup
**Estimated time: 60-90 minutes**

**Note:** This assumes you have Home Assistant running elsewhere (separate server, Pi, or Home Assistant Yellow/Green).

### Step 1: Install HACS
**10-15 minutes**

On your Home Assistant instance:

**For Home Assistant OS/Supervised:**
1. Go to Settings → Add-ons → Add-on Store
2. Click three dots menu → Repositories
3. Add: `https://github.com/hacs/addons`
4. Install "Get HACS" add-on
5. Start it and check logs for instructions

**For Container/Core:**
```bash
wget -O - https://get.hacs.xyz | bash -
```

Then restart Home Assistant.

### Step 2: Install HACS Frontend Components
**15-20 minutes**

In HACS → Frontend, search for and install each component below.

**Required** (directly used in `dashboard.yaml`):
| Component | Purpose |
|-----------|---------|
| `week-planner-card` | Weekly calendar planner view — the main calendar display |
| `bubble-card` | Interactive button/pop-up UI components (person filter buttons, Add Event pop-up) |
| `config-template-card` | Enables dynamic Jinja2/JS templating in card configurations (powers the filter logic) |
| `better-moment-card` | Formatted date/time display in the header |
| `weather-card` | Weather conditions display |
| `button-card` | Custom styled action buttons (the "Add Event to Calendar" submit button) |
| `card-mod` | Apply custom CSS styling to any card (used throughout for visual tweaks) |

**Recommended** (useful for customization and kiosk behavior):
| Component | Purpose |
|-----------|---------|
| `browser_mod` | Browser-level control — hide sidebar, pop-ups, kiosk enhancements |
| `layout-card` | Custom layout arrangements — useful if adapting to non-1080p screens |

### Step 3: Configure the Package
**10-15 minutes**

Home Assistant's [packages](https://www.home-assistant.io/docs/configuration/packages/) system lets you organize configuration into separate files by feature/domain, rather than cramming everything into `configuration.yaml`. The `family_calendar.yaml` package contains all the input helpers, scripts, and automations needed for the calendar's filtering and event-creation features.

1. Edit `configuration.yaml` to enable the packages directory:
```yaml
homeassistant:
  packages: !include_dir_named packages
```

2. Create a `packages` folder in your Home Assistant config directory
3. Download `family_calendar.yaml` from the repo and place it in `packages/`
4. Restart Home Assistant

### Step 4: Set Up Calendars
**10-15 minutes** (Local) / **30-45 minutes** (Google)

**Local Calendar (easiest):**
1. Settings → Devices & Services → Add Integration → Local Calendar
2. Create calendars: `Alice`, `Bob`, `Charlie`, `Daisy`, `Family`

**Google Calendar:**
1. Add Google Calendar integration
2. Edit `packages/family_calendar.yaml` to map your calendar entities

### Step 5: Create the Dashboard
**10-15 minutes**

1. Create new Dashboard → Add View → set view type to **Sections** (a newer HA layout mode that arranges cards in a responsive grid with configurable column spans)
2. Open the raw YAML editor for the view and paste the contents of `dashboard.yaml`
3. **Search and replace placeholder entities** with your own:
   - `person.alice`, `person.bob`, `person.charlie`, `person.daisy` → your Home Assistant person entities (Settings → People)
   - `weather.home` → your weather integration entity
   - `/local/your-background-image.jpg` → path to your background image (upload to `www/` folder in HA config)

### Step 6: Apply Theme (Optional)
**5-10 minutes**

1. Add to `configuration.yaml`:
```yaml
frontend:
  themes: !include_dir_merge_named themes
```
2. Create `themes` folder
3. Download `skylight.yaml` into `themes/`
4. Restart Home Assistant
5. Go to Profile → Theme → Select "Skylight"

---

## Phase 5: Final Configuration
**Estimated time: 10-15 minutes**

### On First Login to Kiosk
**5 minutes**

1. Log into Home Assistant on the kiosk
2. **Check "Keep me logged in"** to prevent repeated logins
3. Navigate to your calendar dashboard

### Configure On-Screen Keyboard (for touch)
**5 minutes**

```bash
DISPLAY=:0 onboard-settings
```
- Enable "Auto-show when editing text"
- Enable Accessibility mode when prompted

### Adjustments for Different Screen Sizes
**5-10 minutes** (if needed)

The dashboard is designed for 1920x1080. For smaller screens:
- Edit `dashboard.yaml` to adjust column counts
- Modify card sizes in the week-planner-card configuration

### Performance Tweaks (Optional)
**5 minutes**

For smoother performance, add to `/boot/firmware/config.txt`:
```ini
# Increase GPU memory for better graphics performance
gpu_mem=256

# Enable hardware acceleration
dtoverlay=vc4-kms-v3d
```

---

## Optional: SSH Public-Key Setup

Public-key authentication is more secure than passwords and lets you SSH without typing a password each time. You can set this up during flashing (Step 2) or add it later.

### On Your Computer (not the Pi)

```bash
# Check if you already have a key
ls ~/.ssh/id_ed25519.pub 2>/dev/null || ls ~/.ssh/id_rsa.pub 2>/dev/null

# If no key exists, generate one (press Enter to accept defaults)
ssh-keygen -t ed25519 -C "your-email@example.com"

# Display your public key
cat ~/.ssh/id_ed25519.pub
```

### Option A: During Flashing (Raspberry Pi Imager)

Copy your public key and paste it into the "authorized_keys" field in the Services tab.

### Option B: After Setup (Pi Already Running)

```bash
# From your computer, copy your key to the Pi (you'll need to enter your password once)
ssh-copy-id your-username@calendar-kiosk.local

# Or manually: SSH into the Pi and add your key
ssh your-username@calendar-kiosk.local
mkdir -p ~/.ssh && chmod 700 ~/.ssh
echo "YOUR_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### Copying Your Public Key to Clipboard

```bash
# macOS
cat ~/.ssh/id_ed25519.pub | pbcopy

# Linux
cat ~/.ssh/id_ed25519.pub | xclip -selection clipboard

# Windows (Git Bash)
cat ~/.ssh/id_ed25519.pub | clip

# Windows (PowerShell)
Get-Content ~/.ssh/id_ed25519.pub | Set-Clipboard
```

After setup, connect without a password:
```bash
ssh your-username@calendar-kiosk.local
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| No display output | Check HDMI connection |
| Touch not working | Verify USB connection; check `dmesg` for touch device |
| Kiosk not starting | Check `sudo systemctl status kiosk` for errors |
| Wrong resolution | Edit `/opt/kiosk/kiosk.sh` window-size parameter |
| Screen stays blank after boot | Wait 30+ seconds; check lightdm logs |
| Calendar not loading | Verify Home Assistant URL is reachable from Pi |
| Micro-HDMI no signal | Use HDMI0 port (closest to USB-C); check cable compatibility |
| Overheating | Add heatsinks and/or fan; check `vcgencmd measure_temp` |

### Pi Restarted During Phase 3 (Stuck on Boot Screen)

If the Pi was restarted during Phase 3 setup, it may boot to a terminal screen showing a message like "Completed socket interaction for boot stage final" with no desktop or browser appearing.

**Why this happens:** Phase 3 partially reconfigured the boot process — for example, `systemctl set-default graphical.target` may have run, but LightDM or other required packages aren't fully installed or configured yet. The system tries to boot into a graphical environment but falls back to a text console.

**1. Get access to the Pi**

The console may be waiting for input below the boot messages. Press **Enter** to get a login prompt, then log in with your Pi credentials.

> **Tip:** If you can't interact with the console directly, SSH in from another machine on the same network: `ssh pi@<your-pi-ip>`

**2. Check which Phase 3 step you were on**

Run these commands to see what's already been completed:

```bash
# Check if packages are installed
dpkg -l | grep -E "lightdm|chromium|onboard|xserver-xorg"

# Check if auto-login is configured
grep autologin-user /etc/lightdm/lightdm.conf

# Check if kiosk script exists
ls /opt/kiosk/kiosk.sh

# Check if kiosk service exists
ls /lib/systemd/system/kiosk.service
```

**3. Resume Phase 3 from the first incomplete step**

All Phase 3 commands are idempotent — safe to re-run even if they partially completed before. Based on what's missing from Step 2:

- If packages haven't finished installing, start from Phase 3, Step 1 (`sudo apt install ...`)
- If packages are installed but auto-login isn't configured, start from the LightDM configuration step
- If the kiosk script or service is missing, pick up from that step

Work through the remaining Phase 3 steps in order.

**4. Reboot**

Once all Phase 3 steps are complete:

```bash
sudo reboot
```

The Pi should now boot into the graphical kiosk environment as expected.

### Touchscreen Not Responding to Touch

If the display shows video but doesn't respond to touch input:

**1. Check the USB connection**

Touch input travels over USB, separate from the HDMI video signal. Make sure:
- The USB cable is a **data cable**, not a charge-only cable (charge-only cables lack the data wires)
- The cable is plugged into one of the Pi's **USB 2.0 ports** (the black ones, not blue)
- Try a different USB cable or port

**2. Verify the Pi detects the touch device**

```bash
# List connected USB devices — look for a touch controller
lsusb

# Check kernel messages for touch/input device registration
dmesg | grep -i touch

# List all recognized input devices
cat /proc/bus/input/devices
```

If nothing touch-related appears, the Pi isn't seeing the device at all — it's almost certainly a cable or port issue.

**3. Check if input events are being generated**

```bash
# Install evtest if not present
sudo apt install evtest -y

# List input devices and test one
sudo evtest
```

Select the touch device from the list, then tap the screen. If you see event output, touch hardware is working and the issue is software-side (calibration or mapping).

**4. Map touch input to the correct display**

If you have multiple displays or the touch coordinates are misaligned, you may need to map the touch device to the HDMI output:

```bash
# Find your touch device name
xinput list

# Find your display output name
xrandr --query

# Map touch input to the display (replace names with yours)
xinput map-to-output "YOUR_TOUCH_DEVICE_NAME" HDMI-1
```

To make this persistent, add the `xinput map-to-output` command to your `/opt/kiosk/kiosk.sh` script before the Chromium launch line.

**5. Install a touch driver (rare)**

Most USB touchscreens work with the generic HID driver built into the kernel. If yours doesn't, check the manufacturer's documentation for a Linux driver or firmware update.

---

## Sources

- [Pi My Life Up - Raspberry Pi Home Assistant Kiosk](https://pimylifeup.com/raspberry-pi-home-assistant-kiosk/)
- [HACS Installation Guide](https://hacs.xyz/docs/use/download/download/)
- [TouchKio - GitHub](https://github.com/leukipp/touchkio)
- [Home Assistant Community - Kiosk Mode for Raspberry Pi](https://community.home-assistant.io/t/kiosk-mode-for-raspberry-pi-with-touch-display/821196)
