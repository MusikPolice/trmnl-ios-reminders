# TRMNL iOS Reminders Plugin — Plan

## Overview

Build a TRMNL private plugin that displays **today's top to-do items** on an 800×480, 2-bit
grayscale e-ink display. The iOS Companion app POSTs reminder data to a webhook; this plugin
surfaces the most urgent items: overdue tasks first, then due today, then due this week, then
any high-priority undated items. Reminders from all lists (Family, Home Maintenance, etc.) are
treated as one flat pool and ranked by urgency alone.

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
| `trmnl.device.width/height`     | 800 / 480                          |
| `trmnl.device.percent_charged`  | Battery level                      |
| `trmnl.system.timestamp_utc`    | Unix timestamp of last refresh     |
| `trmnl.plugin_settings.dark_mode` | "yes" / "no"                     |

---

## Task List

### Phase 1 — Local Dev Environment

- [x] **1.1** Create plugin scaffold: `src/full.liquid`, `src/shared.liquid`, `src/settings.yml`,
      `.trmnlp.yml` with generic fixture data, `bin/serve.ps1` Docker runner
- [ ] **1.2** Pull the trmnlp Docker image and confirm `bin/serve.ps1` renders the plugin at
      `localhost:4567` — run from the repo root in PowerShell

### Phase 2 — Layout Design

- [x] **2.1** Design decided: four sections in urgency order — **Overdue**, **Due Today**,
      **This Week**, **High Priority (no date)**; items beyond "this week" are omitted
      unless high priority
- [x] **2.2** Visual encoding: bold titles for overdue, `!` prefix for priority-1 items,
      date column right-aligned, list name as small trailing label
- [ ] **2.3** Verify layout at actual e-ink resolution using PNG render mode in trmnlp

### Phase 3 — Markup Implementation

- [x] **3.1** First draft of `src/full.liquid` written — four urgency sections with Liquid
      date comparison, CSS `text-overflow: ellipsis` for long titles, priority `!` prefix,
      list name trailing label, and empty-state message
- [ ] **3.2** Iterate on layout after first trmnlp preview — adjust font sizes, row density,
      and TRMNL framework class usage based on actual rendered output
- [ ] **3.3** Handle edge cases once layout is stable:
  - Very long overdue list — consider capping at N items with a count overflow line
  - Note indicator if `reminder.notes != ""`

### Phase 4 — Testing & Polish

- [ ] **4.1** Preview in `trmnlp serve` (HTML mode first, then PNG mode for accurate rendering)
- [ ] **4.2** Verify grayscale rendering — ensure no color-dependent styling
- [ ] **4.3** Test with edge-case fixture data:
  - All items overdue
  - Mix of priority 0 and 1
  - Items with and without notes
  - Items with no due dates
- [ ] **4.4** Check readability at e-ink font sizes (TRMNL framework handles this, but verify)

### Phase 5 — Deployment

- [ ] **5.1** Authenticate with TRMNL: `trmnlp login` (requires API key from trmnl.com settings)
- [ ] **5.2** Link this repo to the private plugin instance: `trmnlp push`
      — OR — copy the rendered markup from `trmnlp serve` into the TRMNL WYSIWYG editor
- [ ] **5.3** Confirm the plugin renders on-device after the iOS Companion app next syncs

---

## Open Questions

- Is `due_date` always UTC midnight for date-only reminders, or does it carry a real time?
  Looking at the sample data, most are midnight-ish UTC but "Flu shots" has a 1pm UTC time.
  Decide whether to show time-of-day on same-day items.
- When the Companion app syncs, does it send *all* incomplete reminders or only a subset?
  This affects whether we need to worry about very large lists overflowing the layout.
