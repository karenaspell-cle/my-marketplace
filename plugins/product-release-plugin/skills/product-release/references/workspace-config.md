# Workspace config — fill in with your own real values

Two Notion databases and one customer list, specific to one team's workspace. Placeholders
below — edit before use. **Do not commit real workspace IDs or customer names to a public
repository.** Keep your real, filled-in copy private.

- **Source database ("Cross-Team Work and Learning"):**
  - Data source ID: `<PRODUCT_RELEASE_DB_ID>` style token, e.g. `collection://...`
  - Properties this skill reads: Learning (title), Segment / Product Layer, Workstream, Status,
    Release Size, Launch date / Completed At, Channel, Customer, Parent item, Sub-item,
    Linear (ticket link)

- **Output database ("Product Release templates"):**
  - Data source ID: `collection://...`
  - URL: `<your Notion database URL>`
  - Properties: Title, Category/Area, Release Size, Go-live Date, Source Card (two-way
    relation back to the source database), Channel, Status (Draft/Ready/Published)

- **Customer list** — the values your team's `Customer` property actually uses (edit to match
  your own picklist, e.g.: Customer A, Customer B, Customer C, Internal).

Re-derive the actual IDs from your own Notion workspace (open each database, check the URL/
share settings) rather than guessing — a wrong ID fails loudly rather than silently pointing at
someone else's data, but there's no reason to hunt for the error when you can just look it up.
