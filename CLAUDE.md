# CLAUDE.md

## Project

TRMNL private plugin that renders iOS Reminders on an 800×480 2-bit grayscale e-ink display.
The iOS Companion app POSTs reminder data to a TRMNL webhook each morning. This repo contains
the Liquid markup that TRMNL renders into a screen image.

## Tech stack

- **Liquid** (Shopify-flavored, Ruby) — templating language for all markup
- **TRMNL CSS framework** — loaded automatically by the device; use its classes where possible
- **trmnlp** — local dev server, run via Docker (`.\bin\serve.ps1`)
- No build step, no package manager, no server — just `.liquid` files

## Key files

| File | Purpose |
|---|---|
| `src/full.liquid` | Full-screen layout (800×480) — primary view, most complete |
| `src/half_horizontal.liquid` | 800×240 layout |
| `src/half_vertical.liquid` | 400×480 layout |
| `src/quadrant.liquid` | 400×240 layout |
| `src/shared.liquid` | CSS/markup included before all layout files |
| `src/settings.yml` | Plugin metadata (strategy: webhook) |
| `.trmnlp.yml` | Dev server config and fixture data |
| `bin/serve.ps1` | Docker runner for trmnlp |
| `resources/trmnl.json` | Sample `{{ trmnl }}` payload (gitignored, personal) |
| `resources/reminders.json` | Sample `{{ reminders }}` payload (gitignored, personal) |

## Data model

`{{ reminders }}` is an array. Every item has `title`, `notes`, `priority` (0 or 1),
`list_name`, and `reminder_identifier`. `due_date` is optional (ISO 8601 UTC string).

`{{ trmnl }}` key paths used in templates:
- `trmnl.system.timestamp_utc` — Unix timestamp of last refresh (integer)
- `trmnl.user.utc_offset` — seconds offset from UTC (e.g. -14400 for EDT)
- `trmnl.user.first_name`
- `trmnl.device.width` / `trmnl.device.height`

## Timezone handling

Always compute today's local date by shifting the UTC timestamp before formatting:

```liquid
{% assign local_ts = trmnl.system.timestamp_utc | plus: trmnl.user.utc_offset %}
{% assign today    = local_ts | date: "%Y-%m-%d" %}
```

Never use `trmnl.system.timestamp_utc | date: "%Y-%m-%d"` directly — Liquid's `date` filter
uses the server's timezone (UTC on TRMNL's backend), which causes off-by-one day errors for
users in negative UTC offsets in the evening hours.

## Display constraints

- 800×480 px, 4 shades of gray only (black, dark gray, light gray, white)
- No color — use weight, size, and shade to convey hierarchy
- TRMNL framework uses `flex--row` layout by default; use `flex flex--col` for vertical lists
- Font: Inter via Google Fonts (`<link>` at top of layout file), system sans-serif fallback

## Layout structure

```html
<div class="screen">
  <div class="view view--full">        <!-- or view--half_horizontal etc -->
    <div class="title_bar">
      <p class="title">...</p>
    </div>
    <div class="flex flex--col">
      <div class="list-wrap">          <!-- centered fixed-width container -->
        <!-- rows here -->
      </div>
    </div>
  </div>
</div>
```

## Row cap per layout

Rows are two-line (title + meta). Approximate maximums:
- Full (800×480): 8 rows
- Half Horizontal (800×240): to be determined
- Half Vertical (400×480): to be determined
- Quadrant (400×240): to be determined
