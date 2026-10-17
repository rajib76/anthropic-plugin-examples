# MCP Spec News — Week of Oct 11–17, 2026

**Answer:** I found no new MCP spec release or spec change published this week (Oct 11–17, 2026). This week's coverage is mostly about how the **2026-07-28** spec is being adopted. That release introduced a stateless core and features aimed at enterprises. Around it there was SDK news, vendor rollouts and commentary from conferences.

## Key points

- The 2026-07-28 spec made the protocol stateless. There is no `initialize` handshake and no `Mcp-Session-Id` header, and every request carries its own protocol version. Agent stacks that talk to MCP over HTTP are being upgraded to match. [3]
- A CIO piece (Oct 14) says the July 2026 update made MCP much easier to use in enterprises. It runs on ordinary infrastructure, IT can control which tools run at the gateway, and tools have a standard way to ask a human for confirmation before acting. The piece also says Anthropic donated MCP to the Linux Foundation's Agentic AI Foundation last December. [2]
- An October "state of MCP" report says TypeScript SDK v2.0.0 has been stable since 27 July 2026, split into `@modelcontextprotocol/server` and `/client` packages. Monthly downloads of the v1 `@modelcontextprotocol/sdk` grew 8.7× from a year earlier, to 273.3M. [4]
- On Oct 14, Google announced MCP support in Apigee hybrid 1.17.0, which runs the MCP data plane inside the customer's own Kubernetes cluster. [5]
- Commentary on AGNTCon+MCPCon (Oct 11) points out that some specs meant to connect agent infrastructure are still unshipped, including PAP v0.1 and the A2A Agent Registry. Vendor-specific extensions such as WebMCP are filling that gap. [1]

## Sources

1. [AGNTCon+MCPCon's 11 Tracks Map the Agent Infrastructure Control Planes (Forkast via CryptoRank)](https://cryptorank.io/news/feed/6cc05-agntconmcpcons-11-tracks-map-the-agent-infrastructure-control-planes-the-specs-that-would-connect-them-remain-unshipped)
2. [MCP is changing the questions marketers can afford to ask (CIO)](https://www.cio.com/article/4235515/mcp-is-changing-the-questions-marketers-can-afford-to-ask.html)
3. [Top 30 AI Agent Developer Interview Questions (Second Talent)](https://www.secondtalent.com/interview-guide/ai-agent-developers)
4. [State of MCP Server Development: October 2026 (LocalCan)](https://www.localcan.com/blog/state-of-mcp-server-development)
5. [Announcing MCP support in Apigee hybrid (Google Developers Blog)](https://developers.googleblog.com/announcing-mcp-support-in-apigee-hybrid-agentic-tools-that-run-in-your-own-cluster)

---
*Compiled 2026-10-17 from Tavily web search results.*
