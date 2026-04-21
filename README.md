> **Note:** This repository contains Anthropic's implementation of skills for Claude. For information about the Agent Skills standard, see [agentskills.io](http://agentskills.io).

# Skills
Skills are folders of instructions, scripts, and resources that Claude loads dynamically to improve performance on specialized tasks. Skills teach Claude how to complete specific tasks in a repeatable way, whether that's creating documents with your company's brand guidelines, analyzing data using your organization's specific workflows, or automating personal tasks.

For more information, check out:
- [What are skills?](https://support.claude.com/en/articles/12512176-what-are-skills)
- [Using skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
- [How to create custom skills](https://support.claude.com/en/articles/12512198-creating-custom-skills)
- [Equipping agents for the real world with Agent Skills](https://anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

# About This Repository

This repository contains skills that demonstrate what's possible with Claude's skills system. These skills range from creative applications (art, music, design) to technical tasks (testing web apps, MCP server generation) to enterprise workflows (communications, branding, etc.).

Each skill is self-contained in its own folder with a `SKILL.md` file containing the instructions and metadata that Claude uses. Browse through these skills to get inspiration for your own skills or to understand different patterns and approaches.

Many skills in this repo are open source (Apache 2.0). We've also included the document creation & editing skills that power [Claude's document capabilities](https://www.anthropic.com/news/create-files) under the hood in the [`skills/docx`](./skills/docx), [`skills/pdf`](./skills/pdf), [`skills/pptx`](./skills/pptx), and [`skills/xlsx`](./skills/xlsx) subfolders. These are source-available, not open source, but we wanted to share these with developers as a reference for more complex skills that are actively used in a production AI application.

## Custom Skills

This fork includes additional skills for Python and Go development:

- **LangGraph** - Build stateful AI applications with LangGraph in Python (core, patterns, production, troubleshooting)
- **Python Style** - Python formatting, language rules, and type annotation best practices
- **Go Style** - Go formatting, language rules, and testing best practices

These custom skills are licensed under MIT. See the respective `LICENSE.txt` files in each skill directory.

## Disclaimer

**These skills are provided for demonstration and educational purposes only.** While some of these capabilities may be available in Claude, the implementations and behaviors you receive from Claude may differ from what is shown in these skills. These skills are meant to illustrate patterns and possibilities. Always test skills thoroughly in your own environment before relying on them for critical tasks.

# Skill Sets

**Original Anthropic Skills:**
- Creative & Design: algorithmic-art, brand-guidelines, canvas-design, frontend-design, theme-factory
- Development & Technical: claude-api, mcp-builder, skill-creator, web-artifacts-builder, webapp-testing
- Enterprise & Communication: doc-coauthoring, internal-comms, slack-gif-creator
- Document Skills: docx, pdf, pptx, xlsx

**Custom Skills (MIT Licensed):**
- LangGraph: langgraph-python-core, langgraph-python-patterns, langgraph-python-production, langgraph-python-troubleshooting
- Python Style: python-language-rules, python-style-rules, python-type-annotations
- Go Style: go-style-rules, go-language-rules, go-testing

- [./spec](./spec): The Agent Skills specification
- [./template](./template): Skill template

# Try in Claude Code, Claude.ai, and the API

## Claude Code
You can register this repository as a Claude Code Plugin marketplace by running the following command in Claude Code:
```
/plugin marketplace add ranjitjana027/skills
```

Then, to install a specific set of skills:
1. Select `Browse and install plugins`
2. Select `anthropic-agent-skills`
3. Choose from the available plugins:
   - `document-skills` - docx, pdf, pptx, xlsx
   - `example-skills` - algorithmic-art, mcp-builder, skill-creator, etc.
   - `claude-api` - Claude API integration
   - `langgraph-skills` - LangGraph Python (core, patterns, production, troubleshooting)
   - `python-style` - Python language, formatting, and type annotations
   - `go-style` - Go language, formatting, and testing
4. Select `Install now`

Alternatively, directly install any plugin via:
```
/plugin install document-skills@anthropic-agent-skills
/plugin install langgraph-skills@anthropic-agent-skills
/plugin install python-style@anthropic-agent-skills
/plugin install go-style@anthropic-agent-skills
```

After installing the plugin, you can use the skill by just mentioning it. For instance:
- "Use the PDF skill to extract form fields from `path/to/file.pdf`"
- "Use the python-style-rules skill to review this code"
- "Use the langgraph skill to build a stateful agent"

## Claude.ai

These example skills are all already available to paid plans in Claude.ai. 

To use any skill from this repository or upload custom skills, follow the instructions in [Using skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude#h_a4222fa77b).

## Claude API

You can use Anthropic's pre-built skills, and upload custom skills, via the Claude API. See the [Skills API Quickstart](https://docs.claude.com/en/api/skills-guide#creating-a-skill) for more.

# Creating a Basic Skill

Skills are simple to create - just a folder with a `SKILL.md` file containing YAML frontmatter and instructions. You can use the **template-skill** in this repository as a starting point:

```markdown
---
name: my-skill-name
description: A clear description of what this skill does and when to use it
---

# My Skill Name

[Add your instructions here that Claude will follow when this skill is active]

## Examples
- Example usage 1
- Example usage 2

## Guidelines
- Guideline 1
- Guideline 2
```

The frontmatter requires only two fields:
- `name` - A unique identifier for your skill (lowercase, hyphens for spaces)
- `description` - A complete description of what the skill does and when to use it

The markdown content below contains the instructions, examples, and guidelines that Claude will follow. For more details, see [How to create custom skills](https://support.claude.com/en/articles/12512198-creating-custom-skills).

# Partner Skills

Skills are a great way to teach Claude how to get better at using specific pieces of software. As we see awesome example skills from partners, we may highlight some of them here:

- **Notion** - [Notion Skills for Claude](https://www.notion.so/notiondevs/Notion-Skills-for-Claude-28da4445d27180c7af1df7d8615723d0)
