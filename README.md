# GetGo Web Skills

Shared Claude Code slash commands for the web platform team.

## Setup

This repo is intended to be consumed as a git submodule:

```bash
git submodule add https://github.com/GetGo-Technologies/getgo-web-skills .claude/skills
git submodule update --init
```

To update to the latest skills in a consuming repo:

```bash
git submodule update --remote .claude/skills
git commit -m "chore: update shared skills ref"
```

## Contributing

Add new skills as `.md` files in the root of this repo. Follow the naming convention `kebab-case.md`. Update this README with inputs, outputs, and usage before merging.