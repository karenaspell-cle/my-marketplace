# karen-personal-tools

A personal Claude Code plugin marketplace, for learning the plugin/marketplace mechanics.

## Structure

```
my-marketplace/
├── .claude-plugin/
│   └── marketplace.json          # catalog: lists plugins + where to fetch them
└── plugins/
    └── meeting-recap-plugin/
        ├── .claude-plugin/
        │   └── plugin.json       # plugin manifest
        └── skills/
            └── meeting-recap/
                └── SKILL.md       # the actual skill
```

## Test it locally, before pushing anywhere

From the parent directory of `my-marketplace` (i.e. one level up from this folder), open Claude Code and run:

```
/plugin validate ./my-marketplace
/plugin marketplace add ./my-marketplace
/plugin install meeting-recap-plugin@karen-personal-tools
```

Then try it: paste some rough meeting notes and ask for a recap. The skill should trigger and you'll see it invoked as `meeting-recap-plugin:meeting-recap`.

If the install summary says "Run /reload-plugins to activate," run that.

## Push it to your own GitHub

```bash
cd my-marketplace
git init
git add .
git commit -m "Initial marketplace: meeting-recap-plugin"
git remote add origin https://github.com/<your-username>/my-marketplace.git
git push -u origin main
```

Once it's on GitHub, anyone (including a future you on a different machine) can add it with:

```
/plugin marketplace add <your-username>/my-marketplace
/plugin install meeting-recap-plugin@karen-personal-tools
```

## Things worth deliberately practicing here

- **Bump `version` in plugin.json before every real change.** If you edit the skill but don't bump the version, `/plugin update` thinks nothing changed and users (including future-you) keep the stale cached copy.
- **Add a second skill to the same plugin**, then a second plugin to the same marketplace — this is the shape Calcs' `eng-skills`, `dev-skills`, etc. namespaces already take (one marketplace, many plugins, each plugin owning a handful of skills).
- **Try a `git-subdir` source** once you're comfortable — point a marketplace entry at a plugin that lives in a subdirectory of a different repo, sparsely cloned. This is the pattern for monorepos, which is likely closer to what Calcs' actual repo looks like.
- **Try renaming a plugin** and adding a `renames` entry in `marketplace.json` so existing installs auto-migrate instead of breaking with `plugin-not-found`.
