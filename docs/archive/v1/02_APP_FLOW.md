# App Flow Document — Tengesa POS
All diagrams are Mermaid. **Lucidchart imports Mermaid directly** (Insert → Diagram as code → Mermaid), so paste any block below to get an editable chart. They also render on GitHub and in Claude Code.

---

## 1. Navigation map (information architecture)

Yoco pattern: bottom tabs on phone, left rail on tablet. Sell is home.

```mermaid
flowchart TD
    A[App Launch] --> B{Merchant set up?}
    B -- No --> C[Onboarding]
    B -- Yes --> D{Staff PIN lock}
    D --> E[SELL tab - home]
    E --- F[HISTORY tab]
    E --- G[STOCK tab]
    E --- H[REPORTS tab]
    E --- I[MORE tab]
    F --> F1[Sale detail] --> F2[Refund / Resend receipt / Edit note]
    G --> G1[Product list] --> G2[Product detail / edit]
    G --> G3[Stock adjustments]
    G --> G4[Low stock alerts]
    H --> H1[Sales summary] & H2[By product] & H3[By staff] & H4[By tender] & H5[End-of-day cash-up]
    I --> I1[Business settings] & I2[Staff management] & I3[Currency rate] & I4[Sync status] & I5[Catalogue import/export] & I6[Lock screen]
```

## 2. First-run onboarding

```mermaid
flowchart TD
    A[Install & open] --> B[Welcome / value screens x2]
    B --> C{Create account or Sign in}
    C -->|Offline?| C1[Offline trial mode - local only, account required to sync]
    C --> D[Business name, phone, logo optional]
    D --> E[Set currencies: USD base + ZWG rate]
    E --> F[Choose tender types to enable]
    F --> G{Import catalogue?}
    G -- CSV --> G1[Map columns -> import]
    G -- Skip --> H
    G1 --> H[Create owner PIN]
    H --> I[Add staff now or later]
    I --> J[SELL screen + 3-step coach marks]
```

## 3. Core sale flow (the money path)

```mermaid
flowchart TD
    A[SELL screen] --> B{Input mode}
    B -- Catalogue --> C[Tap product tiles]
    C --> C1{Has variants/modifiers?}
    C1 -- Yes --> C2[Variant/modifier sheet] --> D
    C1 -- No --> D[Item in cart]
    B -- Quick amount --> E[Keypad amount + optional label] --> D
    D --> F[Cart review: qty, line discount, note]
    F --> G{Action}
    G -- Save bill --> G1[Named open bill] --> A
    G -- Charge --> H[Tender screen: total in USD + ZWG]
    H --> I[Select tender type + amount received]
    I --> I1{Split across tenders? v1.1}
    I --> J[Change due - cross-currency calc]
    J --> K[Commit sale: atomic local write + stock deduction + queue for sync]
    K --> L{Receipt?}
    L -- Share --> L1[WhatsApp / SMS share sheet]
    L -- Skip --> M[Done -> new sale]
    L1 --> M
```

## 4. Refund flow

```mermaid
flowchart TD
    A[HISTORY] --> B[Search / filter sales]
    B --> C[Sale detail]
    C --> D[Refund button]
    D --> E{Role >= Supervisor?}
    E -- No --> E1[Supervisor PIN prompt] -->|fail| C
    E1 -->|pass| F
    E -- Yes --> F[Full or partial - select lines]
    F --> G[Reason - required dropdown]
    G --> H{Restore stock?}
    H --> I[Confirm - irreversible]
    I --> J[Negative sale record, linked to original, queued for sync]
    J --> K[Updated receipt shareable]
```

## 5. Stock adjustment flow

```mermaid
flowchart TD
    A[STOCK tab] --> B[Product]
    B --> C[Adjust stock]
    C --> D{Type}
    D --> D1[Received +qty]
    D --> D2[Damaged/expired -qty]
    D --> D3[Count correction -> set counted qty]
    D1 & D2 & D3 --> E[Reason + optional note]
    E --> F[Role check: Supervisor+ ]
    F --> G[Append adjustment record - never overwrite]
    G --> H[New on-hand = sum of movements]
```

## 6. Offline sync state machine

```mermaid
stateDiagram-v2
    [*] --> Offline
    Offline --> Syncing: connectivity gained / app foreground / manual
    Syncing --> PushQueue: send local ops (idempotency keys)
    PushQueue --> PullChanges: server ack
    PullChanges --> ResolveConflicts: apply remote deltas since cursor
    ResolveConflicts --> Idle: all clean
    ResolveConflicts --> NeedsAttention: unresolvable (rare) -> flag in More > Sync
    Idle --> Offline: connectivity lost
    Idle --> Syncing: new local ops
    NeedsAttention --> Syncing: user resolves
```

Conflict rules (detail in `04_BACKEND.md`): sales are append-only (no conflicts); product edits = last-write-wins on server receive time; stock = additive movement log (order-independent).

## 7. Staff switch & lock

```mermaid
flowchart LR
    A[Any screen] --> B[Lock icon / auto-lock after N min]
    B --> C[PIN pad + staff avatars]
    C -->|correct PIN| D[Session as that staff member]
    C -->|5 fails| E[60s lockout]
    D --> F[All actions stamped with staff_id]
```

## 8. End-of-day cash-up

```mermaid
flowchart TD
    A[REPORTS > End of day] --> B[Select date - default today]
    B --> C[Per currency: opening float + cash sales - cash refunds - payouts = expected]
    C --> D[Enter counted cash USD / ZWG]
    D --> E{Variance?}
    E -- Yes --> F[Variance recorded + note]
    E -- No --> G[Balanced]
    F & G --> H[Z-report: totals by tender, staff, category]
    H --> I[Share as image/PDF to owner WhatsApp]
```

## 9. Screen inventory (build checklist)

Onboarding (4) · PIN lock · Sell · Variant sheet · Cart · Tender · Receipt preview · Open bills · History list · Sale detail · Refund · Stock list · Product edit · Adjustment · Low-stock · Reports home · Report detail ×4 · End-of-day · Settings home · Business settings · Staff list/edit · Currency rate · Sync status · Import/export · About/legal. **≈ 27 screens.**
