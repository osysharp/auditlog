# Osysharp.AuditLog

The app's audit log, as a page of its settings. Every change the platform's change trail holds that the reader may
read: what kind of record, which one, what happened to it, each field from what to what, who made it and when. Newest
first, narrowed by period, by kind of record and by person.

**Use case.** A community garden lets its stewards read who changed what. The garden already says so in `app.Audit`.
With one more line, the stewards find "Audit log" among the garden's settings: "Plot · North bed — Square metres: 10 →
14, by Ada Steward". A gardener is not offered the page, and opening it by its address shows nothing of the trail.

```osy
app Garden {
  use Osysharp.AuditLog@0;                         // the whole of mounting it
}

app.Audit = new AuditConfig {                      // who reads the trail: said once, where it always is
  EntityAuditRecord = new AuditSurface { Read = user => IsSteward },
};
```

## What it does

| | |
|---|---|
| **One line** | `use Osysharp.AuditLog@0;` puts the page at `/settings/audit` and its link among the app's settings (`ISettingsProvider`) |
| **The trail's own rule** | the rows are the ones the app's `app.Audit` read rule gives the reader, and no others. An app that opens the trail to admins for its settings alone gets a log of its settings alone |
| **Offered honestly** | the settings link is shown to whoever that rule lets read at least one change, asked as the reader |
| **The values** | each field from what to what; a reference says only that it moved; a value the app redacts reads `‹redacted›` |
| **Named records** | a record is named from its `Name`, `Title`, `DisplayName`, `Label`, `Code` or `Email`, even on a change that did not touch it, and after it is removed |
| **Narrowed** | by period (7, 30, 90 days, a year), by kind of record and by person; the newest 200 changes at a time |

## What the app may say

- `policy ReadsAuditLog => IsAdmin;` — who may open the page at all (anybody signed in by default). The rows are still
  the trail's to give.
- A component `AuditLogSettings` of its own takes the page over by name. `AuditLogLines(since)` and `AuditLogList(…)`
  are there to build it from.

## Its tests

`tests/` is a community garden: a steward reads each change with its values; a gardener is neither offered the log nor
given a line; a redacted value never appears; a removed plot is still named; and the page from the settings area, on a
phone in the dark too.
