# Home Assistant Setup Guide (From Scratch)

This guide assumes you have **nothing set up yet** — no Home Assistant, no Raspberry Pi configured, nothing. It walks you through everything from zero to a working family calendar kiosk.

---

## Understanding the Two Devices

This project uses **two separate devices** that talk to each other over your home network:

| Device | Role | What It Runs |
|--------|------|-------------|
| **Home Assistant Server** | The brains — stores calendars, manages automations, serves the dashboard | Home Assistant software |
| **Raspberry Pi** | The display — shows the calendar on a screen in kiosk mode | A web browser pointed at your Home Assistant dashboard |

Think of it like this: the Home Assistant server is a website, and the Raspberry Pi is a dedicated monitor showing that website full-screen.

**Important:** The Home Assistant server and the Raspberry Pi are **different devices**. The Pi does NOT run Home Assistant — it just displays it. Home Assistant runs on a separate machine.

---

## What You Need to Buy/Have

Before starting, gather these:

### For the Home Assistant Server (pick ONE option):
- **Option A — Dedicated mini PC (recommended):** A Raspberry Pi 4/5 (separate from the kiosk Pi), Intel NUC, or old laptop
- **Option B — Home Assistant Green/Yellow:** Purpose-built HA hardware from Nabu Casa (~$99-130)
- **Option C — Existing computer:** A machine that can stay on 24/7 (Mac, Windows PC, Linux box)

### For the Kiosk Display:
- Raspberry Pi 4 or 5 (with at least 2GB RAM)
- MicroSD card (32GB+)
- USB-C power supply for the Pi
- A monitor/TV/display with HDMI input
- Micro-HDMI to HDMI cable (for Pi 4) or standard HDMI (for Pi 5)

### For Both:
- A working home Wi-Fi network (or Ethernet cables)
- A computer to do the initial setup from (your regular laptop/desktop)

---

## Phase 1: Set Up the Home Assistant Server

> **Where:** On your chosen server device (NOT the Raspberry Pi kiosk)
>
> **What you're doing:** Installing the Home Assistant software that will manage your calendars and serve the dashboard

### Step 1.1: Choose Your Installation Method

| Method | Best For | Difficulty |
|--------|----------|-----------|
| **Home Assistant OS** (recommended) | Dedicated Pi or mini PC | Easiest |
| **Home Assistant Container** | Existing Linux server with Docker | Medium |
| **Home Assistant Supervised** | Existing Debian machine, want add-ons | Medium |
| **Home Assistant Core** | Advanced users, Python venv | Hardest |

**For beginners, use Home Assistant OS.** It's a complete operating system that includes everything.

### Step 1.2: Install Home Assistant OS

If using a **dedicated Raspberry Pi** as your HA server (separate from the kiosk Pi):

