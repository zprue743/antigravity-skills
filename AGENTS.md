# Repository rules

This repository builds and stores reusable Google Antigravity skills.

## Skill structure

- Follow [Google's official skills documentation](https://antigravity.google/docs/skills/). Check it when creating or changing a skill's structure.
- Store each skill in `.agents/skills/<skill-name>/SKILL.md`, using a unique lowercase, hyphenated folder name.
- Start `SKILL.md` with YAML frontmatter. Include `name` matching the folder and a `description` explaining what the skill does and when to use it. Google requires `description`; explicit `name` is a repository convention.
- Write actionable Markdown instructions below the frontmatter, covering when to use the skill and how to complete its task.
- Add `scripts/`, `examples/`, and `resources/` inside the skill folder only as needed. Reference supporting files with relative paths.

## Working conventions

- Keep each skill focused on one task and instructions concise. Add decision guidance only when useful.
- Keep reusable assets self-contained; avoid machine-specific paths and secrets.
- For helper scripts, document prerequisites, usage, and inputs/outputs; provide `--help` where practical.
- Before finishing, check frontmatter, naming, referenced files, and alignment with Google's structure. Run relevant checks for any changed scripts and report what was verified.
