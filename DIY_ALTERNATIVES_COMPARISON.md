# DIY Skylight Calendar Alternatives: Research & Comparison (Free/No Subscription Only)

## Overview

There are **4 viable free approaches** to building a family calendar display with no ongoing subscription costs. All use a Raspberry Pi as the display device. Below is a detailed comparison including Pi 5-specific setup differences.

---

## 1. Home Assistant + Raspberry Pi Kiosk (Current Approach)

**What it is:** Full smart-home platform running a custom calendar dashboard, displayed on a separate Pi in kiosk mode.

### Pros
- Completely free (no subscriptions)
- Maximum customization — full control over layout, colors, fonts, behavior
- Can add non-calendar features (weather, smart home controls, automations)
- Self-hosted, fully private — no data leaves your network
- Active community with ongoing development
- **Touch support for adding events directly on the display**
- Supports Google Calendar, iCal, local calendars

### Cons
- **Most complex setup by far** — requires two devices, YAML configuration, HACS, multiple custom cards
- Steep learning curve for Home Assistant beginners
- YAML syntax errors can be frustrating to debug
- Requires ongoing maintenance (HA updates, breaking changes in custom components)
- Google Calendar integration requires navigating Google Developer Console (OAuth)

### Setup Complexity: HIGH
- **Hardware:** 2 Raspberry Pis (or Pi + mini PC), 2 SD cards, monitor, cables
- **Software steps:** ~30+ distinct configuration steps across HA install, HACS, 7 custom cards, packages directory, dashboard YAML, theme, kiosk setup
- **Skills needed:** Basic YAML editing, comfort with web interfaces, patience for debugging
- **Realistic effort for a beginner:** A full weekend with troubleshooting; experienced users might manage it in an evening

### Pi 5 Differences
| Aspect | Pi 4 | Pi 5 |
|--------|------|------|
| **HA Server Pi** | Works with Home Assistant OS | Works with Home Assistant OS (same) |
| **Kiosk Pi display server** | X11 (LXDE) — autostart in `/etc/xdg/lxsession/LXDE-pi/autostart` | Wayland (labwc on Trixie) — autostart in `~/.config/labwc/autostart` |
| **Chromium flags** | Standard flags | Add `--ozone-platform=wayland` and `--password-store=basic` |
| **Cursor hiding** | `unclutter` package works | `unclutter` may not work under Wayland; alternatives needed |
| **HDMI disconnect** | Browser stays fullscreen | Browser may exit fullscreen when monitor power-cycles |
| **Keyring prompt** | Not an issue | May get keyring unlock dialog on boot — suppress with `--password-store=basic` |

**Pi 5 kiosk autostart example (`~/.config/labwc/autostart`):**
```bash
chromium-browser --ozone-platform=wayland --password-store=basic --noerrdialogs --disable-infobars --kiosk http://YOUR_HA_IP:8123/family-calendar/family-calendar
```

---

## 2. MagicMirror on Raspberry Pi

**What it is:** Open-source modular display platform (Node.js/Electron) that runs directly on a single Pi. Originally designed for smart mirrors but works great as a wall-mounted calendar display.

### Pros
- Completely free and open-source
- Large ecosystem of community modules (hundreds of add-ons)
- **Single device needed** (one Pi does everything — no separate server)
- Supports iCal feeds, Google Calendar (private via OAuth), weather, photos, clocks
- Can look very similar to Skylight with the right modules (MMM-CalendarExt3) and CSS
- Active community forum
- MagicMirrorOS available — a dedicated Pi distro that boots directly into MagicMirror

