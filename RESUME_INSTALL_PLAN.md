# Resume Interrupted Skylight Calendar Install (Phase 3)

## Context

The Raspberry Pi boots to a **text console** (`tty1`) at a shell prompt showing
`Completed socket interaction for boot stage final`, logged in automatically as
`yaya` on host `calendar-kiosk` (Debian 13). It is **not** reaching the graphical
Chromium kiosk.

This is the documented failure mode in `RASPBERRY_PI_SETUP_GUIDE.md:534`
("Pi Restarted During Phase 3 — Stuck on Boot Screen"): a reboot landed mid
**Phase 3 (Kiosk Mode Setup)** before LightDM/Chromium were fully configured, so
the system falls back to a text console.

- **Phase 1** (flash, hostname, auto-login): done — confirmed by the boot banner.
- **Phase 3**: partially complete — need to find the first incomplete step.

All Phase 3 commands are idempotent, so re-running an already-completed step is safe.

## Step 0 — (Optional) Recover SSH access from the Mac

The Pi is already auto-logged in at the console, so SSH is only needed for
convenience (running the steps below from the Mac instead of at the Pi). The
local account password was set during imaging (Phase 1) and isn't recoverable
remotely — but it can be reset/replaced from the console you're already at.

First, make sure the SSH server is running (no-op if already enabled):
```bash
sudo systemctl enable --now ssh
```

**Option A — reset the password** (RPi OS first user has passwordless sudo):
```bash
sudo passwd yaya     # sets a new password; then: ssh yaya@calendar-kiosk.local
```

**Option B — key-based auth, no password** (preferred; guide line 464):
```bash
# On the Mac:
ls ~/.ssh/id_ed25519.pub 2>/dev/null || ssh-keygen -t ed25519 -C "ztangpccam@gmail.com"
cat ~/.ssh/id_ed25519.pub          # copy the printed key

# On the Pi:
mkdir -p ~/.ssh && chmod 700 ~/.ssh
echo "PASTE_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

If `calendar-kiosk.local` doesn't resolve, get the LAN IP with `hostname -I` on
the Pi and use `ssh yaya@<ip>`.

## Step 0b — (Optional) Enlarge the console font while working at the Pi

The text console (`tty1`) starts with a tiny font. To make the setup steps
easier to read at the Pi, bump the console font (this only affects the text
console — once Phase 3 finishes and the Pi boots into the Chromium kiosk, the
display is sized by the kiosk resolution, not this).

**Persistent (survives reboot):**
```bash
sudo dpkg-reconfigure console-setup
# Encoding: UTF-8
# Character set: Guess optimal character set
# Font: Terminus
# Size: 16x32  (or Terminus 12x24)
sudo setupcon   # apply immediately if it didn't already
```
Settings are stored in `/etc/default/console-setup`
(`FONTFACE="Terminus"`, `FONTSIZE="16x32"`).

**Temporary (just this session, resets on reboot):**
```bash
sudo setfont /usr/share/consolefonts/Lat15-TerminusBold32x16.psf.gz
```

## Step A — Diagnose (run on the Pi, read-only)

You are already at the Pi's prompt. Run these to see how far Phase 3 got:

```bash
dpkg -l | grep -E "lightdm|chromium|onboard|xserver-xorg"   # Step 1: packages
grep autologin-user /etc/lightdm/lightdm.conf               # Step 2: auto-login
grep -E "display-setup-script|xserver-command" /etc/lightdm/lightdm.conf  # Steps 3-4
ls /opt/kiosk/kiosk.sh                                       # Step 5: launch script
ls /lib/systemd/system/kiosk.service                         # Step 6: systemd unit
systemctl is-enabled kiosk 2>/dev/null                       # Step 7: service enabled
```

Interpretation → first failing/empty check = the step to resume from:
- packages missing → resume at **Phase 3, Step 1**
- packages OK but no `autologin-user` line → **Step 2**
- no `display-setup-script` / `xserver-command` → **Steps 3–4**
- `/opt/kiosk/kiosk.sh` missing → **Step 5**
- `kiosk.service` missing → **Step 6**
- service not enabled → **Step 7**

## Step B — Resume Phase 3 from the first incomplete step

Follow `RASPBERRY_PI_SETUP_GUIDE.md` Phase 3 (lines 172–323) in order from that
step. Substitution reminders:
- `YOUR_USERNAME` → `yaya` (seen in the boot banner)
- `RESOLUTION` → your display's native resolution (default `1920x1080`)
- `YOUR_HOME_ASSISTANT_URL` in `/opt/kiosk/kiosk.sh` → your HA host/IP + dashboard path

## Step C — Reboot and verify

```bash
sudo systemctl daemon-reload
sudo systemctl enable kiosk
sudo reboot
```

Success = Pi boots straight into full-screen Chromium showing the HA calendar
dashboard (no text console). If it still drops to console, check
`sudo systemctl status kiosk` and `lightdm` logs per the Troubleshooting table.

## Notes
- I (Claude) am on your Mac and cannot reach the Pi directly. If you want me to
  help interpret the diagnostic output, paste the results of Step A here, or set
  up SSH (`ssh yaya@calendar-kiosk.local`) so you can run them from this machine.
