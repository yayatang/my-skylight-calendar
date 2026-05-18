
<div align="right">
  <details>
    <summary >🌐 Language</summary>
    <div>
      <div align="center">
        <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=en">English</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=zh-CN">简体中文</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=zh-TW">繁體中文</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=ja">日本語</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=ko">한국어</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=hi">हिन्दी</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=th">ไทย</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=fr">Français</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=de">Deutsch</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=es">Español</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=it">Italiano</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=ru">Русский</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=pt">Português</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=nl">Nederlands</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=pl">Polski</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=ar">العربية</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=fa">فارسی</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=tr">Türkçe</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=vi">Tiếng Việt</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=id">Bahasa Indonesia</a>
        | <a href="https://openaitx.github.io/view.html?user=mohesles&project=my-skylight-calendar&lang=as">অসমীয়া</a>
      </div>
    </div>
  </details>
</div>

# DIY Smart Home Family Calendar (Skylight Clone)

![Sklylight calendar](assets/main_view.jpeg)
![DIY Skylight](assets/sky2.png)

## 📖 Introduction

My wife has been recently bombarded in social media with ads for smart home calendars (Skylight, Cozyla, Hearth) and was ready to spend over $300 on one. Before giving her the green light, I asked for a chance to research them.

I realized most offered similar functionality but differed significantly in price. Most importantly, I didn't see any outstanding feature that I couldn't implement in **Home Assistant**.

**The Goal:** A WAF-approved (Wife Acceptance Factor), countertop-friendly touchscreen calendar that integrates deep into our smart home without monthly fees.

## 💡 Why DIY?

Choosing the DIY route with Home Assistant provided several benefits over buying a Skylight/Hearth display:

* **No Monthly Fees:** Avoids subscriptions for "premium" features.
* **Seamless Integration:** It talks to our lights, chores (Grocy), and presence sensors.
* **Old Hardware:** Repurposed a Mini PC and a standard monitor.
* **Privacy:** No vendor lock-in or risk of the company shutting down.

## 🛠 Hardware Selection

This is currently built to show the dashboard on any HD (1920x1080) display.

In my case, the requirement was for it to "look like" skylight, be touchscreen, be countertop, possibility to move it to different locations. Therefore I went with the hardware described below.
Nevertheless, you case might be different and will need you to adjust it as needed, for example if you want to display it on a tablet or something else.

The hardware I originally used I chose based on what I mentioned above plus with the hope to be able to extend functionality using the webcam, speaker and microphone. Currently I would probably build it differently now in hindsight, since I havent had time to address these additional hardware ideas.

* **Monitor:** [HP Engage 15-inch Touchscreen](https://computers.woot.com/offers/hp-engage-16t-fhd-monitor). I chose this over generic portable monitors because it includes a built-in **Speaker, Webcam, and Microphone**, allowing for future voice control or video calls.
* **Computer:** An old Mini PC (NUC/Tiny PC) running Windows/Linux in Kiosk mode, or a Raspberry Pi 4.


## ✨ Features

* **Family-wide & Individual Views:** Toggle specific family members' calendars on/off.
* **Two-way Sync:** Edit events on the screen or on our phones (Google Calendar).
* **"Add Event" Popup:** A custom UI to add events to specific calendars directly from the screen.
* **Weather & Date:** Beautiful, glanceable header.
* **Responsive:** Automatically adjusts day-count based on screen width (Mobile vs Desktop).

---

## ⚙️ Setup

This project has two parts: the **Home Assistant server** (where the calendar lives) and the **kiosk display** (the screen your family looks at). They run on different devices.

Pick the guide that matches what you need:

| Guide | What it covers |
|-------|----------------|
| **[Home Assistant Setup Guide](HOME_ASSISTANT_SETUP_GUIDE.md)** | Installing Home Assistant from scratch, HACS, calendars, weather, the dashboard, and the theme. Start here if you don't already have HA running. |
| **[Raspberry Pi Setup Guide](RASPBERRY_PI_SETUP_GUIDE.md)** | Turning a Raspberry Pi into a full-screen kiosk pointed at your dashboard. Use after the dashboard is working. |
| **[DIY Alternatives Comparison](DIY_ALTERNATIVES_COMPARISON.md)** | If you're not committed to Home Assistant, this compares MagicMirror, FullPageOS, and a custom web app. |

### Files in this repo

| File | Purpose |
|------|---------|
| [`packages/family_calendar.yaml`](packages/family_calendar.yaml) | All helpers, scripts, and automations — drop in `<config>/packages/` |
| [`dashboard.yaml`](dashboard.yaml) | Dashboard layout — paste into the HA dashboard's raw YAML editor |
| [`themes/skylight.yaml`](themes/skylight.yaml) | Optional Ovo-font theme — drop in `<config>/themes/` |
| `calbackgrd.png` | Optional background image — upload to `<config>/www/` |

---

## 📐 How It Works (Under the Hood)

### Filter Logic

The `week-planner-card` does not natively support hiding specific calendars on the fly. To solve this, I used **Input Texts** acting as Regex filters.

* When you click a person's button, it toggles their filter between `.*` (Show everything) and `^$` (Show nothing).
* `config-template-card` injects these variables into the calendar card dynamically.

### Event Creation Script

The "Add Event" popup uses a single script that handles logic for multiple people and event types (All Day vs Timed).

```yaml
# Simplified Logic Example
target_calendar: "{{ calendar_map.get(states('input_select.calendar_select')) }}"

choose:
  - conditions: "All Day Event is ON"
    action: calendar.create_event (start_date, end_date)
  - conditions: "All Day Event is OFF"
    action: calendar.create_event (start_date_time, end_date_time)
```

## NOTES

My original post was just to give a high level overview of how to do it and allow people to adjust code to make it work in their specific scenarios.

In particular I did this because every display and need is different. I can't develop for all potential sizes of displays, dashboards, etc. So it is built to work in the display I mentioned or any (1920x1080) but should be editable for others.

Talking about display, I originally suggested that one because it was on sale at Woot and was a very economic way to get a touchscreen display at the time. This might not be the case now, so use whatever display works for you. Tablet, touchscreen, phone, whatever. The main thing youll need to edit is the dashboard.

