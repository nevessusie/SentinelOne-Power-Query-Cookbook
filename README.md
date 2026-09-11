# SentinelOne PowerQuery Cookbook

A collection of practical SentinelOne Deep Visibility / PowerQuery
patterns for common SOC investigation scenarios, written from real
hands-on triage work — not just documentation summaries.

This is the kind of query cookbook I wish existed when I started
working with SentinelOne: it focuses on the *gotchas* that don't show
up in the official docs — platform-specific noise, query-mode syntax
traps, and how to avoid drawing the wrong conclusion from a technically
correct query.

All scenarios below are generic/synthetic (fictional endpoint names,
usernames, and timestamps) — no data, IOCs, or details from any real
investigation.

## Why PowerQuery gotchas matter

The two biggest ways analysts get burned in SentinelOne aren't query
syntax — they're **query mode confusion** and **mistaking background
noise for human activity**. Both are covered below.

## Query mode: Basic Event Search vs. PowerQuery

SentinelOne has two distinct query surfaces that look similar but
accept different syntax:

| Mode | Accepts pipes (`\|`) | Typical use |
|---|---|---|
| Basic Event Search | ❌ No | Simple single-condition filters |
| PowerQuery | ✅ Yes (`\| group`, `\| columns`, `\| sort`) | Aggregation, grouping, sorted/ranked output |

**The trap:** pasting a PowerQuery pipeline into the Basic Event Search
bar returns a `Don't understand [|]` error that *looks* like a syntax
problem in your filter logic. It usually isn't.

**Fast diagnostic:** strip everything from the first `|` onward and run
the bare filter in Basic Event Search first. If that works, the issue
was query mode, not query logic — switch to PowerQuery mode before
adding `| group` / `| sort` / `| columns`.

See [`queries/mode-diagnostic.md`](queries/mode-diagnostic.md).

## Scenario 1 — "Was anyone active on this endpoint during a specific window?"

A common ask: confirm whether a machine was actually in interactive use
during a defined time window (e.g. an employee's reported absence, or a
badge-access discrepancy).

**The trap:** on macOS, per-user LaunchAgents and background daemons run
*under the logged-in user's UID* even when no human is physically
present. A naive filter like `src.process.user = "<username>"` returns
a large amount of legitimate background noise that has nothing to do
with human interaction — and can make a dormant machine look "active."

**The fix:** process-creation events alone can't answer "was a human
here" — you need the actual login/unlock event, or you need to group
process activity by parent process and manually separate daemon-spawned
noise (parent = `/sbin/launchd`, user = `root`) from genuinely
interactive applications (browsers, terminal apps, productivity tools,
parented by the user's own session).

See [`queries/endpoint-activity-window.md`](queries/endpoint-activity-window.md).

## Scenario 2 — Console timezone vs. UTC

**The trap:** your SentinelOne console may be configured to a local
timezone (e.g. UTC+1), while incident timelines, log correlation, and
other tools (SIEM, badge system, ticketing) are usually recorded in
UTC. If you build a query window using UTC timestamps directly in a
console set to UTC+1, you will silently search the wrong hour.

**The fix:** always confirm the console's configured timezone before
building a time window, and explicitly convert — don't assume the
console matches the timestamps in your source material.

See [`queries/timezone-conversion.md`](queries/timezone-conversion.md).

## Scenario 3 — Hunting for a known bad IP or domain

A generic outbound-connection hunt pattern, useful any time you have an
indicator (IP, domain, or hash) and need to check whether any endpoint
in the fleet has talked to it.

See [`queries/ioc-hunt-network.md`](queries/ioc-hunt-network.md).

## Repository structure

```
sentinelone-powerquery-cookbook/
├── README.md
└── queries/
    ├── mode-diagnostic.md
    ├── endpoint-activity-window.md
    ├── timezone-conversion.md
    └── ioc-hunt-network.md
```

## Disclaimer

All endpoint names, usernames, hostnames, and timestamps in this
repository are fictional and constructed for illustration only. Query
syntax reflects SentinelOne's publicly documented Deep Visibility /
PowerQuery language as of the time of writing; field names and
operators may change between platform versions — always validate
against your own tenant before relying on these in a live
investigation.
