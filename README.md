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

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.AuditLog;
using AuditLogKitTests;

[Test(Garden)]
void a_steward_reads_each_change_with_its_values_who_and_when() {
  runas (Ada) {
    var plot = AddPlot("North bed", 10);
    Resize(plot, 14);
    var lines = AuditLogLines(Yesterday());
    var changed = lines.Single(l => l.Kind == "Plot" && l.Happened == "Changed");
    Assert.Equal("North bed", changed.Name);
    Assert.Contains("Square metres: 10 → 14", changed.Fields);
    Assert.Equal("Ada Steward", changed.By);
    var added = lines.Single(l => l.Kind == "Plot" && l.Happened == "Added");
    Assert.Equal("North bed", added.Name);
    // Newest first.
    Assert.True(lines.IndexOf(changed) < lines.IndexOf(added));
  }
}
```

From [`tests/audit-log.test.osy`](tests/audit-log.test.osy).

```osy
using Osysharp.Identity;
using AuditLogKitTests;

[Test(Garden)]
void signing_in_takes_the_right_code_and_refuses_a_wrong_one() {
  Assert.NotEqual("", SignInWithCode("ada@garden.test", "1111"));
  Assert.Equal("", SignInWithCode("ada@garden.test", "9999"));
}
```

From [`tests/garden.test.osy`](tests/garden.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.AuditLog;
using AuditLogKitTests;

[Test(Garden)]
void somebody_the_trail_is_closed_to_is_neither_shown_the_log_nor_given_a_line() {
  runas (Ada) { Resize(AddPlot("South bed", 8), 9); }
  runas (Gus) {
    Assert.False(new AuditLogSettingsSections().Sections()[0].Shown);
    // The page is anybody's to open; the rows are the trail's to give, and it gives a gardener none.
    Assert.Equal(0, AuditLogLines(Yesterday()).Count);
  }
  runas (Ada) { Assert.True(new AuditLogSettingsSections().Sections()[0].Shown); }
}
```

From [`tests/audit-log.test.osy`](tests/audit-log.test.osy).

## Learn more

- [The Osysharp.AuditLog reference](https://osysharp.com/reference/ui/audit-log-kit/)
- [Osysharp.AuditLog in the package catalogue](https://osysharp.com/packages/#osysharp-auditlog)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
