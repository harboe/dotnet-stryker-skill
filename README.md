# Agent Skills

A collection of agent skills for .NET development workflows.

## Install

### Using skills-cli

Install individual skills with [skills-cli](https://www.npmjs.com/package/skills-cli):

```bash
# Install globally
npx skills-cli add https://github.com/harboe/skills.git -g

# Install into current project
npx skills-cli add https://github.com/harboe/skills.git
```

### Manual (symlinks)

Clone the repo and symlink individual skill folders:

```bash
# Clone once
git clone https://github.com/harboe/skills.git ~/.local/share/agent-skills

# Global -- available in all projects
ln -s ~/.local/share/agent-skills/dotnet-stryker-mutations ~/.config/opencode/skills/dotnet-stryker-mutations

# Project -- available in a single repo (from project root)
ln -s /path/to/skills/dotnet-stryker-mutations .opencode/skills/dotnet-stryker-mutations
```

Skills are discovered from these locations:

| Scope | Path |
|-------|------|
| Project | `.opencode/skills/<name>/SKILL.md` |
| Project | `.agents/skills/<name>/SKILL.md` |
| Global | `~/.config/opencode/skills/<name>/SKILL.md` |
| Global | `~/.agents/skills/<name>/SKILL.md` |

## Upgrade

```bash
# skills-cli
npx skills-cli update -g

# Manual
cd ~/.local/share/agent-skills && git pull
```

## Testing

- **dotnet-stryker-mutations** — Run Stryker.NET mutation testing to evaluate test quality, identify weak tests, and kill surviving mutants.