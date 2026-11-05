# Hyperdrive Skills

[Agent Skills](https://agentskills.io) for working with [Hyperdrive](https://docs.hyperdrive.finance) markets. They work with Claude Code, Codex, Cursor, ChatGPT and any other agent that supports the `SKILL.md` format.

| Skill | What it does |
| --- | --- |
| [`hyperdrive-markets`](skills/hyperdrive-markets/SKILL.md) | Read market data and account positions (debt, collateral, health score), and lend and borrow against Hyperdrive markets. |

## Install

### Claude Code, Codex, Cursor and other coding agents

Use the [`skills` CLI](https://github.com/vercel-labs/skills) (Node.js 22.20 or later):

```bash
npx skills add ambitlabsxyz/hyperdrive-skills
```

To install one skill for a specific agent:

```bash
npx skills add ambitlabsxyz/hyperdrive-skills --skill hyperdrive-markets -a claude-code
```

Swap `claude-code` for `codex`, `cursor`, or any other [supported agent](https://github.com/vercel-labs/skills/blob/main/docs/agents.md). Skills install into the current project by default. Add `-g` to install them for every project.

### ChatGPT

ChatGPT installs skills from a zip upload, not from GitHub.

1. Zip the skill folder so the zip contains a single top-level `hyperdrive-markets/` folder:

   ```bash
   cd skills && zip -r ../hyperdrive-markets.zip hyperdrive-markets
   ```

   On Windows (PowerShell):

   ```powershell
   Compress-Archive -Path skills/hyperdrive-markets -DestinationPath hyperdrive-markets.zip
   ```

2. In ChatGPT, open **Skills**, choose **Create** (or **New skill**), then **Upload from your computer**, and pick the zip.

On Business and Enterprise plans, a workspace admin may need to turn on skills and skill uploads first.

## Connect the Hyperdrive MCP server

The skill tells the agent how Hyperdrive markets work. The live data comes from the Hyperdrive MCP server, which you connect separately in each agent.

**Server URL:** `https://mcp.hyperdrive.finance` (Streamable HTTP, no sign-in needed)

| Tool | What it returns |
| --- | --- |
| `hyperdrive_get_supported_chains` | The chains Hyperdrive is deployed on (Base `8453`, HyperEVM `999`). |
| `hyperdrive_get_position` | An account's position in a market: debt, debt shares, collateral, collateral value, liquidation and borrow limits, and health score. |

### Claude Code

```bash
claude mcp add --transport http hyperdrive https://mcp.hyperdrive.finance
```

Add `--scope user` to make it available in every project.

### Claude Desktop and claude.ai

Go to **Settings → Connectors → Add custom connector** and enter the server URL.

### Codex

Add the server to `~/.codex/config.toml`:

```toml
[mcp_servers.hyperdrive]
url = "https://mcp.hyperdrive.finance"
```

### ChatGPT

1. Turn on **Developer mode** in **Settings → Apps & Connectors → Advanced**. Some versions put it under **Settings → Security**.
2. Create a connector, name it `Hyperdrive`, and enter the server URL.
3. In a new chat, turn on the connector from the tools menu.

ChatGPT only connects to remote HTTPS servers.

## Try it

Once the skill and the MCP server are connected, ask your agent something like:

> What's the debt for `0x…` on the deJAAA market?

The agent looks up the market address in the [deployments list](https://docs.hyperdrive.finance/developers/contracts/llms.txt), queries the position, and converts the raw amounts using each token's decimals.

## Repository layout

```
skills/
└── hyperdrive-markets/
    ├── SKILL.md              # Instructions the agent loads
    └── references/
        └── error-codes.md    # Market error codes
```

To add a skill, create `skills/<skill-name>/SKILL.md`. The `name` in its frontmatter must match the folder name.

## License

[MIT](LICENSE)
