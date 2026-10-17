# Build the tavily-search plugin from scratch

This guide shows how to build the `tavily-search` Claude Code plugin yourself, one file at a time, and how to publish it through a local plugin marketplace.

When you finish, Claude Code will have:

| Part | Name | What it does |
|---|---|---|
| Connector (remote MCP server) | `tavily` | Connects to Tavily's hosted MCP server at `https://mcp.tavily.com/mcp/` and provides tools such as `tavily_search` and `tavily_extract` |
| Skill | `web-research` | Tells Claude how to use those tools: search, read pages when needed, and answer with numbered sources |
| User setting | `tavily_api_key` | Claude Code asks for your Tavily API key when you install the plugin and stores it securely |

---

## 1. Prerequisites

| Requirement | Why | Check |
|---|---|---|
| Claude Code (a recent version) | Runs the plugin. This guide was tested with v2.1.302 | `claude --version` |
| A Tavily API key | Tavily's server needs it for every request | Create a free one at <https://app.tavily.com>. Keys start with `tvly-` |

---

## 2. How the pieces fit together

A **plugin** is a folder with a manifest (`.claude-plugin/plugin.json`) and optional parts such as skills and MCP servers.

A **marketplace** is a folder (or git repo) with a `.claude-plugin/marketplace.json` file that lists one or more plugins. Claude Code installs plugins from marketplaces.

The API key is handled with **`userConfig`**, a section of `plugin.json` that declares values the user must provide:

```
plugin.json declares tavily_api_key
        │
        ▼
/plugin install shows a dialog ──► you type the key ──► stored in your system's secure credential store
                                                         (Keychain on macOS)
        │
        ▼
.mcp.json uses ${user_config.tavily_api_key} ──► "Authorization: Bearer <key>" ──► https://mcp.tavily.com/mcp/
```

The key is never written in the plugin's files, in `settings.json` or in a `.env` file, so it can't be committed to git by mistake.

---

## 3. Create the folders

Pick a folder for your marketplace. This guide uses `my-marketplace`.

```bash
mkdir -p my-marketplace/.claude-plugin
mkdir -p my-marketplace/plugins/tavily-search/.claude-plugin
mkdir -p my-marketplace/plugins/tavily-search/skills/web-research
cd my-marketplace
```

At the end you will have this layout:

```
my-marketplace/
├── .claude-plugin/
│   └── marketplace.json                 # lists the plugins in this marketplace (step 7)
└── plugins/
    └── tavily-search/
        ├── .claude-plugin/plugin.json   # plugin manifest and the API key setting (step 4)
        ├── .mcp.json                    # the tavily connector (step 5)
        └── skills/
            └── web-research/SKILL.md    # instructions for Claude (step 6)
```

All paths in the next steps are relative to `my-marketplace/`.

---

## 4. Write the plugin manifest

Create `plugins/tavily-search/.claude-plugin/plugin.json`:

```json
{
  "name": "tavily-search",
  "version": "1.0.0",
  "description": "Web search and page extraction through the remote Tavily MCP server, plus a web-research skill that uses it.",
  "author": {
    "name": "Your Name"
  },
  "keywords": ["search", "web", "tavily", "mcp", "skills"],
  "userConfig": {
    "tavily_api_key": {
      "type": "string",
      "title": "Tavily API key",
      "description": "Your key from https://app.tavily.com (starts with tvly-)",
      "sensitive": true,
      "required": true
    }
  }
}
```

The metadata fields:

| Field | Required | Meaning |
|---|---|---|
| `name` | Yes | The plugin's ID. Use lowercase letters, digits and hyphens. Skills are named after it, for example `/tavily-search:web-research` |
| `version` | No | Increase it when you change the plugin, so installed copies get updated |
| `description`, `author`, `keywords` | No | Shown in the `/plugin` menu |

The `userConfig` section declares one setting, `tavily_api_key`. Setting names may contain only letters, digits and underscores.

| Field | Meaning |
|---|---|
| `type` | The kind of value. Here a `string`. Other types are `number`, `boolean`, `directory` and `file` |
| `title` | The label shown in the dialog |
| `description` | Help text shown under the field |
| `sensitive: true` | Hides the input as you type and stores the value in the system's secure credential store instead of `settings.json`. Use it for every key, token or password |
| `required: true` | The dialog won't accept an empty value |

