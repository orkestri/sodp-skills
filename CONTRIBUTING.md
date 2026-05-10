# Contributing to sodp-skills

Contributions are welcome — new skills, improvements to existing ones, and bug fixes.

## What are skills?

Each skill is a Markdown file in `.claude/commands/`. When a user types `/skill-name` in Claude Code, Claude reads the file and follows its instructions to perform a task in the user's project.

Good skills are:
- **Specific** — they tell Claude exactly what to detect, generate, and explain
- **Self-contained** — they include all the code snippets Claude needs, so no hallucination
- **Actionable** — they end with a clear result the user can verify

## Adding a new skill

1. Fork the repo and create a branch: `git checkout -b feat/sodp-my-skill`
2. Add your skill file to `.claude/commands/sodp-my-skill.md`
3. Follow the existing structure:
   - What the skill does (first paragraph)
   - What `$ARGUMENTS` accepts
   - Numbered steps Claude should follow
   - Code snippets for each supported language/framework
4. Update the command table in `README.md`
5. Open a pull request

## Skill file conventions

- Filename: `sodp-<name>.md` — always prefixed with `sodp-`
- Use `$ARGUMENTS` to refer to what the user typed after the slash command
- Include working code snippets — prefer copy-pasteable over abstract descriptions
- Cover all supported languages where applicable (TypeScript, React, Python, Java)
- End with a "tell the user" step summarising what was done and what to do next

## Improving existing skills

If a skill generates incorrect code, is missing a framework, or has a better pattern — open a PR with the fix. Include a brief description of what was wrong and what changed.

## Reporting issues

Open a GitHub issue describing:
- Which skill (`/sodp-setup`, `/sodp-watch`, etc.)
- What language/framework you were using
- What Claude generated vs. what you expected
