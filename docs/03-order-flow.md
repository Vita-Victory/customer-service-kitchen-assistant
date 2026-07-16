# Order Lifecycle and Spreadsheet Rotation

## 1. Lifecycle of a dish row in Kitchen Assistant

Each item (dish/drink) moves through a simple state machine in the **Total status** column:

```mermaid
stateDiagram-v2
    [*] --> new: Dish first appears\nin orders with status new
    new --> number: Kitchen starts cooking,\nmanually enters a number
    new --> done: Kitchen finishes the batch

    number --> number: New cycle, no new orders —\nB is left untouched
    number --> number2: New cycle, new orders exist —\nB = number + new sum
    number2--> number

    done --> done: New cycle, no new orders —\nrow stays done and sinks to the bottom
    done --> new: New cycle, new orders exist —\nB = new sum, C resets to new

    note right of number
        The number in column C is the
        kitchen's own working note
        (e.g. "X portions still to cook").
        The agent never overwrites
        this number itself.
    end note
```

## 2. Kitchen screen sorting rule

The kitchen screen must always show **what genuinely needs cooking right now**, not what's already done.

```mermaid
flowchart TD
    Start["Item list after\nAgent 2's update"] --> Split{"Total status?"}
    Split -->|"new or a number"| Top["Active group\nsorted by B descending"]
    Split -->|"done"| Bottom["done group\nat the bottom of the list"]
    Top --> Render["Kitchen screen"]
    Bottom --> Render
```

The rule is simple: a row only sinks to the bottom if it's `done` **and** no new orders for that dish showed up in the last 30-minute cycle. As soon as a new batch arrives, the status resets to `new` and the dish moves back to the top.

## 3. Daily tab rotation

Every calendar day, Agent 1 creates a new tab in the orders spreadsheet, and Agent 2 creates a matching, identically named tab in Kitchen Assistant. The naming format is `<day> <month>`, e.g. `16 July`.

```mermaid
flowchart LR
    subgraph Month["orders_July / kitchen_July"]
        D1["Tab \"15 July\""]
        D2["Tab \"16 July\" (current)"]
        D3["Tab \"17 July\""]
    end
    D1 -.archived, no longer updated.-> D1
    D2 -->|"00:00 → new day"| D3
```

Past days' tabs are never deleted or modified — they're a historical archive of orders and of what was actually cooked (based on the final values in column C).

## 4. Monthly spreadsheet rotation

At the start of a new calendar month, Agent 1 creates a **brand-new pair of files** — the source and destination spreadsheets — instead of adding yet another tab to an ever-growing file.

```mermaid
flowchart LR
    July["orders_July.gsheet\nkitchen_July.gsheet"] -->|"August 1"| August["orders_August.gsheet\nkitchen_August.gsheet"]
    August -->|"September 1"| September["orders_September.gsheet\nkitchen_September.gsheet"]
```

Reasons for this specific design:

- **Performance.** Google Sheets slows down noticeably with many tabs/rows in one file — monthly rotation keeps files lean.
- **Access rights and backups.** Last month's file can simply be frozen (set read-only, exported to archive) without touching current work.
- **Easy navigation.** It's easier for the owner to find "June's orders" than to scroll through one giant file spanning the whole year.

## 5. What happens to an order from intake to cooking

```mermaid
flowchart LR
    O["Order received\n(status: new)"] --> P["Agent 2 sums it up\nevery 30 min"]
    P --> Q["Appears/updates\nin Kitchen Assistant"]
    Q --> R["Agent 1 flips the source row:\nnew → redirected"]
    R --> S["Kitchen cooks,\nsets done/a number"]
    S --> T{"New matching\ndishes in 30 min?"}
    T -->|Yes| Q
    T -->|No| U["Stays done,\nsinks to the bottom"]