1. On your regular computer, download and install [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Insert the microSD card for your **server Pi** into your computer
3. Open Raspberry Pi Imager:
   - Click "Choose OS" → "Other specific-purpose OS" → "Home assistants and home automation" → "Home Assistant" → select your Pi model
   - Click "Choose Storage" → select your SD card
   - Click "Write"
4. Insert the SD card into your **server Pi**
5. Connect the server Pi to your router via Ethernet (recommended) or set up Wi-Fi
6. Plug in the power supply
7. Wait 5-10 minutes for the initial boot

If using a **mini PC, NUC, or old laptop:**
- Download the Home Assistant OS image for your hardware from [the official install page](https://www.home-assistant.io/installation/)
- Flash it to the device's hard drive using Balena Etcher or similar

### Step 1.3: Access Home Assistant for the First Time

1. On your regular computer, open a web browser
2. Navigate to: `http://homeassistant.local:8123`
   - If that doesn't work, try: `http://homeassistant:8123`
   - Still nothing? Find the IP address of your server device on your router's admin page, then go to `http://THAT_IP:8123`
3. You'll see a "Preparing Home Assistant" screen — this can take up to 20 minutes on first boot. Don't refresh.
4. Once ready, you'll see the onboarding wizard:
   - Create your admin account (remember these credentials!)
   - Set your home location (this is used for weather)
   - Choose your units (imperial/metric)
   - Click through the remaining setup steps

**Congratulations — Home Assistant is now running.** Leave this device powered on permanently. It's your home automation server.

---

## Phase 2: Configure Home Assistant for the Calendar

> **Where:** In the Home Assistant web interface (accessed from any browser on your network at `http://homeassistant.local:8123`)
>
> **What you're doing:** Setting up the calendar data, weather, and custom components that power the dashboard

### Step 2.1: Set Up Person Entities

The dashboard shows each family member as a filter button. Each needs a "person" entity.

1. Go to **Settings → People** (in the HA sidebar)
2. Click **Add Person**
3. For each family member:
   - Enter their name (e.g., "Alice")
   - Optionally assign a user account
   - Optionally add a profile picture (shows on the filter button)
4. Repeat for everyone who should have their own calendar

**Note their entity IDs** — after creating, go to **Developer Tools → States**, filter for `person.`, and record what HA generated (e.g., `person.alice`, `person.john_smith`). You'll need these later.

### Step 2.2: Set Up Calendar Integrations

The dashboard displays events from calendar entities. Choose one:

#### Option A: Local Calendar (Simplest — Start Here)

Local Calendar stores events directly in Home Assistant. No external accounts needed.

1. Go to **Settings → Devices & Services**
2. Click **Add Integration** (bottom right)
3. Search for **Local Calendar**
4. Click to add it
5. Create one calendar per family member plus shared ones:
   - `Alice` → creates `calendar.alice`
   - `Bob` → creates `calendar.bob`
   - `Family` → creates `calendar.family`
   - `Birthdays` → creates `calendar.birthdays`
   - `Holidays` → creates `calendar.holidays`

#### Option B: Google Calendar (Syncs with Existing Google Calendars)

More complex setup, but pulls in events you already have.

1. Follow the [official Google Calendar integration docs](https://www.home-assistant.io/integrations/google/)
2. You'll need to create a Google Cloud project, enable the Calendar API, and create OAuth credentials
3. Add the integration in HA and authorize your Google account(s)

#### Verifying Calendars Work

1. Go to **Developer Tools → States**
2. Filter for `calendar.`
3. Confirm each calendar entity exists
4. Add a test event through HA to verify it works

### Step 2.3: Set Up Weather Integration

1. Go to **Settings → Devices & Services → Add Integration**
2. Search for **Meteorologisk institutt (Met.no)** — this is free, no API key needed
3. Enter your location coordinates (it may auto-detect from your home location set during onboarding)
4. This creates a `weather.home` entity (or similar)

Verify: **Developer Tools → States** → filter `weather.` → confirm it shows a condition like "sunny" or "cloudy."

### Step 2.4: Install HACS (Home Assistant Community Store)

HACS lets you install the custom dashboard cards this project needs. It's not built into HA — you install it separately.

**For Home Assistant OS (what you installed in Phase 1):**

1. Go to **Settings → Add-ons → Add-on Store**
2. Click the three-dot menu (top right) → **Repositories**
3. Add this URL: `https://github.com/hacs/addons`
4. Find and install the **"Get HACS"** add-on
5. Start the add-on and check its logs — it will give you a setup link
6. Click the link and authorize HACS with a GitHub account (you'll need a free GitHub account — create one at github.com if you don't have one)

After installation, HACS appears as a new item in the HA sidebar.

### Step 2.5: Install Required Frontend Components

These custom cards power the dashboard UI. Install each one through HACS.

For **each component below**:
1. In the HA sidebar, click **HACS**
2. Click **Frontend** (top)
3. Click the **+** button (bottom right) or search bar
4. Search for the component name
5. Click it, then click **Download**

| Search For | Author/Note | What It Does |
|------------|-------------|--------------|
| `week-planner-card` | — | The main calendar grid |
| `bubble-card` | — | Filter buttons, Add Event popup, view selector |
| `config-template-card` | — | Makes calendar react to filter changes |
| `better-moment-card` | — | Date/time display in header |
| `weather-card` | by bramkragten | Weather display in header |
| `button-card` | by RomRider | Styled "Add Event" button |
| `card-mod` | — | CSS customization throughout dashboard |

Optional but recommended:
| `browser_mod` | — | Hides HA sidebar/header for kiosk mode |

**After installing all components:**
1. Go to **Settings → System → Restart** (restart Home Assistant)
2. After restart, hard-refresh your browser (Ctrl+Shift+R)
3. Verify: go to any dashboard → Edit → Add Card — you should see the new card types listed

### Step 2.6: Set Up the Packages Directory

The `family_calendar.yaml` file from this repo contains all the helpers and scripts that make the calendar work. Home Assistant's "packages" feature lets you load it as a single file.

1. **You need to edit a file on your Home Assistant server.** Access it via one of these methods:
   - **File Editor add-on (easiest):** Go to Settings → Add-ons → Add-on Store → search "File editor" → install it → open it from the sidebar
   - **SSH add-on:** Settings → Add-ons → search "Terminal & SSH" → install → open terminal
   - **Samba add-on:** Settings → Add-ons → search "Samba share" → install → access files from your computer's file explorer

2. **Edit `configuration.yaml`** — add these lines (if a `homeassistant:` section already exists, just add the `packages:` line under it):

```yaml
homeassistant:
  packages: !include_dir_named packages
```

3. **Create the packages directory:**
   - Using File Editor: navigate to `/config/`, create a new folder called `packages`
   - Using SSH terminal: `mkdir -p /config/packages`

4. **Copy `packages/family_calendar.yaml` from this GitHub repo** into your `/config/packages/` directory
   - Easiest method: open the file on GitHub, click "Raw", copy all the text, then paste it into a new file called `family_calendar.yaml` in your `/config/packages/` folder using the File Editor add-on

5. **Restart Home Assistant** (Settings → System → Restart)

6. **Verify it loaded:** Go to Developer Tools → States → filter for `input_text.` — you should see entries like `input_text.alice_calendar_filter`, `input_text.bob_calendar_filter`, etc.

### Step 2.7: Customize for Your Family

The package and dashboard ship with example names (Alice, Bob, Charlie, Daisy). Replace these with your actual family members.

**In `packages/family_calendar.yaml`**, find and replace every occurrence of the example names. The key places to edit:

1. **Input text filters** — rename `alice_calendar_filter` to match your family members
2. **Input select options** — update the dropdown list of calendar names
3. **Scripts** — rename `alice_calendar_visible_filter` scripts to match
4. **Calendar map** — map dropdown names to your actual calendar entity IDs:
```yaml
calendar_map:
  "Mom": "calendar.mom"
  "Dad": "calendar.dad"
  "Kids": "calendar.kids"
  "Family": "calendar.family"
```

**Colors** — each person has a color used on buttons and calendar events. Edit hex codes in `dashboard.yaml`:
| Default Member | Color | Hex |
|---------------|-------|-----|
| Alice | Coral | `#fb8072` |
| Bob | Orange | `#fdbf6f` |
| Charlie | Blue | `#a6cee3` |
| Daisy | Purple | `#cab2d6` |
| Family | Blue | `#4A90E2` |
| Birthdays | Green | `#33a02c` |
| Holidays | Orange | `#ff7f00` |

### Step 2.8: Create the Dashboard

1. Go to **Settings → Dashboards**
2. Click **Add Dashboard**
3. Name it "Family Calendar"
4. Set the URL path to `family-calendar`
5. Optionally set icon: `mdi:calendar`
6. Open your new dashboard
7. Click the three-dot menu (top right) → **Edit Dashboard**
8. Click the three-dot menu again → **Raw configuration editor**
9. Paste the entire contents of `dashboard.yaml` from this repo
10. Click **Save**

**Then replace placeholder entities** throughout the pasted YAML:
- `person.alice` → your person entities
- `calendar.alice` → your calendar entities
- `weather.home` → your weather entity
- `script.alice_calendar_visible_filter` → your renamed scripts
- `input_text.alice_calendar_filter` → your renamed input helpers
- `/local/your-background-image.jpg` → your background image (see below)

**Background image:**
1. Using File Editor, navigate to `/config/`
2. Create a folder called `www` if it doesn't exist
3. Upload your background image into `/config/www/` (e.g., `calendar-bg.jpg`)
4. Reference it in the dashboard as `/local/calendar-bg.jpg`

### Step 2.9: Set Up Theme (Optional)

Applies a clean serif font (Ovo) for a wall-calendar look.

1. Add to `configuration.yaml`:
```yaml
frontend:
  themes: !include_dir_merge_named themes
```

2. Create `/config/themes/` directory
3. Copy `themes/skylight.yaml` from this repo into that directory
4. Restart Home Assistant
5. Apply: click your profile icon (bottom-left) → Theme → select "Skylight"

### Step 2.10: Verify Everything Works

Open the dashboard in a regular browser on your computer and check:

- [ ] Date and time display correctly in the header
- [ ] Weather shows current conditions
- [ ] Calendar events appear in the week planner
- [ ] Tapping filter buttons toggles calendar visibility
- [ ] "Add Event" button opens the popup form
- [ ] Submitting an event creates it on the correct calendar
- [ ] View selector (Today/Tomorrow/Week/etc.) changes the displayed days

**Common issues:**

| Problem | Fix |
|---------|-----|
| Cards show "Custom element doesn't exist" | Install the missing HACS component, then clear browser cache (Ctrl+Shift+R) |
| Filter buttons show "unavailable" | Entity ID mismatch — check Developer Tools → States for correct IDs |
| Calendar shows no events | Calendar entity ID wrong, or no events exist yet |
| Weather shows nothing | Weather entity ID doesn't match what's in the dashboard YAML |
| Dashboard is blank | YAML syntax error — check Settings → System → Logs |

---

## Phase 3: Set Up the Raspberry Pi Kiosk

> **Where:** On the **kiosk Raspberry Pi** (the one connected to your display)
>
> **What you're doing:** Setting up a dedicated Pi to display the calendar full-screen on a monitor/TV

This is a separate device from your Home Assistant server. It runs Raspberry Pi OS and opens a web browser pointed at your Home Assistant dashboard.

**Follow the [Raspberry Pi Setup Guide](RASPBERRY_PI_SETUP_GUIDE.md) for detailed kiosk instructions.**

The key thing you'll need from the HA setup above is the **dashboard URL**:
```
http://YOUR_HA_SERVER_IP:8123/family-calendar/family-calendar
```

Replace `YOUR_HA_SERVER_IP` with the actual IP address of your Home Assistant server (find it in your router's admin page, or on the HA server itself under Settings → System → Network).

---

## Phase 4: Configure Browser Mod for Kiosk Mode (Optional)

> **Where:** In the Home Assistant web interface (but targeting the Pi's browser)
>
> **What you're doing:** Telling HA to hide the sidebar and header when the Pi accesses it

This makes the dashboard truly full-screen on the Pi display, removing HA's navigation elements.

1. Ensure `browser_mod` was installed via HACS (Step 2.5)
2. Open the calendar dashboard **on the Raspberry Pi's browser** (it should already be showing it if you completed Phase 3)
3. On your regular computer, go to **Settings → Devices & Services → Browser Mod**
4. Register the Pi's browser as a device
5. Configure its settings:
   - **Hide sidebar:** On
   - **Hide header:** On
   - **Default dashboard:** `/family-calendar`

---

## Summary: What's Installed Where

| Component | Installed On | Purpose |
|-----------|-------------|---------|
| Home Assistant OS | HA Server (separate Pi or mini PC) | Runs the calendar backend |
| HACS | HA Server (inside Home Assistant) | Installs custom cards |
| All HACS frontend components | HA Server (inside Home Assistant) | Powers the dashboard UI |
| Calendar integrations | HA Server (inside Home Assistant) | Provides calendar data |
| Weather integration | HA Server (inside Home Assistant) | Provides weather data |
| `family_calendar.yaml` | HA Server (in `/config/packages/`) | Helpers and scripts |
| `dashboard.yaml` | HA Server (pasted into HA UI) | Dashboard layout |
| `skylight.yaml` theme | HA Server (in `/config/themes/`) | Optional visual theme |
| Raspberry Pi OS | Kiosk Pi | Operating system for display |
| Chromium browser | Kiosk Pi | Displays the HA dashboard |
| Kiosk script | Kiosk Pi | Launches browser in full-screen mode |

---

## File Reference

| File in This Repo | Where It Goes | Purpose |
|-------------------|--------------|---------|
| `packages/family_calendar.yaml` | HA Server: `/config/packages/` | All helpers, scripts, automations |
| `dashboard.yaml` | HA Server: pasted into dashboard raw editor | Dashboard layout |
| `themes/skylight.yaml` | HA Server: `/config/themes/` | Optional serif font theme |
| Your background image | HA Server: `/config/www/` | Dashboard background (accessed via `/local/`) |
