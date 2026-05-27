# AgeVolt Portal MCP Server

The production MCP server is implemented in `bff-service` under
`com.agevolt.bff.mcp`.

This plugin directory contains only the public-safe bootstrap needed by Codex:
the `.mcp.json` HTTP endpoint, skill instructions, and user-facing KB. Do not
copy secrets, JWTs, refresh tokens, private keys, or deployment credentials into
this marketplace.

## Endpoints

- MCP: `https://api1.my.agevolt.com/mcp/agevolt/mcp`
- OAuth issuer: `https://api1.my.agevolt.com/mcp/auth`
- Protected resource metadata:
  `https://api1.my.agevolt.com/mcp/agevolt/.well-known/oauth-protected-resource`

## Source

- Java package: `bff-service/src/main/java/com/agevolt/bff/mcp`
- Tools: `agevolt_charging_transactions_list`, `agevolt_current_context`,
  `agevolt_spaces_list`, `agevolt_space_switch`, `agevolt_logout`
