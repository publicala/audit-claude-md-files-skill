# audit-claude-md-files

Claude Code skill - audits every loaded CLAUDE.md line by line and prunes what a session can derive on its own, with every cut backed by evidence.

The CLAUDE.md trio: [feed-claude-md-files](https://github.com/publicala/feed-claude-md-files-skill) adds rules from observed patterns, [bake-claude-md-files](https://github.com/publicala/bake-claude-md-files-skill) converts crystallized rules into tooling, and [audit-claude-md-files](https://github.com/publicala/audit-claude-md-files-skill) prunes and verifies what remains. Install all three from [publicala/claude-plugins](https://github.com/publicala/claude-plugins).

## How it works

1. Inventories every resident CLAUDE.md with its estimated token cost
2. Cuts what a fresh session can derive with a few tool calls (setup commands, stack inventories, generic best practices)
3. Verifies every "the tooling enforces this" claim against the actual configs and tests, then classifies each rule by its feedback loop
4. Counts what the codebase already teaches: conventions the code demonstrates get cut, conventions the code contradicts stay
5. Judges borderline blocks with independent low-effort agents (docs serve the weakest model that reads them) and verifies examples against real APIs
6. Audits pointers and skill descriptions: a pointer carries the trigger and the path, never a content summary
7. Presents the full evidence-backed report, then applies approved cuts as granular commits with a PR

## Install

### Via Plugin Marketplace

```
/plugin marketplace add publicala/claude-plugins
/plugin install audit-claude-md-files@publicala
```

### Via skills.sh

```bash
npx skills add publicala/audit-claude-md-files-skill
```

### Manual

Copy `skills/audit-claude-md-files/SKILL.md` into your skills directory:

```bash
# Global (all projects)
mkdir -p ~/.claude/skills/audit-claude-md-files
cp skills/audit-claude-md-files/SKILL.md ~/.claude/skills/audit-claude-md-files/

# Project-level
mkdir -p .claude/skills/audit-claude-md-files
cp skills/audit-claude-md-files/SKILL.md .claude/skills/audit-claude-md-files/
```

## Usage

Run it in the project you want audited. The command depends on how you installed it:

- **skills.sh / manual**: `/audit-claude-md-files`
- **Plugin marketplace**: `/audit-claude-md-files:audit-claude-md-files` (plugin skills are namespaced as `/<plugin>:<skill>`)

## Resources

- [feed-claude-md-files](https://github.com/publicala/feed-claude-md-files-skill) - Surfaces patterns into new CLAUDE.md rules
- [bake-claude-md-files](https://github.com/publicala/bake-claude-md-files-skill) - Converts CLAUDE.md rules into automated checks
- [CLAUDE.md Guide](https://github.com/publicala/claude-md-guide) - Presentation slides about CLAUDE.md files
- [CLAUDE.md docs](https://docs.anthropic.com/en/docs/claude-code/memory) - Official documentation

## License

MIT
