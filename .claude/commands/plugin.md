---
description: Plugin manager — list, search, inspect, scaffold, and publish skill plugins in the marketplace. Usage: /plugin <list|search|info|create|publish> [args]
---

Parse `$ARGUMENTS` to extract the sub-command and any arguments. If no sub-command is given, show usage.

---

## list

Read `.claude-plugin/marketplace.json`. Parse the `plugins` array. Display each plugin grouped by category with: name, one-sentence description, version. Sort alphabetically within each category.

Example output:
```
PLUGIN MARKETPLACE  (235 plugins, v2.3.0)

Engineering (45)
  agenthub         Multi-agent orchestration framework    v2.3.0
  ...
```

---

## search `<query>`

Read `.claude-plugin/marketplace.json`. Match `<query>` (case-insensitive) against each plugin's `name`, `description`, and `keywords`. Rank: exact name match > keyword match > description match. Show top 10 results with name, path, and description.

---

## info `<name>`

Search `.claude-plugin/marketplace.json` for a plugin whose `name` or `source` matches `<name>`. If found, also read the skill's `SKILL.md` and `.claude-plugin/plugin.json` if they exist. Display: path, category, version, description, keywords, script count, reference count, and associated command if any.

---

## create `<name> [domain]`

Scaffold a new plugin. `<name>` must be kebab-case. `[domain]` is one of: `engineering`, `engineering-team`, `product-team`, `marketing-skill`, `c-level-advisor`, `business-growth`, `finance`, `ra-qm-team`, `project-management`. If domain is omitted, ask the user.

Create this directory structure at `{domain}/{name}/`:
- `SKILL.md` — pre-filled frontmatter with name, description placeholder, tier STANDARD, category from domain, version 2.3.0, platforms [claude-code, codex, cursor], and stub workflow sections
- `.claude-plugin/plugin.json` — pre-filled with name, description placeholder, version 2.3.0, author stub, license MIT, skills "./"
- `scripts/` — empty directory
- `references/` — empty directory
- `assets/` — empty directory

After creating files, show the path and next steps: fill SKILL.md, add scripts, run `/plugin-audit {domain}/{name}`, then `/plugin publish {domain}/{name}`.

---

## publish `<skill-path>`

Register a skill in the marketplace.

1. Read `{skill-path}/SKILL.md` — extract name, description, category, tags, version from frontmatter
2. Warn if version does not match `2.3.0`
3. Read `.claude-plugin/marketplace.json`
4. Build marketplace entry: `{ "name", "source": "./{skill-path}", "description", "version": "2.3.0", "keywords": [tags], "category" }`
5. If an entry with the same `source` already exists, update it. Otherwise append it.
6. Write the updated `marketplace.json`
7. Confirm: show the entry that was added/updated and remind the user to run `/plugin-audit {skill-path}` before merging
