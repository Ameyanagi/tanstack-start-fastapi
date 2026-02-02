# Agent Guidance

This repository contains an Agent Skill for initializing full-stack monorepos.

## Repository Structure

- `skills/init-tanstack-fastapi/SKILL.md` - Main skill instructions
- `skills/init-tanstack-fastapi/references/stack-details.md` - All configuration file contents

## When modifying this skill

- Keep SKILL.md under 500 lines (currently ~180)
- All file contents belong in `references/stack-details.md`, not in SKILL.md
- The skill `name` in frontmatter must match the directory name
- Test changes by running `/init-tanstack-fastapi` in a clean directory