Claude Code rejects unknown fields inside a `userConfig` entry, and the plugin won't load. Use only the fields above (plus `default`, `options`, `multiple`, `min` and `max`, which this plugin doesn't need).

Only `plugin.json` goes inside `.claude-plugin/`. Skills and `.mcp.json` go in the plugin's root folder.

---

## 5. Add the MCP server

Create `plugins/tavily-search/.mcp.json`:

```json
{
  "mcpServers": {
    "tavily": {
      "type": "http",
      "url": "https://mcp.tavily.com/mcp/",
      "headers": {
        "Authorization": "Bearer ${user_config.tavily_api_key}"
      }
    }
  }
}
```

| Field | Meaning |
|---|---|
| `"tavily"` | The server's name. In `/mcp` it is shown as `plugin:tavily-search:tavily` |
| `type: "http"` | A remote server that uses the streamable HTTP transport |
| `url` | Tavily's hosted MCP endpoint |
| `headers` | HTTP headers sent with every request. Here, the API key as a Bearer token |
| `${user_config.tavily_api_key}` | Replaced with the value saved for the `tavily_api_key` setting from step 4 |

**Put the key in `headers`, not `headersHelper`.** `headersHelper` runs a script through a shell, and Claude Code refuses to put `${user_config.*}` values into it. The server is then reported as misconfigured. The `headers` field isn't run through a shell, so it's the supported place for the key.

---

## 6. Write the skill

A skill is a folder with a `SKILL.md` file. The `description` in the frontmatter tells Claude when to use the skill. The body tells Claude how to do the task.

Create `plugins/tavily-search/skills/web-research/SKILL.md`:

````markdown
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
````

Tips for writing a skill:
- Make the `description` specific. Claude reads only the name and description to decide whether to load the skill.
- Keep the steps short and in order. Set limits, such as "no more than 3 searches", so the skill doesn't run too long.
- Say what to do when something fails, such as the tools not being available.

---

## 7. Create the marketplace

Create `.claude-plugin/marketplace.json` in the marketplace's root folder (not inside the plugin):

```json
{
  "name": "my-local-marketplace",
  "owner": {
    "name": "Your Name"
  },
  "metadata": {
    "description": "Local marketplace for learning Claude Code plugins"
  },
  "plugins": [
    {
      "name": "tavily-search",
      "source": "./plugins/tavily-search",
      "description": "Web search and page extraction through the remote Tavily MCP server, plus a web-research skill.",
      "version": "1.0.0"
    }
  ]
}
```

| Field | Required | Meaning |
|---|---|---|
| `name` | Yes | The marketplace's ID. You use it when installing: `tavily-search@my-local-marketplace` |
| `owner.name` | Yes | Who maintains the marketplace |
| `metadata.description` | No | Shown when browsing marketplaces |
| `plugins` | Yes | The list of plugins this marketplace offers |
| `plugins[].name` | Yes | Must match `name` in the plugin's `plugin.json` |
| `plugins[].source` | Yes | Where the plugin is. A path starting with `./` is relative to the marketplace's root folder |
| `plugins[].description`, `plugins[].version` | No | Shown in the `/plugin` menu. If both files set `version`, the one in `plugin.json` wins, so keep them the same |

**Adding to an existing marketplace.** If you already have a `marketplace.json`, don't create a new one. Add the `tavily-search` object to its `plugins` array.

**Sharing it.** Push the `my-marketplace` folder to a git repo. Other people can then add it with `claude plugin marketplace add <github-user>/<repo>` or with the repo's URL. Each person enters their own API key when they install.

Check both files for mistakes:

```bash
claude plugin validate ./plugins/tavily-search
claude plugin validate .
```

Both should print `✔ Validation passed`. The validator also reports an error if `.mcp.json` uses a `${user_config.KEY}` that `plugin.json` doesn't declare, which catches spelling mistakes in the setting name.

---

## 8. Install it and enter your key

