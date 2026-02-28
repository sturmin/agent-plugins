# Skills

My experiments with [Claude Agent Skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview).

## Usage

Add as a submodule at `.claude/shared-skills/` — this keeps your project-level
skills in `.claude/skills/` untouched:

```bash
git submodule add <repo-url> .claude/shared-skills
```

Then symlink whichever skills you want to activate into `.claude/skills/`:

```bash
mkdir -p .claude/skills
ln -s ../shared-skills/cli-design .claude/skills/cli-design
```

Project-level skills and shared skills coexist in `.claude/skills/`:

```
.claude/
├── skills/
│   ├── cli-design -> ../shared-skills/cli-design   # symlinked from this repo
│   └── my-project-skill/                           # project-specific
└── shared-skills/                                  # this repo
```


```bash
git submodule update --init
ln -s ../shared-skills/cli-design .claude/skills/cli-design
```

## Skills

| Skill | Description |
|-------|-------------|
| [cli-design](cli-design/SKILL.md) | Design and update CLI interface specifications following POSIX and GNU standards |
