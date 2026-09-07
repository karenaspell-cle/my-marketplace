# Workspace config — fill in with your own real values

This file holds the identifiers that are specific to one team's Notion/Slack workspace.
Nothing in this file should be treated as globally correct — every team installing this
plugin edits this file to point at their own database and channel.

**Do not commit real workspace IDs to a public repository.** If you're publishing this
plugin publicly, keep this file as placeholders in the published version and maintain your
real, filled-in copy privately (a private repo, or just locally).

- **Database:** <Your Sales → Product Meeting Requests database name>
- **URL:** <your Notion database URL>
- **Data source ID:** <collection://your-id-here>
- **Location:** <workspace location / team name>
- **Slack channel:** <#your-channel-name>, `<channel-id>` — confirm this is still current
  before posting; channels move.

Fields expected on the database (edit if your schema differs): Request (title), Requestor,
Customer / Account, Meeting Date, Date Submitted, The Ask, Who's in the Room (Technical /
Exec sponsor / Mixed), Prior Commitments, Status, Owner, Notice Flag (formula).
