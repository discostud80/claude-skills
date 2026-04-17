---
name: plugin
description: |
  Plugin manager for the claude-skills marketplace. Browse, search, inspect, scaffold, and register
  skill plugins. Usage: /plugin <list|search|info|create|publish> [args]
---

# /plugin

Plugin manager for the claude-skills marketplace. Provides sub-commands to browse available plugins, inspect plugin details, scaffold new plugins, and register them in the marketplace.

## Usage

```bash
/plugin list                        List all plugins in the marketplace
/plugin search <query>              Search plugins by keyword
/plugin info <name>                 Show details for a specific plugin
/plugin create <name> [domain]      Scaffold a new plugin in the given domain
/plugin publish <skill-path>        Register a skill as a plugin in the marketplace
```

## Sub-Commands

---

### list

Display all plugins registered in `.claude-plugin/marketplace.json`.

1. Read `.claude-plugin/marketplace.json`
2. Parse the `plugins` array
3. For each entry, extract: `name`, `description` (first sentence), `version`, `category`, `keywords`
4. Display as a formatted table sorted by category then name

**Output format:**
```
PLUGIN MARKETPLACE  (235 plugins, v2.3.0)

Engineering (45)
  agenthub              Multi-agent orchestration framework         v2.3.0
  agent-designer        Design and scaffold AI agent systems        v2.3.0
  ...

Marketing (44)
  copywriting           AI-powered copywriting workflows            v2.3.0
  ...
```

---

### search `<query>`

Search plugins by name, description, or keyword.

1. Read `.claude-plugin/marketplace.json`
2. Match query (case-insensitive) against: `name`, `description`, `keywords`
3. Rank results: exact name match > keyword match > description match
4. Display top 10 matches with name, description, category, and path

**Output format:**
```
Search results for "seo" (8 matches)

  seo-audit         marketing-skill/seo-audit           SEO technical audit and optimization
  seo-auditor       engineering/seo-auditor             Doc-level SEO scanning command
  ...
```

---

### info `<name>`

Show full details for a plugin by name or partial path.

1. Search `.claude-plugin/marketplace.json` for the plugin by name
2. If not found, search subdirectories for a matching `SKILL.md`
3. Read the skill's `SKILL.md` frontmatter and first section
4. If `.claude-plugin/plugin.json` exists, read it
5. Display full details

**Output format:**
```
Plugin: seo-audit
  Path:        marketing-skill/seo-audit/
  Category:    Marketing
  Version:     v2.3.0
  Description: Technical SEO audit with keyword gap analysis
  Keywords:    seo, audit, keywords, backlinks
  Scripts:     3 Python tools
  References:  4 guides
  Command:     /seo-audit (commands/seo-audit.md)
  Plugin.json: marketing-skill/seo-audit/.claude-plugin/plugin.json
```

---

### create `<name>` `[domain]`

Scaffold a new plugin in the given domain directory.

**Domain defaults:** If domain is not specified, detect from context or prompt the user.

**Valid domains:** `engineering`, `engineering-team`, `product-team`, `marketing-skill`, `c-level-advisor`, `business-growth`, `finance`, `ra-qm-team`, `project-management`

**Steps:**
1. Validate `name` is kebab-case (lowercase, hyphens only)
2. Confirm target path: `{domain}/{name}/`
3. Check path does not already exist
4. Scaffold the following structure:

```
{domain}/{name}/
├── SKILL.md              # Pre-filled with frontmatter from user's name/domain
├── .claude-plugin/
│   └── plugin.json       # Pre-filled with name, description stub, version 2.3.0
├── scripts/              # Empty, ready for Python tools
├── references/           # Empty, ready for knowledge base docs
└── assets/               # Empty, ready for templates
```

**SKILL.md template:**
```yaml
---
name: "{name}"
description: "TODO: one-line description of what this plugin does"
tier: STANDARD
category: {Domain}
tags: []
version: 2.3.0
author: ""
license: MIT
platforms: [claude-code, codex, cursor]
---

# {Name}

TODO: Overview of what this skill does and when to use it.

## Workflows

### Phase 1: TODO

- [ ] Step 1
- [ ] Step 2

## References

- `references/` — Add domain knowledge guides here

## Scripts

- `scripts/` — Add Python CLI tools here

## Related Skills

- TODO: link to related skills
```

**plugin.json template:**
```json
{
  "name": "{name}",
  "description": "TODO: description",
  "version": "2.3.0",
  "author": { "name": "" },
  "license": "MIT",
  "skills": "./"
}
```

After scaffolding, display:
```
Created: {domain}/{name}/
  Next steps:
  1. Fill in SKILL.md — description, workflows, and references
  2. Add Python tools to scripts/ (stdlib-only, CLI-first)
  3. Add knowledge docs to references/
  4. Run /plugin-audit {domain}/{name} when ready
  5. Run /plugin publish {domain}/{name} to register in marketplace
```

---

### publish `<skill-path>`

Register an existing skill as a plugin in `.claude-plugin/marketplace.json`.

**Pre-flight checks:**
1. Verify `{skill_path}/SKILL.md` exists
2. Read frontmatter — extract `name`, `description`, `category`, `tags`, `version`
3. Check that version matches repo version (`2.3.0`). If not, warn.
4. Check if already registered in `marketplace.json` — if so, update the entry

**Steps:**
1. Read `.claude-plugin/marketplace.json`
2. Build new marketplace entry:
   ```json
   {
     "name": "{name}",
     "source": "./{skill_path}",
     "description": "{description}",
     "version": "2.3.0",
     "author": "{author}",
     "keywords": ["{tags}"],
     "category": "{category}"
   }
   ```
3. Add or update the entry in the `plugins` array
4. Write updated `marketplace.json`
5. Verify the write succeeded

**Output:**
```
Published: {name} → .claude-plugin/marketplace.json
  Source:   ./{skill_path}
  Category: {category}
  Keywords: {tags}

  Run /plugin-audit {skill_path} to validate before merging.
```

---

## Skill References

| Tool | Path |
|------|------|
| Marketplace registry | `.claude-plugin/marketplace.json` |
| Plugin audit command | `commands/plugin-audit.md` |
| Skill validator | `engineering/skill-tester/scripts/skill_validator.py` |
| Quality scorer | `engineering/skill-tester/scripts/quality_scorer.py` |
