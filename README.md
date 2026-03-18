# Lido MCP Server

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A Model Context Protocol (MCP) server that makes Lido staking natively callable by any AI agent. Built for [The Synthesis Hackathon 2026](https://ethglobal.com/events/synthesis) — Lido MCP Prize submission.

**The goal**: A developer points Claude (or any MCP-compatible AI) at this server and stakes ETH from a conversation with zero custom code.

---

## 🚀 Quick Start

### Installation

```bash
# Clone the repository
git clone https://github.com/GeObts/lido-mcp.git
cd lido-mcp

# Install dependencies
npm install

# Configure environment
cp .env.example .env
# Edit .env and add your private key

# Start the server
npm start
```

### Connect to Claude

Add to your Claude Desktop config (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```json
{
  "mcpServers": {
    "lido": {
      "command": "node",
      "args": ["/path/to/lido-mcp/index.js"],
      "env": {
        "AOX_BANKER_PRIVATE_KEY": "0x..."
      }
    }
  }
}
```

Restart Claude. The Lido tools will appear in your conversation.

---

## 🛠️ Available Tools

| Tool | Description | Chain |
|------|-------------|-------|
| `lido_stake` | Stake ETH, receive stETH | Ethereum Mainnet |
| `lido_unstake` | Request withdrawal of stETH → ETH | Ethereum Mainnet |
| `lido_wrap` | Convert stETH → wstETH (non-rebasing) | Base |
| `lido_unwrap` | Convert wstETH → stETH | Base |
| `lido_balance` | Check stETH/wstETH balances across chains | Multi-chain |
| `lido_rewards` | Get current APY and reward estimates | Multi-chain |
| `lido_governance_vote` | Vote on Lido DAO proposals | Ethereum Mainnet |

**All write operations support `dry_run: true` for safe simulation.**

---

## 💬 Demo Conversation

**User**: "Stake 0.1 ETH"

**Claude**: I'll stake 0.1 ETH via Lido for you. Let me simulate first to confirm the details.

*[Calls `lido_stake` with `dry_run: true`]*

**Claude**: Simulation successful. This will:
- Stake 0.1 ETH
- Receive ~0.1 stETH
- Gas cost: ~0.015 ETH
- Current APY: 3.2%

Shall I proceed?

**User**: "Yes"

*[Calls `lido_stake`]*

**Claude**: Done! Transaction confirmed:
- Hash: `0xabc123...`
- Received: 0.0998 stETH
- Your stETH balance is now auto-compounding at 3.2% APY

---

## 🏦 How AOX Uses This

The **Banker Agent** (wallet: `0x7e7f825248Ae530610F34a5deB9Bc423f6d63373`) runs an autonomous Lido investment strategy:

### Daily Operations
1. Receives USDC from Marketplace Agent sales
2. Swaps 20% to ETH via Uniswap
3. Stakes ETH → stETH via this MCP server
4. Accumulates stETH for yield

### Weekly Rebalancing
1. Monitors Base L2 gas costs
2. Bridges excess stETH to Base
3. Wraps to wstETH for gas-efficient DeFi
4. Logs all transactions to treasury ledger

**Result**: Treasury earns 3-4% APY on idle funds, fully autonomously.

---

## 📁 Project Structure

```
lido-mcp/
├── index.js           # MCP server implementation
├── package.json       # Dependencies
├── lido.skill.md      # AI agent mental model for Lido
└── README.md          # This file
```

---

## 🔧 Configuration

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `AOX_BANKER_PRIVATE_KEY` | Yes | Private key for transaction signing |

The server reads from `~/.openclaw/.env` automatically or uses environment variables.

### RPC Endpoints

- **Ethereum Mainnet**: `https://eth.llamarpc.com`
- **Base**: `https://mainnet.base.org`

Override by setting `ETH_RPC` and `BASE_RPC` environment variables.

---

## 🔐 Security

- Private keys never logged or exposed
- All write operations require explicit confirmation (use `dry_run`)
- Gas limits set conservatively
- Contract addresses hardcoded (no injection attacks)

---

## 📊 Contract Addresses

### Ethereum Mainnet
- **Lido stETH**: `0xae7ab96520DE3A18E5e111B5EaAb095312D7fE84`
- **Withdrawal Queue**: `0x889edC2eDab5f40e902b864aD4d7AdE8E412F9B1`

### Base
- **wstETH**: `0xc1CBa3fCea344f92D9239c08C0568f6F2F0ee452`
- **stETH**: `0xE3F4b4891bC8830e1D8228C85D1fBc05d762B77`

---

## 🧪 Testing

```bash
# Test dry runs (no funds needed)
npm start
# In another terminal:
curl -X POST http://localhost:3300/api/tool \
  -H "Content-Type: application/json" \
  -d '{
    "tool": "lido_stake",
    "args": {"amount": "0.01", "dry_run": true}
  }'
```

---

## 🤝 Integration Examples

### For Claude Desktop
See [Quick Start](#quick-start) above.

### For Custom Agents

```javascript
import { Client } from '@modelcontextprotocol/sdk/client/index.js';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

const transport = new StdioClientTransport({
  command: 'node',
  args: ['lido-mcp/index.js']
});

const client = new Client({ name: 'my-agent', version: '1.0.0' });
await client.connect(transport);

// Stake ETH
const result = await client.callTool('lido_stake', {
  amount: '0.5',
  dry_run: true
});
```

---

## 🏆 Hackathon: The Synthesis 2026

**Prize**: Lido MCP Integration ($5,000)

**Judging Criteria**:
- ✅ AI-native experience (zero custom code for users)
- ✅ Production-ready code
- ✅ Cross-chain support (Mainnet + Base)
- ✅ Complete feature set (stake, unstake, wrap, balance, rewards)

**Team**: AOX (Agent Opportunity Exchange)  
**Built by**: Goyabean  
**Contact**: `@pupAOXbot` on Telegram

---

## 📜 License

MIT License — see [LICENSE](LICENSE) for details.

---

## 🙏 Acknowledgments

- [Model Context Protocol](https://modelcontextprotocol.io/) — Anthropic's open standard
- [Lido Finance](https://lido.fi/) — Liquid staking infrastructure
- [Ethers.js](https://ethers.org/) — Ethereum library
- [AOX](https://aox.llc) — Autonomous agent infrastructure

---

## 📞 Support

Issues? Questions?
- Open an issue on GitHub
- Contact via Telegram: `@pupAOXbot`

**Staking should be as simple as a conversation. Now it is.**
