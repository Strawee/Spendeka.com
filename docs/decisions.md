# Key design decisions

[← back to overview](../README.md)

A selection of the trade-offs behind the system, written in short decision-record form.

### 1. Work alongside SAP, never inside it
**Context:** Changing the ERP is slow, expensive and risky.
**Decision:** Read only standard SAP exports (stock, open POs, sales, delivery notes) and keep all planning state in a separate database.
**Consequence:** Adoption needed no IT project. The cost is an import step, which I made forgiving: saved column mappings, validation, and stage-then-swap loading.

### 2. No front-end framework
**Context:** A single developer who also runs procurement full-time, and long-lived screens heavy with data.
**Decision:** Plain ES modules and HTML strings, with a strict module contract and a token-based design system.
**Consequence:** No framework upgrades to chase and a small bundle. Discipline comes from conventions and tests instead of a library.

### 3. Migrate from a JSON document to Postgres, one table at a time
**Context:** The first version stored everything in one JSON document. That worked for one user and broke down with several.
**Decision:** Move each business domain to relational tables in its own phase, with a round-trip test proving no data is lost and a Python twin of each mapper for server-side sync.
**Consequence:** The app stayed in daily use throughout the 40+ phase migration, and every step could be rolled back.

### 4. Write a small LP solver instead of adding a dependency
**Context:** Supplier allocation needs a linear program, but the models are small (hundreds of rows) and the answer must be reproducible for audit.
**Decision:** A deterministic dense-tableau simplex method with lexicographic objectives. It is kept behind one function call so it can be swapped for HiGHS if the models grow.
**Consequence:** The same input always gives the same award, and the priorities (cover demand, then capacity, then cost) need no weights.

### 5. Robust regression for freight cost
**Context:** Transporter invoices contain outliers such as detours, waiting time and data-entry errors.
**Decision:** Fit cost per km with Theil–Sen, and show least squares next to it for comparison.
**Consequence:** One bad invoice no longer moves the rate used for every future quote.

### 6. Pushed is not live
**Context:** Work pushed to a branch can look finished without being reviewed, tested or live.
**Decision:** Production deploys only from `main`, only after CI passes on that exact commit, and the deploy confirms the live `/version.json`.
**Consequence:** What is live always matches a reviewed, tested commit.

### 7. Documentation is part of the change
**Context:** Documents went stale quickly while the code changed every day.
**Decision:** A register maps each document to the code paths it describes, and CI fails when those paths change without the document being updated or an explicit waiver.
**Consequence:** The developer handbook and user guide stay current enough to hand to someone else.
