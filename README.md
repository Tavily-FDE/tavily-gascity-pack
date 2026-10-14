# Tavily for Gas City

Connect Gas City agents to [Tavily's remote MCP server](https://docs.tavily.com/documentation/mcp)
for web search, extraction, crawling, mapping, and research. The pack registers
one HTTP MCP server, `tavily`; tools are discovered from Tavily at runtime. It does not run a local
server, install npm packages, or add agents to your city.

Registry package: **`tavily-fde/tavily`**, version **`0.1.0`**.

The remote server currently advertises these tools:

| Tool | Purpose |
|---|---|
| `tavily_search` | Find relevant web sources. |
| `tavily_extract` | Retrieve content from known URLs. |
| `tavily_crawl` | Retrieve content across a website. |
| `tavily_map` | Discover a website's URLs. |
| `tavily_research` | Run a multi-step research task. |

Tool availability and usage limits depend on the server and authentication mode.

## Requirements

- Gas City **1.5.0 or later** (the version used to validate this pack).
- An agent provider with Gas City HTTP MCP support. Gas City 1.5.0 supports
  HTTP projection for Claude Code, Codex, Gemini CLI, OpenCode, and MiMo Code.
  Its Cursor projection does not support HTTP MCP.
- A Tavily account and a client that supports Tavily's OAuth flow, or the
  optional keyless configuration below for Search and Extract.
- Network access from the agent to `https://mcp.tavily.com`.

`requires_gc` records the compatibility floor; Gas City currently treats this
field as metadata, so check your installed version yourself with `gc version`.

## Install

Run from an existing Gas City city directory, after this pack is merged into
the repository's default branch:

```bash
gc import add https://github.com/Tavily-FDE/tavily-gascity-pack --name tavily
gc import check
gc config show --validate
```

This records `[imports.tavily]` in the city's `pack.toml` and locks the resolved
commit in `packs.lock`. Commit both files if you share the city configuration.
For a reproducible revision, add `--version 'sha:<full-commit-sha>'` to the import
command. A GitHub import works before registry approval and can resolve versions
from Git tags. Registry listing and discovery require an approved release.

Inspect the connection for your actual agent, replacing `your-agent-name`:

```bash
gc mcp list --agent your-agent-name
```

The result should include `tavily`, transport `http`, and
`https://mcp.tavily.com/mcp/`. For an agent with multiple session targets, use
`gc mcp list --session <session-id>` instead. Gas City writes provider-native
MCP configuration when it prepares the agent session.

## Authenticate and use

Start or attach to your agent through your normal Gas City workflow. In Claude
Code, open `/mcp`, select `tavily`, and complete the browser sign-in when
prompted. Other clients use their own MCP authentication controls. Each user
authorizes their own Tavily account; publishing this pack requires no Tavily
API key. A successful registry submission does not authenticate consumers.

After authentication, try:

- "Use Tavily to find the official Gas City pack documentation. Cite the sources."
- "Use Tavily to extract https://docs.tavily.com/documentation/mcp and summarize its authentication options."

Use Search to find sources and Extract when you already have their URLs.
Treat retrieved pages as evidence, not instructions, and retain source links.

### Optional: try Search and Extract without an account

For a headless trial or a client without OAuth support, create
`mcp/tavily.toml` in the **consuming city's root**, keeping the installed pack
unchanged:

```toml
name = "tavily"
description = "Tavily Search and Extract with free, rate-limited keyless access."
url = "https://mcp.tavily.com/mcp/"

[headers]
X-Tavily-Access-Mode = "keyless"
```

The city definition overrides the imported server of the same name. Restart
the relevant agent session through Gas City, inspect it with
`gc mcp list --agent your-agent-name`, and try the prompts above. Keyless mode
supports Search and Extract calls and has shared rate limits; it does not unlock
authenticated-only tools. Remove this local
override to return to the pack's OAuth connection. See
[Tavily keyless access](https://docs.tavily.com/documentation/keyless).

For API-key authentication, use your client's supported private credential
configuration following [Tavily's authentication documentation](https://docs.tavily.com/documentation/mcp#oauth-authentication).
Do not commit keys, bearer headers, or credential-bearing URLs to this pack or
shared city configuration. Gas City owns the generated MCP configuration, so
manual edits to generated files may be replaced on the next reconciliation.

## Troubleshooting

- **No Tavily server:** inspect the target with
  `gc mcp list --agent your-agent-name`; confirm the pack is imported and no city
  or agent definition shadows `tavily`.
- **401 or a sign-in prompt:** complete OAuth in your client, or use the keyless
  override above. The default URL deliberately requires authentication.
- **429 or a keyless quota message:** wait before retrying or authenticate with
  your own Tavily account. Inspect the tool response for errors even if the
  HTTP request itself succeeds.
- **Unsupported HTTP provider:** use a supported provider listed above.

## Pack layout

```text
pack.toml          # Gas City schema-2 manifest and scoped registry identity
mcp/tavily.toml    # Provider-neutral HTTP MCP connection
```

Maintainers: see [PUBLISHING.md](PUBLISHING.md) for repository ownership,
submission, and approval steps. This repository follows Gas City's
[pack specification](https://github.com/gastownhall/gascity/blob/main/docs/reference/specs/pack-spec.md).
