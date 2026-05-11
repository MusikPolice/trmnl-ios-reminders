# trmnl-ios-reminders

A private [TRMNL](https://usetrmnl.com) plugin that displays your most urgent iOS Reminders
on the e-ink display. Powered by the TRMNL iOS Companion app, which POSTs reminder data to a
webhook each morning.

## What it shows

A single sorted list of reminders that are **overdue or due today**, oldest first. Future items
are hidden. Each row shows the task title, how overdue it is (or "Today"), and which list it
came from. High-priority items are prefixed with `!`.

## Local development

Requires [Docker Desktop](https://www.docker.com/products/docker-desktop/).

```powershell
# from the repo root
.\bin\serve.ps1
```

Then open `http://localhost:4567` in a browser. The server watches `src/` and `.trmnlp.yml`
for changes and reloads automatically.

### Fixture data

`.trmnlp.yml` contains generic sample reminders used during local preview. The real reminder
data (`resources/reminders.json`) is gitignored — it's written by the iOS Companion app and
contains personal information.

To adjust which items appear in the local preview, edit the `reminders` array in `.trmnlp.yml`.
Date strings are UTC ISO 8601; for Eastern Time, midnight = `T04:00:00.000Z`.

## Deployment

This is a private TRMNL plugin using the **webhook** strategy. The iOS Companion app sends
reminder data directly to TRMNL's webhook endpoint. No hosted server is required.

To deploy markup changes:

```powershell
# authenticate once (stores credentials in ~/.config/trmnlp)
docker run --rm -it `
  --volume "${PWD}:/plugin" `
  --volume "$env:USERPROFILE/.config/trmnlp:/root/.config/trmnlp" `
  trmnl/trmnlp login

# push markup to your TRMNL plugin
docker run --rm -it `
  --volume "${PWD}:/plugin" `
  --volume "$env:USERPROFILE/.config/trmnlp:/root/.config/trmnlp" `
  trmnl/trmnlp push
```

Or copy the rendered HTML from `localhost:4567` into the plugin's WYSIWYG editor at
`trmnl.com/plugins/my`.

## Layouts

| Layout | Status | Description |
|---|---|---|
| Full (800×480) | complete | Up to 8 items, two-line rows with age label and list name |
| Half Horizontal (800×240) | complete | 8 items in two columns of 4 via CSS grid |
| Half Vertical (400×480) | complete | 8 items, age label only (list name omitted to save width) |
| Quadrant (400×240) | complete | 6 items, title only, compact single-line rows |

## Data model

The iOS Companion app provides two Liquid variables:

**`{{ reminders }}`** — array of reminder objects

| Field | Type | Notes |
|---|---|---|
| `title` | string | Always present |
| `notes` | string | Always present, often empty |
| `priority` | int | `0` = none, `1` = high |
| `list_name` | string | e.g. "Family", "Home Maintenance" |
| `due_date` | string | Optional. ISO 8601 UTC timestamp |
| `reminder_identifier` | string | Stable UUID |

**`{{ trmnl }}`** — device and user context (see `resources/trmnl.json` for shape)
