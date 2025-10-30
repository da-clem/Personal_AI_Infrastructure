# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

**PAI (Personal AI Infrastructure)** is an open-source personal AI system designed to augment humans with AI capabilities. It provides a modular, extensible framework for managing your life and work through AI assistants.

**Core Mission:** Make powerful AI accessible to everyone by providing a complete, open-source platform that works with Claude, GPT, Gemini, or any AI system.

## Repository Structure

As of v0.6.0, all PAI infrastructure lives in the `.claude/` directory:

```
.claude/
├── agents/              # Specialized AI agents (researcher, engineer, designer, pentester, architect)
├── commands/            # Custom slash commands for Claude Code
├── documentation/       # System documentation
├── hooks/               # Event-driven automation scripts (TypeScript)
├── skills/              # Modular capability packages (intent-based activation)
├── voice-server/        # Text-to-speech notification service (Bun-based)
├── settings.json        # Claude Code configuration with permissions, hooks, MCP servers
├── .mcp.json            # MCP (Model Context Protocol) server configurations
├── .env.example         # Environment variable template
├── setup.sh             # Automated setup script
└── PAI.md               # Core system identity and global context
```

**Installation Location:** Users copy `.claude/` from repo to `~/.claude/` on their system.

## Development Commands

### Testing & Development

```bash
# Run voice server (requires Bun)
cd ~/.claude/voice-server && bun server.ts

# Test voice server health
curl http://localhost:8888/health

# Run setup script
./.claude/setup.sh
```

### Package Management

- **JavaScript/TypeScript**: Use `bun` (NOT npm, yarn, or pnpm)
- **Python**: Use `uv` (NOT pip) if needed

### Testing Hook Scripts

Hooks are TypeScript files executed by Claude Code. Test manually:

```bash
# Example: Test a hook
bun ~/.claude/hooks/update-tab-titles.ts
```

## Architecture

### Skills System

PAI uses a **skills-based architecture** for modular capabilities. Skills activate based on user intent.

**Skill Structure:**
- `SKILL.md` (required): Quick reference with YAML frontmatter containing name/description
- `CLAUDE.md` (optional): Comprehensive methodology for complex skills
- Components (optional): Subdirectories with templates and resources

**Progressive Disclosure:**
1. Skill description (always in system prompt) → triggers intent matching
2. SKILL.md body → loaded when skill activates
3. CLAUDE.md and components → loaded as needed

**Available Skills:**
- `PAI/` - Core PAI identity and global context
- `research/` - Multi-source research with parallel agents
- `fabric/` - 242+ AI patterns for content processing
- `prompting/` - Prompt engineering standards
- `create-skill/` - Skill creation framework
- `alex-hormozi-pitch/` - $100M Offers pitch framework
- `ffuf/` - Web fuzzing for penetration testing
- `agent-observability/` - Tracking and analysis

### Hook System

Event-driven automation via TypeScript hooks in `.claude/hooks/`:

- `SessionStart` hooks: Initialize PAI session, load core context
- `UserPromptSubmit` hooks: Update tab titles, capture events
- `PreToolUse`/`PostToolUse` hooks: Tool execution monitoring
- `SessionEnd` hooks: Capture session summaries
- `PreCompact` hooks: Context compression

All hooks use `${PAI_DIR}` variable for portability.

### Agent System

Specialized agents in `.claude/agents/`:
- `researcher.md` - General research agent
- `perplexity-researcher.md` - Real-time web research (requires PERPLEXITY_API_KEY)
- `claude-researcher.md` - Deep analysis (built-in to Claude Code)
- `gemini-researcher.md` - Multi-source research (requires GOOGLE_API_KEY)
- `engineer.md` - Software development
- `architect.md` - System design
- `designer.md` - UI/UX design
- `pentester.md` - Security testing

### MCP Integration

PAI integrates with Model Context Protocol servers (configured in `.mcp.json`):
- `Ref` - Documentation search across 100+ frameworks
- `playwright` - Browser automation
- `brightdata` - Web scraping with CAPTCHA bypass
- `apify` - Web automation
- `stripe` - Payment processing
- Custom MCP servers (httpx, content, daemon, naabu)

## Environment Configuration

### Required Variables

```bash
PAI_DIR="/path/to/PAI"           # Repository root (system agnostic)
PAI_HOME="$HOME"                 # User home directory
```

