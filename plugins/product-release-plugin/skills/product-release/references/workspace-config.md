# Workspace config — Calcs.com workspace

Two Notion databases and one customer list, specific to the Calcs.com team's workspace.
**Do not commit this filled-in file to a public repository** — it contains real workspace IDs.
Keep it private. (This repo is private, so it's fine here.)

- **Source database ("Cross-Team Work and Learning"):**
  - Data source ID: `collection://b7e21ab4-3bc5-4707-96b5-7243f56e5791`
  - URL: https://app.notion.com/p/437a475b8b3e4b89bad56d6bf06802b7
  - Properties this skill reads: Learning (title), Segment / Product Layer, Workstream, Status,
    Release Size, Launch date / Completed At, Channel, Customer, Parent item, Sub-item,
    Linear (ticket link). Note: `Days in stage` and `Cycle days` are read-only formula
    properties and cannot be queried via SQL or written to.

- **Output database ("Product Release templates"):**
  - Data source ID: `collection://17138423-4c60-44fd-8ad9-4ab9b2b813ab`
  - URL: https://app.notion.com/p/abe8f849965046d2b2927ffabb0e2ed8
  - Properties: Title, Category/Area, Release Size, Go-live Date, Source Card (two-way
    relation back to the source database), Channel, Status (Draft/Ready/Published)

- **Customer list** — the values the `Customer` property on the source database uses:
  Atlas Tube, ICC, Nucor, MAFI, ITI, Internal, Spacial, API.

Both IDs were re-derived directly from the Calcs.com Notion workspace on 2026-09-07 (via
workspace search and a schema fetch of each data source) rather than guessed. If either
database is ever recreated or its data source ID changes, re-resolve rather than assuming
these still work — a wrong ID fails loudly rather than silently pointing at someone else's data.