Start Claude Code in the `my-marketplace` folder and run:

```
/plugin marketplace add ./
/plugin install tavily-search@my-local-marketplace
```

A dialog opens and asks for the **Tavily API key**. Paste your key. The input is hidden, and the key is saved to your system's secure credential store.

Then run `/reload-plugins` (or restart Claude Code).

**Installing from the shell instead.** `claude plugin install` doesn't show the dialog. Install first, then open the dialog inside Claude Code:

```bash
claude plugin marketplace add ./
claude plugin install tavily-search@my-local-marketplace
claude      # then run: /plugin configure tavily-search@my-local-marketplace
```

You can also pass the key on the command line with `claude plugin install tavily-search@my-local-marketplace --config tavily_api_key=tvly-...`, but the key then stays in your shell history.

**Testing with `--plugin-dir`.** `claude --plugin-dir ./plugins/tavily-search` loads the plugin without installing it, which is handy while editing the skill. The key dialog belongs to the `/plugin` install flow, so install from the marketplace at least once to save your key.

---

## 9. Check that it works

| Check | You should see |
|---|---|
| `/mcp` | `plugin:tavily-search:tavily` **connected** |
| Type `/tavily-search:` | `web-research` in the autocomplete list |

---

## 10. Use it

Ask a question that needs current information. Claude chooses the skill because of its `description`:

```
What's the latest stable Python release?
```

Or run the skill directly:

```
/tavily-search:web-research summarize this week's news about the MCP spec
```

The answer follows the format from `SKILL.md`: a short answer, key points with source numbers, and a list of sources.

---

## 11. Change the key or the plugin later

**Change the key** (for example, after rotating it):

```
/plugin configure tavily-search@my-local-marketplace
```

Then run `/mcp` to reconnect. Sensitive settings aren't shown in the `/config` panel, so use `/plugin configure`.

**Change the plugin:**

1. Edit the plugin's files.
2. Increase `version` in both `plugin.json` and `marketplace.json`, for example to `1.0.1`.
3. Update the installed copy:
   ```bash
   claude plugin marketplace update my-local-marketplace
   claude plugin update tavily-search@my-local-marketplace
   ```
4. Run `/reload-plugins` in Claude Code, or restart it.

If you add a new `userConfig` setting in an update, run `/plugin configure` to fill it in.

---

## 12. Troubleshooting

| Symptom | Fix |
|---|---|
| `/mcp` shows `tavily` as **failed**, or the server needs authentication | The key is missing or wrong. Run `/plugin configure tavily-search@my-local-marketplace`, enter the key, then run `/mcp` |
| The key dialog never appeared | You installed with the `claude plugin install` shell command, which never prompts. Run `/plugin configure tavily-search@my-local-marketplace` inside Claude Code |
| Server returns 401 | Check the key at <https://app.tavily.com>, then enter it again with `/plugin configure`. If the key is valid and it still fails, Tavily may not accept the key as a Bearer header. Its documented method is the URL: in `.mcp.json`, remove `headers` and set `"url": "https://mcp.tavily.com/mcp/?tavilyApiKey=${user_config.tavily_api_key}"` |
| The plugin fails to load after editing `userConfig` | An unknown field inside a `userConfig` entry stops the plugin from loading. Run `claude plugin validate ./plugins/tavily-search` |
| Validation error about `${user_config...}` | The name in `.mcp.json` doesn't match the name in `plugin.json`. Both must be `tavily_api_key` |
| The server is reported as misconfigured | You used `${user_config.*}` in `headersHelper`. Move it to `headers` (step 5) |
| Plugin isn't listed after install | Check that the name in `marketplace.json` matches `plugin.json`, run `claude plugin marketplace update my-local-marketplace`, and restart Claude Code |
| The skill is never used | Make the skill's `description` more specific, or call it directly with `/tavily-search:web-research` |

---

## Reference

- [Plugin manifest reference: user configuration](https://code.claude.com/docs/en/plugins-reference#user-configuration)
- [Add components to a plugin: ask the user for configuration values](https://code.claude.com/docs/en/plugins/components)
- [MCP in Claude Code](https://code.claude.com/docs/en/mcp)
