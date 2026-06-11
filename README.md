# GetGo Web Skills

Shared Claude Code slash commands for the web platform team.

## Setup

Clone this repo into `.claude/skills` inside your project, then gitignore it so nothing gets committed:

```bash
git clone https://github.com/GetGo-Technologies/getgo-web-skills .claude/skills
echo ".claude/skills" >> .gitignore
```

To update to the latest skills:

```bash
git -C .claude/skills pull
```

## Contributing

Add new skills as `.md` files in the root of this repo. Follow the naming convention `kebab-case.md`. Update this README with inputs, outputs, and usage before merging.