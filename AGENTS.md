# AGENTS Instructions for /Users/devmayor/Documents/blockchain/docs

## Skills

A skill is a set of local instructions to follow that is stored in a `SKILL.md` file.

### Available skills

- skill-creator: Guide for creating effective skills. Use when creating or updating a skill. (file: /Users/devmayor/.codex/skills/.system/skill-creator/SKILL.md)
- skill-installer: Install Codex skills into `$CODEX_HOME/skills` from curated lists or GitHub repo paths. (file: /Users/devmayor/.codex/skills/.system/skill-installer/SKILL.md)
- slides: Build, edit, render, import, and export presentation decks with the artifacts tool. (file: /Users/devmayor/.codex/skills/.system/slides/SKILL.md)
- spreadsheets: Build, edit, recalculate, import, and export spreadsheet workbooks with the artifacts tool. (file: /Users/devmayor/.codex/skills/.system/spreadsheets/SKILL.md)
- mintlify: Build and maintain documentation sites with Mintlify; use for docs pages, navigation, components, and API references. (file: /Users/devmayor/.agents/skills/mintlify/SKILL.md)

### How to use skills

- Discovery: The list above is the skills available in this session (name + description + file path).
- Trigger rules: If the user names a skill (with `$SkillName` or plain text) OR the task clearly matches a skill's description shown above, you must use that skill for that turn. Multiple mentions mean use them all. Do not carry skills across turns unless re-mentioned.
- Missing/blocked: If a named skill is not in the list or the path cannot be read, say so briefly and continue with the best fallback.

### Skill workflow (progressive disclosure)

1. After deciding to use a skill, open its `SKILL.md`. Read only enough to follow the workflow.
2. When `SKILL.md` references relative paths (for example, `scripts/foo.py`), resolve them relative to the skill directory first.
3. If `SKILL.md` points to extra folders such as `references/`, load only the specific files needed.
4. If `scripts/` exist, prefer running or patching them instead of retyping large code blocks.
5. If `assets/` or templates exist, reuse them instead of recreating from scratch.

### Coordination and sequencing

- If multiple skills apply, choose the minimal set that covers the request and state the order you will use them.
- Announce which skill(s) you are using and why (one short line). If you skip an obvious skill, say why.

### Context hygiene

- Keep context small: summarize long sections instead of pasting them.
- Avoid deep reference-chasing unless blocked.
- When variants exist, pick only the relevant reference file(s) and note that choice.

### Safety and fallback

- If a skill cannot be applied cleanly (missing files, unclear instructions), state the issue, pick the next-best approach, and continue.
