# TRMNL iOS Reminders Plugin — Plan

## Overview

A TRMNL private plugin that displays **today's top to-do items** on an e-ink display.
The iOS Companion app POSTs reminder data to a TRMNL webhook each morning. The plugin renders
a flat list sorted by urgency: overdue items first (oldest first), then items due today.
Future items are hidden. Reminders from all lists are treated as one flat pool.

**Constraints:**
- 800×480 px display, 4-shade grayscale (black, dark gray, light gray, white)
- Liquid templating (Shopify-flavored) with TRMNL's CSS framework
- Data arrives via webhook as `{{ reminders }}` (array) and `{{ trmnl }}` (device/user context)
- Private plugin model: markup lives in TRMNL's WYSIWYG editor (no hosted server required)

---

## Data Reference

### `{{ reminders }}` — array of reminder objects

Schema is consistent across all items. Only `due_date` is optional.

| Field                 | Type   | Present  | Notes                                      |
|-----------------------|--------|----------|--------------------------------------------|
| `title`               | string | always   | Reminder text                              |
| `notes`               | string | always   | Body text; often empty string              |
| `priority`            | int    | always   | 0 = none, 1 = high (only two values seen)  |
| `list_name`           | string | always   | e.g. "Family", "Home Maintenance"          |
| `due_date`            | string | optional | ISO 8601 UTC timestamp                     |
| `reminder_identifier` | string | always   | UUID, stable across syncs                  |

### `{{ trmnl }}` — device/user context

| Field                           | Notes                              |
|---------------------------------|------------------------------------|
| `trmnl.user.first_name`         | "Jonathan"                         |
| `trmnl.user.time_zone_iana`     | "America/New_York"                 |
| `trmnl.user.utc_offset`         | Seconds offset, e.g. -14400 (EDT) |
| `trmnl.device.width/height`     | 800 / 480                          |
| `trmnl.system.timestamp_utc`    | Unix timestamp of last refresh     |
| `trmnl.plugin_settings.dark_mode` | "yes" / "no"                     |

---

## Task List

### Phase 1 — Local Dev Environment

- [x] **1.1** Create plugin scaffold: `src/full.liquid`, `src/shared.liquid`, `src/settings.yml`,
      `.trmnlp.yml` with generic fixture data, `bin/serve.ps1` Docker runner
- [x] **1.2** Pull the trmnlp Docker image and confirm `bin/serve.ps1` renders the plugin at
      `localhost:4567`

### Phase 2 — Layout Design

- [x] **2.1** Design: flat list sorted ascending by `due_date`, filtered to `d <= today` only.
      No sections. Fuzzy age labels: "Today", "1 day overdue", "N days overdue".
- [x] **2.2** Visual encoding: bold titles for overdue, `!` prefix for priority-1 items,
      two-line rows (title + meta), Inter font, open circle checkboxes, iOS Reminders aesthetic
- [x] **2.3** Timezone fix: compute `today` as `timestamp_utc + utc_offset` formatted in UTC
      to avoid off-by-one errors in negative-offset timezones in the evening

### Phase 3 — Markup Implementation

- [x] **3.1** `src/full.liquid` — 8 rows, 580px centered container, age label + list name
- [x] **3.2** `src/half_horizontal.liquid` — 8 rows, two-column CSS grid
      (`grid-auto-flow: column`, `grid-template-rows: repeat(4, auto)`)
- [x] **3.3** `src/half_vertical.liquid` — 8 rows, left-aligned via `align-self: flex-start`,
      list name omitted from meta
- [x] **3.4** `src/quadrant.liquid` — 6 rows, title only, compact padding
- [x] **3.5** `src/shared.liquid` — common styles extracted (font import, row, circle, meta)
- [x] **3.6** Fix `flex--col` centering issue: TRMNL framework sets `align-items: center` on
      `.flex--col`; narrow layouts override with `align-self: flex-start` on `.list-wrap`

### Phase 4 — Testing & Polish

- [x] **4.1** HTML preview verified for all four layouts in trmnlp
- [ ] **4.2** PNG render mode — verify grayscale rendering on actual device
- [ ] **4.3** Edge cases to watch on real hardware:
  - Google Fonts loading (headless renderer may not have network access — fallback to system sans-serif if needed)
  - Row overflow if many items are overdue
  - Very long task titles

### Phase 5 — Deployment

- [ ] **5.1** Authenticate with TRMNL: `docker run ... trmnl/trmnlp login`
- [ ] **5.2** Push markup: `docker run ... trmnl/trmnlp push`
- [ ] **5.3** Confirm the plugin renders on-device after the iOS Companion app next syncs

---

## Open Questions

- Does the Google Fonts `<link>` load successfully in the TRMNL headless PNG renderer, or does
  it fall back to system sans-serif? If it falls back, consider inlining Inter as a base64
  `@font-face` in `shared.liquid`.
- When the Companion app syncs, does it send all incomplete reminders or only a subset?
  If lists are large, the 8-item cap may hide important items without any indication.
