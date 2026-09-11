# Console timezone vs. UTC

## The trap

Your SentinelOne console may be configured to display times in a local
timezone (for example, UTC+1), while:

- your incident timeline was built from UTC timestamps (badge logs,
  SIEM correlation, another tool's export),
- or a colleague reported an event time in UTC.

If you take a UTC time window and type it directly into a console set
to UTC+1, you will silently search the wrong hour — the query runs
without error, just against the wrong window, and a real event outside
the (incorrectly shifted) window will be missed entirely.

## Fix

1. **Confirm the console's configured timezone first** — check the
   console/user settings, don't assume it matches UTC.
2. **Convert explicitly before building the time window.**

   Example: an incident window defined as 12:50–14:00 UTC, on a console
   set to UTC+1, should be entered as **13:50–15:00** in the console's
   time picker.

3. **Read every timestamp in the results as shifted by the same
   offset.** A result timestamped 13:33 in a UTC+1 console corresponds
   to 12:33 UTC — if you're cross-referencing against a UTC-based
   report (e.g. a colleague's account of "something happened around
   12:30"), do the conversion before comparing, not after.

## Practical habit

When starting an investigation that spans multiple tools, write the
window down in UTC first, then note the required per-tool offset next
to it, e.g.:

```
Incident window: 12:50–14:00 UTC
  SentinelOne console (UTC+1): enter as 13:50–15:00
  SIEM (UTC): enter as-is, 12:50–14:00
```

This avoids re-deriving the conversion under time pressure partway
through an investigation, and makes it easy to spot the mistake if a
result doesn't line up with expectations.
