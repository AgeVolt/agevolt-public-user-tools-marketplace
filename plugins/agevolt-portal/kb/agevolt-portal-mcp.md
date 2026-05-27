# AgeVolt Portal MCP

The AgeVolt Portal MCP server exposes user-facing AgeVolt tools through a
standard MCP OAuth flow.

## Public interface

- MCP server id: `agevolt-portal`
- MCP URL: `https://api1.my.agevolt.com/mcp/agevolt/mcp`
- OAuth issuer: `https://api1.my.agevolt.com/mcp/auth`
- Scope: `MCP.Access`

## Auth model

Codex uses MCP OAuth login. The browser is redirected to `my.agevolt.com`, where
the user signs in with the normal AgeVolt account. The BFF stores AgeVolt token
material server-side and returns only MCP tokens to Codex.

Never ask the user to paste JWTs or refresh tokens. Never read local token files
or browser storage from Codex.

## Tools

- `agevolt_charging_transactions_list`
- `agevolt_current_context`
- `agevolt_spaces_list`
- `agevolt_space_switch`
- `agevolt_logout`

Returns completed charging transactions visible to the signed-in user. Default
period is the rolling last 30 days.
