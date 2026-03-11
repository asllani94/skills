# Skills

A collection of agentic skills for [GitHub Copilot](https://github.com/features/copilot), designed to be used with `npx skills` and published to [skills.sh](https://skills.sh).

## Overview

This repository contains custom skills that extend GitHub Copilot's capabilities with domain-specific knowledge and workflows. Skills are self-contained expert knowledge bases that help developers work more effectively in specialized contexts.

## Usage

### Running Skills

```bash
npx skills
```

### Installing a Skill

Skills from this repository can be installed directly from [skills.sh](https://skills.sh):

```bash
npx skills install <skill-name>
```

## Repository Structure

```
skills/
└── <skill-name>/
    ├── SKILL.md      # Main skill definition
    ├── agents/       # Agent definitions (if applicable)
    ├── scripts/      # Utility scripts
    └── ...
```

## Available Skills

| Skill | Description |
|-------|-------------|
| `ing-skill-generator` | Generate GitHub Copilot skills from documentation repositories. Includes evaluation framework, grading agents, and benchmark tools. |

## Creating New Skills

Each skill should include:

1. **SKILL.md** - The main skill file with frontmatter metadata and comprehensive knowledge base
2. **LICENSE.txt** - License information
3. **NOTICE.md** - Attribution and modification details (if derivative work)

### Skill Frontmatter

```yaml
---
name: your-skill-name
description: >
  A clear description of what the skill does and when it should be triggered.
  Include keywords that users might use to invoke the skill.
---
```

## Publishing

Skills are published to [skills.sh](https://skills.sh), a skill registry maintained by [Vercel](https://vercel.com).

## Contributing

1. Fork this repository
2. Create your skill in `.github/skills/<skill-name>/`
3. Test your skill locally with `npx skills`
4. Submit a pull request

## License

See individual skill directories for specific license information.
