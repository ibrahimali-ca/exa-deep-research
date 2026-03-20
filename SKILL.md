---
name: deep-research
description: Deep research using Exa AI. Use when the user needs comprehensive research, literature review, multi-source synthesis, or asks to "research", "deep search", "look into", or "find out about" a topic.
user-invocable: true
argument-hint: [fast|balanced|pro] topic or question
allowed-tools:
  - mcp__exa__deep_researcher_start
  - mcp__exa__deep_researcher_check
  - mcp__exa__deep_search_exa
  - mcp__exa__web_search_exa
  - mcp__exa__crawling_exa
  - mcp__exa__get_code_context_exa
  - mcp__exa__company_research_exa
---

# Exa Deep Research

Run comprehensive research using Exa AI's suite of search and research tools.

## Argument Parsing

Parse the user's input from `$ARGUMENTS`:

1. **Model tier prefix** (optional): If the query starts with `fast:`, `pro:`, or `balanced:`, extract it as the tier. Default tier is `fast`.
2. **Research query**: Everything after the optional prefix.

| Prefix | Model | Typical time | Use case |
|--------|-------|-------------|----------|
| `fast:` (default) | `exa-research-fast` | ~15s | Quick overviews, simple questions |
| `balanced:` | `exa-research` | 15-45s | Most research tasks |
| `pro:` | `exa-research-pro` | 45s-3min | Complex topics needing depth |

## Tool Routing

Choose the right tool based on the query content:

### 1. URL provided → `crawling_exa`
If the query contains a URL (starts with `http://` or `https://`), use `crawling_exa` to extract the page content. Set `maxCharacters` to 10000 for full extraction. Then summarize the content for the user.

### 2. Company name query → `company_research_exa`
If the query is specifically about a company (e.g., "research Stripe", "tell me about OpenAI"), use `company_research_exa` with the company name.

### 3. Programming/code question → `get_code_context_exa`
If the query is about code, APIs, libraries, SDKs, or programming patterns, use `get_code_context_exa`. Set `tokensNum` to 8000 for thorough results.

### 4. Quick factual lookup → `web_search_exa`
If the query is a simple factual question (who, what, when, where) that doesn't need synthesis, use `web_search_exa` for speed.

### 5. Focused question needing citations → `deep_search_exa`
If the query needs multi-angle search but not a full report, use `deep_search_exa`:
- Default type: `deep`
- Use `deep-reasoning` for queries that need analysis (triggered by `pro:` prefix or complex phrasing)

### 6. Complex research (default) → `deep_researcher_start` + `deep_researcher_check`
For anything that needs a comprehensive report with synthesis — this is the **default path** for most research queries.

**Steps:**
1. Call `deep_researcher_start` with:
   - `instructions`: The full research query, enriched with specificity if the user's query is vague
   - `model`: Based on the tier prefix (see table above)
2. Tell the user: "Starting Exa deep research (model: {model})... This will take {expected time}."
3. **Poll** by calling `deep_researcher_check` with the returned `researchId`:
   - If status is NOT `completed`, wait 5 seconds, then call again
   - Repeat until status is `completed`
   - Do NOT call more than 30 times (timeout safety)
4. Present the completed report to the user

## Output Format

When presenting results:

1. **Lead with the answer** — synthesize the key findings in 2-3 paragraphs
2. **Citations** — list sources with titles and URLs
3. **Offer to save** — if the report is longer than ~500 words, ask: "Want me to save this report to a file?"

## Examples

```
/exa-deep-research what is retrieval augmented generation
→ Routes to deep_researcher_start with exa-research-fast

/exa-deep-research pro: comparative analysis of transformer architectures for education
→ Routes to deep_researcher_start with exa-research-pro

/exa-deep-research https://arxiv.org/abs/2401.12345
→ Routes to crawling_exa

/exa-deep-research how does the Anthropic Python SDK handle streaming
→ Routes to get_code_context_exa

/exa-deep-research Notion company
→ Routes to company_research_exa

/exa-deep-research who won the 2024 Nobel Prize in Physics
→ Routes to web_search_exa
```
