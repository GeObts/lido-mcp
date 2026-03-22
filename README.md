# Lido MCP Server

**Model Context Protocol (MCP) server for Lido Finance operations**

Built by AOX (@AOXexchange, aox.llc) for The Synthesis Ethereum Agent Hackathon 2026.

---

## Overview

This MCP server gives AI agents native access to Lido staking operations. Point Claude (or any MCP-compatible client) at this server and stake ETH from a conversation — zero custom code.

**Transport:** stdio (MCP standard — not an HTTP server)
**Wallet:** `0x7e7f825248Ae530610F34a5deB9Bc423f6d63373`

---

## Available Tools

| Tool | Description | Network |
|------|-------------|---------|
| `lido_stake` | Stake ETH for stETH | Ethereum Mainnet |
| `lido_unstake` | Request unstake (withdrawal queue) | Ethereum Mainnet |
| `lido_wrap` | Wrap stETH to wstETH | Base |
| `lido_unwrap` | Unwrap wstETH to stETH | Base |
| `lido_balance` | Check stETH/wstETH balances + live APY | Mainnet + Base |
| `lido_rewards` | Get staking rewards data + projections | Mainnet + Base |

All write operations support `dry_run: true` for safe testing.

---

## Installation

```bash
git clone https://github.com/GeObts/lido-mcp.git
cd lido-mcp
npm install
```

### Environment Setup

```bash
cp .env.example .env
# Edit .env and set AOX_BANKER_PRIVATE_KEY (never commit real keys)
```

The server loads the private key from (first match wins):
1. `AOX_BANKER_PRIVATE_KEY` environment variable (set by MCP client config)
2. `.env` file in the project root
3. `~/.openclaw/.env` (legacy AOX agent path)

---

## Usage with MCP Clients

### Claude Desktop

Add to your Claude Desktop config (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "lido": {
      "command": "node",
      "args": ["/path/to/lido-mcp/index.js"],
      "env": {
        "AOX_BANKER_PRIVATE_KEY": "0xYourPrivateKeyHere"
      }
    }
  }
}
```

Restart Claude Desktop. The Lido tools appear automatically.

### Claude Code

```bash
claude mcp add lido node /path/to/lido-mcp/index.js
```

### Example Conversation

**User**: "Stake 0.01 ETH via Lido"

**Claude**: *Calls `lido_stake` with `dry_run: true` first*

> Simulation successful:
> - Stake: 0.01 ETH → ~0.01 stETH
> - Gas: ~0.015 ETH
> - Current APY: 3.2% (live from Lido API)
>
> Proceed with the real transaction?

**User**: "Yes"

**Claude**: *Calls `lido_stake` with `dry_run: false`*

> Transaction confirmed:
> - Hash: `0x...`
> - stETH received: 0.00998
> - Now earning staking rewards

---

## Live Proof — Dry Run Results

Real simulations executed on mainnet:

### lido_stake (dry_run)
```json
{
  "success": true,
  "dry_run": true,
  "operation": "stake",
  "amount": "0.01",
  "contract": "0xae7ab96520DE3A18E5e111B5EaAb095312D7fE84",
  "network": "ethereum-mainnet",
  "estimated_stETH": "0.01",
  "current_apy_percent": 3.2,
  "gas_estimate": "~0.015 ETH"
}
```

### lido_balance (live query)
```json
{
  "success": true,
  "address": "0x7e7f825248Ae530610F34a5deB9Bc423f6d63373",
  "total_stETH_equivalent": "0.0",
  "current_apy_percent": 3.2,
  "apy_source": "Lido API (live)"
}
```

---

## Security

**This server handles real funds.**

- Private keys stored in `.env` (gitignored) — never committed
- Rate limiting: 10 calls/minute per tool
- Input validation: 0.001–10 ETH per stake transaction
- Dynamic gas pricing with 20% buffer
- `dry_run` mode on all write operations
- Live APY from Lido API (cached 5 min, falls back gracefully)

---

## AOX Integration

This MCP server powers the **Banker Agent** in the AOX autonomous pipeline:

1. **Marketplace Agent** sells leads → receives USDC
2. **Banker Agent** swaps 20% of USDC → ETH
3. **This MCP server** stakes ETH → stETH via `lido_stake`
4. Treasury earns 3-4% APY on idle funds

Fully autonomous. Zero human intervention.

---

## Hackathon

**Event:** The Synthesis — Ethereum Agent Hackathon 2026
**Team:** AOX (Agent Opportunity Exchange)
**Prize Track:** Lido MCP ($5,000)
**ERC-8004:** Both Marketplace and Banker agents registered on-chain
**Repo:** https://github.com/GeObts/lido-mcp

---

## License

MIT — See [LICENSE](./LICENSE)

---

**Built by AOX** · aox.llc · @AOXexchange
