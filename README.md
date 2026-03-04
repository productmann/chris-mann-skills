# Chris Mann Skills

A collection of Cowork skills — anyone with Claude Cowork can install and use them.

## Available Skills

### youcom-research
Research any topic using the You.com Research API. Returns deep, multi-step web research with cited sources and inline citations.

**Requires:** A You.com API key (free to get at [you.com/api](https://you.com/api))

**API Docs:** [You.com Research API Overview](https://docs.you.com/research/overview)

**How to invoke in Cowork:**

The most reliable way is to reference the skill by name in your message:

> *"use youcom-research to find the best electric cars of 2026"*

Saying something generic like *"research X"* may work but could instead use Cowork's built-in web search or the base AI model rather than the You.com Research API.

## Installing a Skill

1. Clone or download this repo
2. Copy the skill folder (e.g. `skills/youcom-research/`) into `~/.claude/skills/`
   - On Mac, open Finder → press **Cmd+Shift+G** → type `~/.claude/skills/`
3. Restart Cowork — the skill is immediately available

## First-Time API Key Setup (youcom-research)

1. Get your key at [you.com/api](https://you.com/api)
2. Open or create `~/.claude/.env`
3. Add this line: `YDC_API_KEY=your_key_here`
4. Restart Cowork

The skill will also walk you through this the first time you use it.
