# /deep-research — Exa AI Skill for Claude Code

A Claude Code skill that wraps [Exa AI's](https://exa.ai) MCP tools into a single `/deep-research` slash command. Ask a question, get a comprehensive research report with citations.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- [Exa MCP server](https://github.com/exa-labs/exa-mcp-server) configured with an API key

## Install

```bash
# Clone into your Claude Code skills directory
git clone https://github.com/ibrahimali-ca/exa-deep-research.git ~/.claude/skills/exa-deep-research
```

Or copy `SKILL.md` manually into `~/.claude/skills/exa-deep-research/SKILL.md`.

## Usage

```
/deep-research what is retrieval augmented generation
```

### Model tiers

Prefix your query with a tier to control depth vs speed:

| Prefix | Model | Time | Best for |
|--------|-------|------|----------|
| *(none)* / `fast:` | `exa-research-fast` | ~15s | Quick overviews |
| `balanced:` | `exa-research` | 15-45s | Most research |
| `pro:` | `exa-research-pro` | 45s-3min | Complex topics |

```
/deep-research pro: comparative analysis of transformer architectures for education
```

## Smart routing

The skill automatically picks the best Exa tool based on your query:

| Query type | Tool used | Example |
|-----------|-----------|---------|
| URL provided | `crawling_exa` | `/deep-research https://arxiv.org/abs/2401.12345` |
| Company name | `company_research_exa` | `/deep-research Notion company` |
| Code/API question | `get_code_context_exa` | `/deep-research how does the Anthropic SDK handle streaming` |
| Quick fact | `web_search_exa` | `/deep-research who won the 2024 Nobel Prize in Physics` |
| Focused + citations | `deep_search_exa` | `/deep-research effects of spaced repetition on retention` |
| Complex research | `deep_researcher_start` | `/deep-research what is retrieval augmented generation` |

## How it works

For complex queries (the default path), the skill:

1. Calls `deep_researcher_start` with your query and chosen model tier
2. Polls `deep_researcher_check` until the report is complete
3. Presents a synthesized report with citations
4. Offers to save long reports to a file

## Exa MCP tools used

- `deep_researcher_start` / `deep_researcher_check` — async deep research agent
- `deep_search_exa` — multi-angle search with synthesis
- `web_search_exa` — fast web search
- `crawling_exa` — full page content extraction
- `get_code_context_exa` — code and documentation search
- `company_research_exa` — company intelligence

## License

MIT
