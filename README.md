# SafePay ransomware (CISA advisory AA26-293A): indicators and hunt guide

> **Read first.** This package is aggregated from a public government advisory by SOC-Lite by BansalAI. It is not tested
> by us, it comes with no warranty, and it is not professional advice. It is as of **2026-10-27** and will go stale. See
> [DISCLAIMER.md](DISCLAIMER.md). We are not affiliated with any agency credited in [CREDITS.md](CREDITS.md), and none
> of them reviewed or endorsed this package.

On 2026-10-20 the FBI, CISA, the U.S. Secret Service and partner agencies in Germany and the United Kingdom published
a joint advisory on **SafePay**, a ransomware group that has been active since August 2024 and, by the agencies' count,
has hit at least 600 victims. The advisory describes how the group gets in, how it steals data and attacks backups, and
lists the indicators the agencies found.

This repository copies **those indicators** (with credit) and adds a short guide to checking your own systems for them.

**Not technical? Start with the plain-language version:**
[SafePay Ransomware: What to Ask Your IT Person About Backups and Remote Access](https://blog.bansalai.com/news/safepay-ransomware-backups-remote-access/?ref=github)
on the SOC-Lite blog. It gives four questions to put to your IT person or provider.

## What is in here

| File | What |
|---|---|
| [`iocs/iocs.csv`](iocs/iocs.csv) | One row per indicator, with the source, the date, how fragile it is, and a review date |
| [`docs/hunt-guide.md`](docs/hunt-guide.md) | What to check on Windows computers, servers and backup storage, what the results do and do not mean, and what to do if something looks real |
| [`CREDITS.md`](CREDITS.md) | The agencies whose advisory this is built from, and what we took from it |
| [`DISCLAIMER.md`](DISCLAIMER.md) | The full disclaimer |

## Read the results carefully

- **A match is a reason to look, not a verdict.** The advisory says several listed files are legitimate tools, and
  that it recommends investigating or verifying an address or tool before acting, such as blocking.
- **Do not mass-block addresses.** Some of the nine addresses look like shared infrastructure. Our own check of the
  address ranges suggests a few belong to services many ordinary people use (the rows say which). The agencies also
  warn that an address used for harm at one time can later be reassigned to harmless services. Look each one up
  against your own logs first.
- **We left out everyday tools on purpose.** The advisory lists tools such as WinRAR, AnyDesk, PsExec and a network
  scanner because the group used them. They are legitimate and common. Listing them as indicators would bury a real
  result in false alarms, so the hunt guide describes them in words instead.
- **The advisory is one source.** Every row comes from it, so no row is marked verified. We mark a row verified only
  when two independent sources report it.
- **These indicators go stale quickly.** The agencies observed them between November 2024 and August 2026. File names
  and hashes are cheap for an attacker to change, so a clean result is not an all-clear.
- **There is more in the advisory than we copy.** It names exploited flaws in internet-facing equipment, a mapping of
  the group's methods, and mitigations. Our copy of the page did not include the list of flaws, so read the advisory
  itself for it (link in [CREDITS.md](CREDITS.md)).

## Using `iocs.csv`

| Column | Meaning |
|---|---|
| `id` | Stable row ID (`SP-nnnn`) |
| `wave` | Always `safepay` in this release |
| `type` | `ip`, `hostname`, `filename`, `sha1`, `sha256` or `behaviour` |
| `value` | The indicator. Addresses are written as normal text, not defanged. Never open them in a browser |
| `match` | `exact` or `wildcard` |
| `evidence` | Where it would show: `network`, `file`, `process` or `event-log` |
| `fragile` | `yes` if the attacker can change it trivially |
| `verified` | `yes` only when two independent primary sources report it. `no` means a single source |
| `source_url`, `source_date` | Where we took it from and the date of that source |
| `fetched`, `review_by` | When we read it, and when it should be re-checked. Addresses and hostnames have short review dates |
| `notes` | Context and caveats. Sentences starting "Our note" are ours, not the agencies' |

Search your own systems with the steps in the hunt guide. This package does not ship anything that runs on your
systems or collects data from them.

## Licence

The text we wrote is licensed under CC BY 4.0 ([LICENSE](LICENSE)). The indicator values and facts belong to the
agencies that reported them: they are credited in [CREDITS.md](CREDITS.md), and we do not relicense them. Details and
how to credit us are in [NOTICE.md](NOTICE.md).

## Where to go for the full, current picture

The advisory is the source for everything here and may be updated. For its complete tables, its list of exploited
flaws and its mitigations, follow the link in [CREDITS.md](CREDITS.md).

## Help with a suspected compromise

If something looks real, preserve the logs first, isolate the affected computers, and call your IT provider. The agencies
also ask victims to report to the FBI (see the advisory). [SOC-Lite by BansalAI](https://cyber.bansalai.com/?ref=github)
helps small and mid-sized Central Florida businesses review alerts and work with their IT person or MSP.
[Book a free 20-minute visibility check](https://cyber.bansalai.com/?ref=github#book). More plain-language security
write-ups are on the [SOC-Lite blog](https://blog.bansalai.com/?ref=github).
