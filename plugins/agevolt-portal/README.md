# AgeVolt Portal

Public-safe Codex plugin for signed-in AgeVolt Portal user workflows.

The plugin uses the `agevolt-portal` MCP server from `bff-service`. OAuth login
redirects the user to `my.agevolt.com`; Codex receives only MCP tokens and never
AgeVolt JWT or refresh tokens.

## Tools

- `agevolt_charging_transactions_list`
- `agevolt_current_context`
- `agevolt_spaces_list`
- `agevolt_space_switch`
- `agevolt_logout`

## Endpoints

- MCP: `https://api1.my.agevolt.com/mcp/agevolt/mcp`
- OAuth issuer: `https://api1.my.agevolt.com/mcp/auth`
- Web portal consent: `https://my.agevolt.com/auth/mcp-consent`
- Scope: `MCP.Access`
