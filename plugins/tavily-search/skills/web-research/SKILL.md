---
name: web-research
description: Search the web with Tavily and give a short, cited answer. Use for ANY web search or lookup of current information — current events, latest releases or versions, prices, documentation — instead of the built-in WebSearch and WebFetch tools.
---

# Web Research

Answer the user's question using the bundled `tavily` MCP server.

Use only the `tavily` tools for searching and reading pages. Do not use the built-in `WebSearch` or `WebFetch` tools.

## Steps

1. Turn the question into one focused search query. Add the year if the topic is time-sensitive.
2. Call the search tool from the `tavily` server with that query.
   - For news, set the topic to news if the tool offers it.
   - Ask for about 5 results.
3. If the snippets are too thin to answer, call the `tavily` extract tool on the 1–2 most relevant URLs to read the full page.
4. If the first search misses, try one rephrased query. Do not search more than 3 times.
5. Reply in this format:

```
**Answer:** <2–4 sentence answer>

**Key points**
- <point> [1]
- <point> [2]

**Sources**
1. <title> — <url>
2. <title> — <url>
```

## Rules

- Only state facts that appear in the results. If results disagree, say so.
- Cite every key point with a source number.
- If the `tavily` tools are not available, do not fall back to `WebSearch`. Tell the user to enter their Tavily API key with `/plugin configure tavily-search@<marketplace>` and run `/mcp` to reconnect.