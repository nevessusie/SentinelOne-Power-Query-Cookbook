# Hunting for a known-bad IP or domain across the fleet

## Scenario

You've received an indicator of compromise (IP, domain, or file hash) —
from a vendor advisory, threat intel feed, or an incident report — and
need to check whether any endpoint in your environment has connected to
it.

## Network connection hunt

```
event.type = "DNS Resolved" and event.dns.request contains "example-bad-domain.com"
```

```
event.type = "IP Connect" and event.network.destination.ip = "203.0.113.50"
```

**Note on DNS event types:** SentinelOne exposes more than one
DNS-related event type (e.g. resolved vs. unresolved queries). A narrow
event type filter can silently under-return results — if a hunt for a
known-bad domain comes back empty, verify you're using the broadest
relevant DNS event type before concluding the domain wasn't contacted,
rather than assuming a clean result.

## File hash hunt

```
event.type = "File Creation" and src.process.image.sha256 = "<hash>"
```

or, to check whether a known-bad binary ever executed:

```
event.type = "Process Creation" and src.process.image.sha256 = "<hash>"
```

## Turning a one-off hunt into standing detection

If the indicator is from an active/ongoing threat rather than a
one-time check, convert the hunt into a saved watchlist or STAR rule so
future matches alert automatically instead of relying on someone
re-running the same query later.

## Common mistakes to avoid

- Using a domain string with a stray space or the wrong separator
  (hyphen vs. underscore) — a typo returns zero results with no error,
  which is easy to misread as "no findings" rather than "bad query."
- Assuming vendor-published IOCs for a *named threat group* apply to an
  unrelated incident just because the names sound similar (e.g. two
  unrelated actors or malware families sharing a common word in their
  name) — confirm attribution before applying someone else's IOC list
  to your own investigation.
- Running the hunt only against live/recent data when the suspected
  compromise window is older than your default lookback — always set
  the time range to cover the full window under investigation, not the
  console's default.
