# Raspberry Pi 3 Smart Calendar Kiosk Setup Guide

A complete guide to building a DIY Skylight-style family calendar using a Raspberry Pi 3, external touchscreen, and Home Assistant.

---

## Parts List

### Required Components

| Component | Notes |
|-----------|-------|
| **Raspberry Pi 3 Model B/B+** | Pi 4 recommended if budget allows for better performance |
| **MicroSD Card** | 32GB+ Class 10 recommended |
| **Power Supply** | 5V 2.5A for Pi 3 (official recommended) |
| **Touchscreen Display** | See options below |
| **HDMI/DSI Cable** | Depends on display type |
| **MicroSD Card Reader** | For flashing OS |

### Touchscreen Options

**Option A: Official Raspberry Pi 7" Touch Display (~$60)**
- 800x480 resolution
- Connects via DSI ribbon cable
- Includes mounting hardware
- Touch works out of box

**Option B: HDMI Touchscreen (7"-10") (~$50-100)**
- 1024x600 to 1920x1080 resolution
- USB for touch, HDMI for display
- More resolution options available
- Example: Elecrow 10.1" 1920x1080

**Option C: Larger HDMI Touchscreen (15"+)**
- For countertop display similar to original project
- Requires more power

### Optional Components

| Component | Purpose |
|-----------|---------|
| **Case/Stand** | Countertop mounting |
| **Heatsinks** | Thermal management |
| **USB Keyboard/Mouse** | Initial setup only |

---

## Phase 1: Raspberry Pi OS Setup

### Step 1: Download Raspberry Pi Imager
1. Download from [raspberrypi.com/software](https://www.raspberrypi.com/software/)
2. Install on your computer

### Step 2: Flash the SD Card
1. Insert MicroSD card into your computer
2. Open Raspberry Pi Imager
3. Click **Choose Device** → Select **Raspberry Pi 3**
4. Click **Choose OS** → **Raspberry Pi OS (64-bit)** with Desktop
5. Click **Choose Storage** → Select your SD card
6. Click **Next**, then **Edit Settings**:
   - Set hostname: `calendar-kiosk`
   - Set username/password
   - Configure Wi-Fi (SSID and password)
   - Enable SSH (Services tab)
7. Click **Save** → **Yes** to apply settings → **Yes** to flash

### Step 3: First Boot
1. Insert SD card into Raspberry Pi
2. Connect keyboard, mouse, and monitor temporarily
3. Connect power supply
4. Wait for initial boot and setup (2-3 minutes)

### Step 4: Update the System
Open Terminal and run:
```bash
sudo apt update
sudo apt upgrade -y
```

---

## Phase 2: Touchscreen Hardware Setup

### For Official 7" DSI Display

1. **Power off the Pi** completely
2. **Connect the ribbon cable**:
   - Insert one end into the "DISPLAY" port on the display board (contacts facing away from screen)
   - Insert other end into the "DISPLAY" port on the Pi (contacts facing inward toward Pi)
3. **Connect power jumper wires**:
   - Connect Pi GPIO **5V** (Pin 2 or 4) → Display **5V**
   - Connect Pi GPIO **GND** (Pin 6) → Display **GND**
4. **Mount the Pi** using the included standoffs
5. **Power on** - display should work automatically

### For HDMI Touchscreen

1. Connect **HDMI cable** from display to Pi
2. Connect **USB cable** from display to Pi (for touch)
3. Power on the display and Pi
4. Touch should work automatically on modern Raspberry Pi OS

### Rotate Display (if needed)
```bash
# For desktop environment
# Menu → Preferences → Screen Configuration
# Right-click display → Orientation → Choose rotation

# For command line (headless)
# Add to /boot/firmware/cmdline.txt:
# video=DSI-1:800x480@60,rotate=180
```

---

## Phase 3: Kiosk Mode Setup

### Step 1: Install Required Packages
```bash
sudo apt install xserver-xorg lightdm onboard chromium-browser sed -y
```

### Step 2: Configure Auto-Login
```bash
# Enable graphical boot
sudo systemctl --quiet set-default graphical.target

# Set auto-login (replace YOUR_USERNAME with your actual username)
sudo sed /etc/lightdm/lightdm.conf -i -e "s/^\(#\|\)autologin-user=.*/autologin-user=YOUR_USERNAME/"
```

### Step 3: Set Screen Resolution
```bash
# For 1920x1080 display:
RESOLUTION=1920x1080
sudo sed /etc/lightdm/lightdm.conf -i -e "s/^\(#\|\)display-setup-script=.*/display-setup-script=xrandr -s $RESOLUTION/"

# For 800x480 (official 7" display):
RESOLUTION=800x480
sudo sed /etc/lightdm/lightdm.conf -i -e "s/^\(#\|\)display-setup-script=.*/display-setup-script=xrandr -s $RESOLUTION/"
```

### Step 4: Disable Screen Blanking
```bash
sudo sed /etc/lightdm/lightdm.conf -i -e "s/^\(#\|\)xserver-command=.*/xserver-command=X -s 0 dpms/"
```

### Step 5: Create Kiosk Launch Script
```bash
sudo mkdir -p /opt/kiosk/
sudo nano /opt/kiosk/kiosk.sh
```

Add this content (replace `YOUR_HOME_ASSISTANT_URL` with your actual URL):
```bash
#!/bin/bash

# Start on-screen keyboard for touch input
/usr/bin/dbus-run-session /usr/bin/onboard &

# Launch Chromium in kiosk mode
# Replace URL and resolution as needed
/usr/bin/chromium-browser \
  --app=http://YOUR_HOME_ASSISTANT_URL:8123/lovelace/calendar \
  --noerrdialogs \
  --disable-infobars \
  --kiosk \
  --disable-session-crashed-bubble \
  --disable-restore-session-state \
  --window-position=0,0 \
  --window-size=1920,1080 \
  --check-for-update-interval=31536000
```

Make it executable:
```bash
sudo chmod +x /opt/kiosk/kiosk.sh
```

### Step 6: Create Systemd Service
```bash
sudo nano /lib/systemd/system/kiosk.service
```

Add this content (replace `YOUR_USERNAME`):
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

### Step 7: Enable and Start Service
```bash
sudo systemctl daemon-reload
sudo systemctl enable kiosk
sudo reboot
```

---

## Phase 4: Home Assistant Dashboard Setup

**Note:** This assumes you have Home Assistant running elsewhere (separate server, Pi, or Home Assistant Yellow/Green).

### Step 1: Install HACS
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

### Step 2: Install Required HACS Frontend Components
In HACS → Frontend, install:
- `week-planner-card`
- `bubble-card`
- `config-template-card`
- `card-mod`
- `better-moment-card`
- `weather-card`
- `browser_mod`
- `layout-card`

### Step 3: Configure the Package
1. Edit `configuration.yaml`:
```yaml
homeassistant:
  packages: !include_dir_named packages
```

2. Create `packages` folder in your config directory
3. Download `family_calendar.yaml` from the repo and place it in `packages/`
4. Restart Home Assistant

### Step 4: Set Up Calendars
**Local Calendar (easiest):**
1. Settings → Devices & Services → Add Integration → Local Calendar
2. Create calendars: `Alice`, `Bob`, `Charlie`, `Daisy`, `Family`

**Google Calendar:**
1. Add Google Calendar integration
2. Edit `packages/family_calendar.yaml` to map your calendar entities

### Step 5: Create the Dashboard
1. Create new Dashboard → Add View (type: **Sections**)
2. Copy contents from `dashboard.yaml`
3. Search/replace person entities, weather entity, and background image

### Step 6: Apply Theme (Optional)
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

### On First Login to Kiosk
1. Log into Home Assistant on the kiosk
2. **Check "Keep me logged in"** to prevent repeated logins
3. Navigate to your calendar dashboard

### Configure On-Screen Keyboard (for touch)
```bash
DISPLAY=:0 onboard-settings
```
- Enable "Auto-show when editing text"
- Enable Accessibility mode when prompted

### Adjustments for Different Screen Sizes
The dashboard is designed for 1920x1080. For smaller screens:
- Edit `dashboard.yaml` to adjust column counts
- Modify card sizes in the week-planner-card configuration

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| No display output | Check ribbon cable connection (DSI) or HDMI connection |
| Touch not working | Verify USB connection; check `dmesg` for touch device |
| Kiosk not starting | Check `sudo systemctl status kiosk` for errors |
| Wrong resolution | Edit `/opt/kiosk/kiosk.sh` window-size parameter |
| Screen stays blank after boot | Wait 30+ seconds; check lightdm logs |
| Calendar not loading | Verify Home Assistant URL is reachable from Pi |

---

## Sources

- [Pi My Life Up - Raspberry Pi Home Assistant Kiosk](https://pimylifeup.com/raspberry-pi-home-assistant-kiosk/)
- [Raspberry Pi Touch Display Documentation](https://www.raspberrypi.com/documentation/accessories/display.html)
- [HACS Installation Guide](https://hacs.xyz/docs/use/download/download/)
- [TouchKio - GitHub](https://github.com/leukipp/touchkio)
- [Home Assistant Community - Kiosk Mode for Raspberry Pi](https://community.home-assistant.io/t/kiosk-mode-for-raspberry-pi-with-touch-display/821196)
- [Howchoo - Official Raspberry Pi Touchscreen Setup](https://howchoo.com/pi/raspberry-pi-touchscreen-setup)
