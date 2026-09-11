# Was anyone actively using this endpoint during a specific window?

## Scenario

You need to confirm whether a machine was in genuine interactive use
during a defined window — for example, corroborating an employee's
reported absence, or investigating a badge-access discrepancy against
endpoint activity.

Fictional example used throughout: endpoint `WORKSTATION-7YXQ42AB`,
user `j.doe`, window 12:50–14:00 UTC.

## Step 1 — Don't start with a raw process-user filter

This looks like the obvious first query:

```
event.type = "Process Creation" and src.process.user = "j.doe"
```

**Problem:** on macOS in particular, per-user LaunchAgents and many
background services run *under the logged-in user's UID* even when no
human is present at the keyboard. This query will return a large volume
of legitimate background noise — Spotlight indexing, sync clients,
update checkers — that has nothing to do with human interaction. Using
this alone to conclude "the user was active" is a common false
positive.

## Step 2 — Anchor on the actual login/unlock event

The decisive signal for "did a human authenticate to this machine" is
the login/unlock event, not process creation:

```
event.category = "logins" and endpoint.name contains "7YXQ42AB"
```

Use the unique alphanumeric tail of the hostname (`7YXQ42AB`) rather
than the full display name — this avoids encoding issues if the full
endpoint name contains spaces or accented characters.

Note: login event *type* values differ between macOS and Windows
endpoints. Don't hard-filter on an assumed type string — run the base
filter first and read the actual values back before narrowing further.

## Step 3 — If you need to characterize activity, group and inspect manually

To distinguish interactive use from background noise across a window,
group process creation by process name and user, then judge each row:

```
event.type = "Process Creation" and endpoint.name contains "7YXQ42AB"
| group src.process.name, src.process.user
| sort -hits
```

**How to read the results:**

- Rows where `src.process.user = root` and the parent is
  `/sbin/launchd` (or the Windows equivalent, a service host process)
  → system/background activity, not human interaction.
- Rows showing the named user with interactive application names
  (browser, terminal, mail client, office apps) → likely genuine human
  presence.

This requires manual judgment — there isn't a single filter that
reliably separates the two categories on its own.

## Common mistakes to avoid

- **Field mix-up:** putting a username value into `endpoint.name`
  (which expects a hostname) instead of `src.process.user` — this
  silently returns zero results rather than an error, which is easy to
  misread as "no activity" when it's actually "wrong field."
- **Assuming Windows patterns apply to macOS** — e.g. filtering by a
  specific parent process like `explorer.exe` to detect interactive
  shells is a Windows-specific technique with no macOS equivalent.
- Treating a single suspicious-looking process at a specific timestamp
  as conclusive without checking whether its parent process indicates
  background/system activity first.
