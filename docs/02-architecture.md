# System Architecture

## Components

| Component | Role |
|---|---|
| 3 order sources | Walk-in customer, café website, phone call |
| **Agent 1** | Order intake, status tracking, sheet/spreadsheet rotation by day and month |
| Orders sheet (`orders_<month>`) | The "source" spreadsheet, one tab per day |
| **Agent 2** | Every 30 minutes, re-tallies orders by dish and updates the kitchen's list |
| `kitchen assistant_<month>` spreadsheet | The kitchen's working list, one tab per day |
| Kitchen | Cooks from the list, manually sets the actual status |

## Overall diagram

```mermaid
flowchart TB
    subgraph SRC["Order sources"]
        direction LR
        A1["🧍 Walk-in customer"]
        A2["🌐 Café website"]
        A3["📞 Phone"]
    end

    subgraph AG1["Agent 1 — intake & routing"]
        direction TB
        B1["Log the order<br>status = new"]
        B2["Create the day's tab<br>'16 July' etc."]
        B3["Create a new spreadsheet<br>at the start of the month"]
        B4["After Agent 2 confirms<br>new → redirected"]
    end

    OS[("📄 Orders spreadsheet<br>orders_July<br>tab = current day")]

    subgraph AG2["Agent 2 — kitchen summary (every 30 min)"]
        direction TB
        C1["Read rows<br>status = new"]
        C2["Sum Quantity<br>by dish"]
        C3{"Check column C<br>Total status"}
        C4["C = done →<br>B = new sum<br>C = new"]
        C5["C = number →<br>B = number + new sum"]
        C6["New dish →<br>add row, C = new"]
        C7["Sort —<br>new/number on top, done at bottom"]
    end

    KS[("📋 Kitchen assistant<br>tab = current day<br>A dish · B qty · C status")]

    K["👨‍🍳 Kitchen"]

    A1 --> B1
    A2 --> B1
    A3 --> B1
    B1 --> OS
    B2 -.-> OS
    B3 -.-> OS

    OS --> C1 --> C2 --> C3
    C3 -->|done| C4
    C3 -->|number| C5
    C3 -->|no row yet| C6
    C4 --> C7
    C5 --> C7
    C6 --> C7
    C7 --> KS

    KS --> K
    K -->|"sets done / a number"| KS
    AG2 -.processing confirmed.-> B4
    B4 --> OS
```

## Syncing over time (the 30-minute cycle)

```mermaid
sequenceDiagram
    participant W as Walk-in / Website / Phone
    participant A1 as Agent 1
    participant OS as Orders spreadsheet
    participant A2 as Agent 2
    participant KS as Kitchen assistant
    participant K as Kitchen

    W->>A1: New order (at any moment)
    A1->>OS: Row, status = new

    loop Every 30 minutes
        A2->>OS: Reads rows with status = new
        A2->>A2: Sums Quantity by dish
        A2->>KS: Updates B and C per the rules
        A2->>A1: Confirmation: orders processed
        A1->>OS: status new → redirected
        K->>KS: Checks the list, cooks
        K->>KS: Sets Total status (done / a number)
    end
```

## Why it's built this way

- **Write access is split.** Agent 2 only reads the orders spreadsheet and writes to kitchen assistant; Agent 1 is the only one that changes status in the orders spreadsheet. This rules out a race condition where two processes edit the same status field at once.
- **`redirected` is only set after confirmation.** If Agent 2 fails mid-cycle, the unprocessed orders stay `new` and get picked up next time — no data is lost.
- **Column C ("Total status") is written only by the kitchen**, except in two moments: creating a new row and reactivating a row out of `done` — there, Agent 2 sets it to `new` to signal "there's something new to cook for this dish again."