### Cons
- **Display-only** — no built-in way to add events from the screen
- Requires editing JavaScript config files (`config.js`) — actual JS object syntax, not YAML
- Google Calendar private access requires OAuth setup (similar complexity to HA's)
- Customizing the look beyond defaults requires CSS knowledge
- Some community modules become unmaintained over time
- Debugging module issues requires reading Node.js error logs

### Setup Complexity: MEDIUM
- **Hardware:** 1 Raspberry Pi, SD card, monitor, cables
- **Software steps:** ~10-15 steps: flash OS (or MagicMirrorOS), install MagicMirror (automated script available), edit `config.js` to add calendar URLs, optionally install additional modules, configure PM2 for auto-start
- **Skills needed:** Basic text editing, comfort with terminal commands, understanding of JSON-like object syntax
- **Realistic effort for a beginner:** An afternoon for basic setup with public iCal calendars; add a few more hours for private Google Calendar OAuth and visual customization

### Key Modules for a Skylight-like Display
| Module | Purpose |
|--------|---------|
| `calendar` (built-in) | Basic event list from iCal feeds |
| `MMM-CalendarExt3` | Week/month grid view (closest to Skylight layout) |
| `MMM-GoogleCalendar` | Private Google Calendar access without making calendars public |
| `MMM-WeatherChart` | Weather forecast display |
| `MMM-GooglePhotos` | Photo slideshow background |

### Pi 5 Differences
| Aspect | Pi 4 | Pi 5 |
|--------|------|------|
| **MagicMirrorOS** | Supported (32-bit and 64-bit) | Supported (32-bit and 64-bit) |
| **Start command** | `npm run start` | `npm run start:wayland` (required for Wayland) |
| **Display server** | X11 or Wayland depending on RAM | Wayland by default (labwc on Trixie) |
| **Module compatibility** | All modules work | Some screen on/off modules affected by Wayland |
| **Node.js version** | Must match MagicMirror requirements | Same — but ensure Node 18+ (use NVM if needed) |
| **Monitor power control** | `vcgencmd` or `xrandr` | Different commands for Wayland; `wlr-randr` may be needed |

**Pi 5 note:** If modules that control screen power (sleep/wake on schedule) don't work, it's likely a Wayland compatibility issue. Check the module's GitHub issues for Wayland fixes, or switch to X11 via `sudo raspi-config > Advanced Options > Wayland > X11`.

---

## 3. FullPageOS + Web Calendar

**What it is:** A Raspberry Pi OS distro that boots directly into a full-screen Chromium browser showing any URL. Point it at Google Calendar's web interface or any web-based calendar.

### Pros
- Free
- Extremely simple concept — just a browser showing a webpage
- Works with ANY web-based calendar (Google Calendar, Outlook web, Nextcloud, etc.)
- Single device, almost no configuration
- Can rotate between multiple URLs with FullPageDashboard
- No coding required

### Cons
- **NOT compatible with Raspberry Pi 5** (last stable build: March 2022)
- Display-only — relies entirely on the target website's UI
- Google Calendar's web UI isn't optimized for wall displays (small text, navigation chrome)
- If the calendar session expires, you're staring at a login page
- No customization of what you're displaying
- Calendar web UIs aren't designed for touch-free wall viewing
- Looks like a webpage on a screen, not a purpose-built calendar

### Setup Complexity: LOW
- **Hardware:** 1 Raspberry Pi (2, 3, or 4 — **NOT 5**), SD card, monitor, cables
- **Software steps:** ~4-5 steps: flash FullPageOS via Pi Imager, edit `fullpageos.txt` with your URL, configure WiFi in `fullpageos-wpa-supplicant.txt`, boot
- **Skills needed:** Ability to edit a text file on an SD card
- **Realistic effort for a beginner:** 15-30 minutes if you have a suitable URL ready. The challenge is finding a calendar view that looks good full-screen.

### Pi 5 Differences
**FullPageOS does NOT support Pi 5.** The last stable release (2022-03-07) predates the Pi 5. Nightly builds have also had issues.

**Pi 5 alternative:** Instead of FullPageOS, manually set up Raspberry Pi OS with Chromium in kiosk mode:
1. Flash standard Raspberry Pi OS Lite (64-bit)
2. Install minimal GUI: `sudo apt install --no-install-recommends xserver-xorg x11-xserver-utils xinit chromium-browser`
   - Or on Trixie: install labwc and chromium
3. Create autostart entry at `~/.config/labwc/autostart`:
```bash
chromium-browser --ozone-platform=wayland --password-store=basic --noerrdialogs --disable-infobars --kiosk YOUR_URL_HERE
```
4. This adds ~10 minutes of setup compared to FullPageOS's edit-one-file approach

---

## 4. Custom Web App (Self-Built with FullCalendar.io or similar)

**What it is:** Build your own calendar web page using HTML/CSS/JavaScript with a library like FullCalendar.io, pull events from iCal feeds or Google Calendar API, then display it on a Pi in kiosk mode.

### Pros
- Completely free
- Total design control — make it look exactly like Skylight
- No subscriptions, no cloud dependency (if using iCal feeds)
- Single device (Pi serves and displays the app)
- Can build "add event" functionality if you want
- Transferable web development skills
- Can be as simple or complex as you want

### Cons
- **Requires web development skills** (HTML, CSS, JavaScript minimum)
- Must build and maintain everything yourself
- Google Calendar API still requires OAuth setup for private calendars
- No community support for your specific implementation
- Can become a never-ending project (scope creep)
- Security considerations if you add write functionality
- You're the only maintainer

### Setup Complexity: MEDIUM-HIGH (depends on web dev experience)
- **Hardware:** 1 Raspberry Pi, SD card, monitor, cables
- **Software steps:** Build a web app (static HTML + JS fetching iCal, or a full React/Vue app), host it locally on the Pi (simple HTTP server or Node.js), configure Pi to display it in kiosk mode
- **Skills needed:** HTML, CSS, JavaScript. Optionally: Node.js/Python for backend, API knowledge for calendar integration
- **Realistic effort for a beginner (non-developer):** Not recommended without web development experience
- **Realistic effort for a developer:** A day for a basic read-only calendar pulling iCal feeds. A week+ for a polished, interactive display with add-event functionality.

### Key Libraries
| Library | What It Does |
|---------|-------------|
| [FullCalendar.io](https://fullcalendar.io/) | Full-featured calendar UI (month/week/day views), free open-source |
| [ical.js](https://github.com/kewisch/ical.js) | Parse iCal feeds in the browser |
| React Big Calendar | React-based calendar component |
| Open Web Calendar | Embed iCal sources as a customizable web widget |

### Pi 5 Differences
Same as the kiosk setup notes above — the web app itself doesn't care what Pi it runs on, but the kiosk configuration differs:

| Aspect | Pi 4 | Pi 5 |
|--------|------|------|
| **Serving the app** | Same (Node.js, Python http.server, nginx, etc.) | Same |
| **Kiosk autostart** | LXDE autostart or `.xinitrc` | labwc autostart (`~/.config/labwc/autostart`) |
| **Chromium flags** | Standard | Add `--ozone-platform=wayland --password-store=basic` |

---

## Comparison Matrix (Free Options Only)

| Approach | Devices | Complexity | Customization | Add Events from Display | Privacy | Pi 5 Support |
|----------|---------|-----------|---------------|------------------------|---------|-------------|
| **Home Assistant** | 2 (server + kiosk) | HIGH | Unlimited | Yes | Full (local) | Yes (different kiosk config) |
| **MagicMirror** | 1 | MEDIUM | High (CSS + modules) | No (display-only) | Full (local) | Yes (use `start:wayland`) |
| **FullPageOS** | 1 | LOW | None (raw website) | Depends on site | Depends | **No** (manual kiosk instead) |
| **Custom Web App** | 1 | MEDIUM-HIGH | Unlimited | If you build it | Full (local) | Yes (different kiosk config) |

---

## Pi 5 Universal Notes

These apply regardless of which approach you choose for the kiosk display:

1. **Wayland is the default** — Pi 5 on modern Raspberry Pi OS (Bookworm/Trixie) uses Wayland with the labwc compositor. Old X11/LXDE autostart methods won't work.

2. **Autostart location changed** — Use `~/.config/labwc/autostart` (not the old `/etc/xdg/lxsession/LXDE-pi/autostart`)

3. **Chromium needs extra flags:**
   - `--ozone-platform=wayland` (required for Wayland)
   - `--password-store=basic` (prevents keyring unlock dialog)
   - `--window-size=1920,1080` (if browser only fills half the screen)

4. **Screen power management is different** — Traditional tools like `xset dpms` and `xrandr` don't work under Wayland. Use `wlr-randr` or compositor-level settings instead.

5. **Cursor hiding is harder** — The `unclutter` package may not work. Alternatives: hide via CSS (`cursor: none` on the page body), or use labwc configuration.

6. **You can always fall back to X11** — Run `sudo raspi-config > Advanced Options > Wayland > X11` to switch back. You'll lose some performance and Pi Connect support, but all the old tutorials will work again.

7. **HDMI power cycling** — If the monitor turns off and back on, Wayland may not restore fullscreen. Consider a cron job or systemd timer to periodically restart the browser, or use a smart plug with a fixed schedule instead of the monitor's auto-sleep.

---

## Recommendations

- **"I want maximum features (add events, filters, weather) and don't mind complexity"** → Home Assistant (current approach)
- **"I want free + self-hosted with moderate effort on a single device"** → MagicMirror
- **"I'm a web developer and want full design control"** → Custom web app with FullCalendar.io
- **"I just want the simplest possible thing on a Pi 4 or older"** → FullPageOS (or manual kiosk on Pi 5)

---

## Sources

- [Home Assistant Community: DIY Family Calendar](https://community.home-assistant.io/t/diy-family-calendar-skylight/844830)
- [XDA: I built a Skylight calendar clone](https://www.xda-developers.com/i-built-a-skylight-calendar-clone-thanks-to-home-assistant/)
- [MagicMirror Forum: Skylight Calendar DIY Build](https://forum.magicmirror.builders/topic/19688/skylight-calendar-diy-build)
- [MagicMirror Docs: Requirements](https://docs.magicmirror.builders/getting-started/requirements.html)
- [MagicMirror Forum: Pi 5 Compatibility](https://forum.magicmirror.builders/topic/18832/compatibility-with-raspberry-pi-5-right-now-and-in-the-future)
- [RaspberryTips: MagicMirror Guide](https://raspberrytips.com/magic-mirror-guide/)
- [RaspberryTips: FullPageOS](https://raspberrytips.com/fullpageos-raspberry-pi/)
- [Raspberry Pi Forums: Pi 5 Kiosk Mode (Bookworm Lite, 2025)](https://forums.raspberrypi.com/viewtopic.php?t=389880)
- [GitHub: RasPi5 Kiosk Setup](https://github.com/TheGeekOfAllTrades/RasPi5_Kiosk)
- [Pi My Life Up: HA Kiosk](https://pimylifeup.com/raspberry-pi-home-assistant-kiosk/)
- [Raspberry Pi Official: Kiosk Mode Tutorial](https://www.raspberrypi.com/tutorials/how-to-use-a-raspberry-pi-in-kiosk-mode/)
- [Ben Swift: Automated RPi Web Kiosk Setup in 2025](https://benswift.me/blog/2025/07/16/automated-rpi-web-kiosk-setup-in-2025)
- [Luca Scalzotto: Chromium Kiosk on Pi](https://www.scalzotto.nl/posts/raspberry-pi-kiosk/)
- [Scott Hanselman: Wall-Mounted Family Calendar](https://www.hanselman.com/blog/how-to-build-a-wall-mounted-family-calendar-and-dashboard-with-a-raspberry-pi-and-cheap-monitor)
- [FullCalendar.io](https://fullcalendar.io/)