### Optional Variables

```bash
DA="Kai"                         # AI assistant name
DA_COLOR="purple"                # Display color
PORT="8888"                      # Voice server port
```

### API Keys (Optional)

Configure in `.claude/.env` (copy from `.env.example`):
- `PERPLEXITY_API_KEY` - For perplexity-researcher agent
- `GOOGLE_API_KEY` - For gemini-researcher agent
- `ANTHROPIC_API_KEY` - For Claude API features
- `OPENAI_API_KEY` - For GPT integration
- `REPLICATE_API_TOKEN` - For AI image/video generation
- `ELEVENLABS_API_KEY` - For premium voice synthesis

## Key Files to Understand

### Core Configuration

- **`.claude/settings.json`**: Claude Code settings, permissions, hooks, statusline
- **`.claude/PAI.md`**: Global context with identity, stack preferences, security rules
- **`.claude/.mcp.json`**: MCP server definitions and authentication

### Documentation

- **`.claude/documentation/architecture.md`**: System architecture overview
- **`.claude/documentation/skills-system.md`**: Skills system comprehensive guide
- **`.claude/documentation/how-to-start.md`**: Getting started guide
- **`README.md`**: Project overview, features, installation

## Development Guidelines

### Security Practices

From PAI.md security section:
- **NEVER commit sensitive data to public repos**
- **ALWAYS verify repository before committing** (`git remote -v`)
- **CHECK THREE TIMES** before git operations
- `~/.claude/` may contain private data - be extremely careful
- Before public commits, ensure NO credentials, keys, passwords, or personal data

### Working Directory

- **Scratchpad for experiments**: Use `~/.claude/scratchpad/YYYY-MM-DD-HHMMSS_description/`
- **NEVER drop random projects** directly in `~/.claude/` directory
- Clean up scratchpad periodically

### Code Style

- **TypeScript preferred** over Python
- Use `bun` for JavaScript/TypeScript package management
- Use imperative form in instructions ("Create directory" not "You should create")
- Be specific and actionable ("Run `bun dev`" not "Start the application")

### Hooks Development

- All hooks are TypeScript files
- Use `spawn` with argument arrays (no shell interpretation)
- Validate all inputs
- Use environment variables for configuration
- Handle errors gracefully with fallbacks
- Log to `${PAI_DIR}/logs/` or `CLAUDE_HOOKS_LOG_DIR`

### Skills Development

When creating/modifying skills:
1. Always include YAML frontmatter with `name` and `description`
2. Include "USE WHEN" triggers in description
3. Use imperative instructions
4. Reference global context, don't duplicate
5. Follow progressive disclosure (SKILL.md → CLAUDE.md → Components)
6. Test with realistic user requests

## Common Patterns

### Adding a New Skill

```bash
# 1. Create skill directory
mkdir -p ~/.claude/skills/my-skill

# 2. Create SKILL.md with frontmatter
cat > ~/.claude/skills/my-skill/SKILL.md << 'EOF'
---
name: my-skill
description: What it does, when to use it. USE WHEN user says 'trigger phrase'...
---

# My Skill

## When to Activate This Skill
- User requests X
- Task involves Y

## Core Workflow
[Instructions here]
EOF

# 3. Test activation with natural language
```

### Adding a Hook

```typescript
// ~/.claude/hooks/my-hook.ts
#!/usr/bin/env bun

// Get environment variables
const PAI_DIR = process.env.PAI_DIR || `${process.env.HOME}/.claude`;

// Hook logic here

console.log("Hook completed successfully");
```

### Creating a Command

Commands are markdown files in `.claude/commands/`:

```markdown
---
name: my-command
description: Command description
---

# /my-command

Command implementation instructions...
```

## Version Notes

- **Current Version**: v0.6.0+
- **Breaking Changes in v0.6.0**: Repository restructured to use `.claude/` directory
- **Migration from v0.5.0**: Copy `.claude/` to `~/.claude/` on your system
- PAI_DIR now points to repository root (not `PAI_DIRECTORY` subdirectory)

## External Documentation

- **Main Project**: https://github.com/danielmiessler/Personal_AI_Infrastructure
- **Claude Code**: https://claude.ai/code
- **Anthropic Skills**: https://github.com/anthropics/skills
- **Daniel Miessler's Blog**: https://danielmiessler.com/blog/personal-ai-infrastructure
