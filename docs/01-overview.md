# Project Overview

## Who the client is

A small café with a single location. Pre-orders come in three ways:

| Source | How the order arrives | Who enters the data |
|---|---|---|
| Walk-in customer | Says the order on the spot | Café staff (tablet/register) |
| Café website | Pre-order widget | The customer, via the website |
| Phone | Phone call | Café staff, entered manually |

## The problem

Before automation, all three channels funneled to one person who hand-wrote the kitchen's list. As volume grew, this led to:

- duplicated dishes on the list (mental math, rounding mistakes);
- stale information — the kitchen had no idea how much of something was actually ordered until someone asked;
- no prioritization — it wasn't clear what to cook first.

## What was needed

1. A single collection point for orders from all three sources.
2. Automatic recalculation from "by customer" to "by dish" every 30 minutes.
3. A working list for the kitchen that hides what's already cooked and surfaces what genuinely needs cooking now.
4. Archiving by day and month with no manual cleanup — so sheets don't grow indefinitely and past days remain a historical record without getting in the way of current work.

## What we deliberately did not do (and why)

- **We didn't build a separate POS system.** For a single café with three channels, that's overkill in cost and timeline — see [budget and hosting](05-budget-and-hosting.md).
- **We didn't build a complex dashboard with logins/passwords.** Kitchen staff already work comfortably with a spreadsheet — the interface should stay familiar rather than introduce a new tool that needs training.
- **We didn't automate marking a dish "done" in the kitchen.** This is the one deliberately manual step — only a person on the line actually knows what's really finished.

## The core architectural idea

All the data lives in ordinary Google Sheets — already familiar to the owner and staff. Two agents (described in [docs/04-agent-prompts.md](04-agent-prompts.md)) act as an "invisible employee" who recalculates and re-sorts the orders every 30 minutes, instead of someone doing it by hand.
