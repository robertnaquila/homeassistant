# HA Dashboard — implementation

A standalone, buildless implementation of the Claude Design **HA Dashboard**
prototypes (`project/HA Dashboard.dc.html` landscape + `project/HA Dashboard portrait.dc.html`).

Everything lives in a single self-contained file: **`index.html`**. No build
step, no dependencies, no framework — just open it in a browser.

## What it is

A futuristic dark Home Assistant dashboard (electric-blue `#00b3ff`) built for a
wall-mounted **landscape** tablet kiosk and a Fire-tablet **portrait** kiosk.

Tabs: **Overview · Devices · Security · Family · Screen · Park**

- **Overview** — holographic BYD M6 hero with tap-to-cycle tyre pressure
  (kPa → bar → psi), battery/charge, Who's-Home avatars (tap → location map
  dialog), kids screen-time line graphs, live
  camera strip (tap → enlarge), calendar (Month/Week/Day with ‹ › navigation,
  fed by your HA `calendar.*` entities; the **tune** icon opens a dialog to
  show/hide individual calendars), Climate/Lights "active at a glance" tiles,
  NAS/Air/Carpark tiles, and an Away/Sleep scenes dock.
- **Devices** — AC zones, fans & purifiers (portrait fans get a speed slider +
  oscillate toggle; the Dining Sensibo fan has neither), indoor air quality, and
  a Lighting column. Toilet/bright lights are mutually exclusive, matching the
  HA automation.
- **Security** — 3 Synology camera feeds + gate/armed status, then the Synology
  NAS gauges (CPU/RAM/temp/volume), drive temps, storage, network throughput,
  and a Claude AI usage panel.
- **Family** — people & presence cards (tap → map) and kids screen-time
  breakdown cards (tap → weekly line-graph dialog).
- **Screen / Park** — live iframes to the NAS web apps.

Active devices glow blue with bright fill and animate (fans spin, AC pulses,
charging shimmers, alerts pulse); off devices are dim outlines.

## Layout selection

The layout auto-switches on device **orientation** (portrait vs landscape) and
re-evaluates on rotation. Force one with a query param:

```
index.html?layout=landscape
index.html?layout=portrait
index.html?tab=security        # also deep-links a starting tab
```

## Running it / live demo

Open `index.html` in any modern browser. Out of the box it runs as a **live
demo** using mock state that matches the prototype exactly (toggles flip
locally, scenes apply, dialogs open). A `DEMO` chip shows in the top bar.

## Connecting to Home Assistant

Tap the **DEMO** chip (top bar) → enter your HA **Base URL** and a
**Long-Lived Access Token** (HA → Profile → Security → Long-lived access
tokens). Connection details are stored in this browser's `localStorage`.

Once connected:
- Entity states stream live over the HA **WebSocket API** (`get_states` +
  `subscribe_events`), so the UI reflects real device state.
- Tapping a device calls the matching HA service (`light.turn_on/off`,
  `switch.turn_on/off`, `fan.turn_on/off`, `climate.turn_on/off`,
  `media_player.turn_on/off`). Updates are optimistic and reconciled by the
  state stream.
- Scenes (Away / Sleep) issue the corresponding per-entity service calls.
- Camera tiles render the real signed `entity_picture` snapshots, refreshed
  periodically.
- The calendar discovers every `calendar.*` entity and pulls events for the
  visible Month/Week/Day window from HA's REST calendar API
  (`/api/calendars/<entity_id>?start=&end=`), each calendar drawn in its own
  colour. The **tune** icon on the calendar card opens a dialog to toggle
  individual calendars on/off; the hidden set is stored in `localStorage`
  (`ha_cal_hidden`). The REST calls are same-origin when the dashboard is
  served from HA's `config/www/`; for cross-origin connections add the
  dashboard origin to HA's `http.cors_allowed_origins`.

You can also pass config via URL (handy for kiosk provisioning):

```
index.html?hass=http://homeassistant.local:8123&token=YOUR_LONG_LIVED_TOKEN
```

The entity map (logical key → HA entity_id) is defined near the top of the
script in `index.html` (`DEVICES`, `SENS`, `KID_SENS`), derived from the
exported entity list. Adjust there if any entity_id differs on your instance.

## Deploying into Home Assistant

Copy `index.html` into HA's `config/www/` folder, e.g.
`config/www/ha-dashboard/index.html`, then add a **Webpage** dashboard / iframe
panel pointing at `/local/ha-dashboard/index.html`. Run the tablet browser in
kiosk mode at 1280×800 (landscape) or 820×1180 (portrait).

> The Screen Time and Carpark tabs embed `screentime.robertnaquila.synology.me`
> and `parking.robertnaquila.synology.me` — they render on your home network.
