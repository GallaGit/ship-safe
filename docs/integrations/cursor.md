# Cursor integration

Ship Safe can be used by Cursor through Cursor's MCP client configuration. This gives Cursor's agent access to Ship Safe's security tools while it works in a repository.

## Supported surfaces

| Surface | Status |
| --- | --- |
| Cursor IDE (desktop app, local `stdio` MCP server) | Documented below. |
| Cursor CLI | Not verified. Cursor documents shared MCP configuration, but this setup has not been tested with the CLI. |
| Cursor cloud / background agents | Not verified. A local `stdio` server on your machine is not available to cloud agents. |

Do not treat the unverified surfaces as supported until they appear in [Tested versions](#tested-versions).

## Configure the MCP server

Cursor reads MCP servers from two locations:

- **Project:** `.cursor/mcp.json` in the repository root. Applies only to that project.
- **User:** `~/.cursor/mcp.json` in your home directory. Applies to every project you open.

Prefer the project configuration. Ship Safe's MCP server only accepts paths inside its working directory (the MCP workspace), so it should start from the repository you want to scan. Pin the package version so the server does not change between sessions.

### Project configuration (recommended)

`.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "ship-safe": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "ship-safe@11.0.0", "mcp"]
    }
  }
}
```

Replace `11.0.0` with the Ship Safe version you have reviewed. Pass paths inside the project to tools (for example `"."` or `"src"`); paths outside the server's working directory are rejected.

### User configuration

`~/.cursor/mcp.json` uses the same format. Because one user-level server is shared across projects, its working directory may not match the project you have open. If tools return `Path must stay within the MCP workspace`, use the project configuration instead.

Cursor's `mcp.json` has no documented `cwd` field. Which directory Cursor uses as the server's working directory is not verified in this guide; confirm it with `ship_safe_status` and a small scan before relying on it.

### Credentials

Ship Safe's MCP server does not need API keys to run. Do not add `env` or `envFile` entries with credentials to `.cursor/mcp.json`. That file is often committed and is part of the repository's trust boundary. See [Handle MCP environment variables safely](../mcp-env-safety.md).

## Verify the connection

1. Save the configuration and reload Cursor (or open the project in a new window).
2. Open Cursor Settings > MCP (Customize in newer versions) and check that `ship-safe` is enabled and lists its tools.
3. In Agent chat, ask: `Call the ship-safe ship_safe_status tool.`
4. Approve the call. The response should report `state: available` and `protected: false`.

`ship_safe_status` takes no arguments, reads no files, and changes nothing, so it is a safe first call. After that, try a small scoped check such as `get_checklist` or `scan_secrets` with `path: "."`.

Available tools include `scan_repo`, `scan_secrets`, `analyze_file`, `get_findings`, `get_checklist`, `suppress_finding`, and `ship_safe_status`. Note that `suppress_finding` edits files by adding `ship-safe-ignore` comments.

## Tool approvals

Cursor asks for approval before running MCP tools by default. Keep that default. Review each tool's arguments before approving, especially `path` values and any `suppress_finding` call.

Do not add a blanket allowlist or auto-approve every Ship Safe tool. If you allowlist anything, limit it to read-only tools such as `ship_safe_status` and `get_checklist`.

## What it does not claim

MCP access is not continuous enforcement. Ship Safe only runs when the agent chooses to call one of its tools. It does not intercept Cursor's file edits, terminal commands, or other tool calls, and it does not block anything on its own.

Describe this setup as **Ship Safe available in Cursor**, not as `Protected by Ship Safe`. `ship_safe_status` reports `protected: false` for that reason. Lifecycle enforcement exists today only for [Claude Code hooks](./claude-code.md).

## Troubleshooting

- **Server does not appear or shows an error:** open the Output panel, select **MCP Logs**, and check for startup errors.
- **`npx` not found:** Cursor must find `npx` on its `PATH`. Install Node.js, restart Cursor, or use the full path to `npx` in `command`.
- **First start is slow or times out:** `npx -y` downloads the package on first use. Run `npx -y ship-safe@11.0.0 --version` in a terminal once, then reload Cursor.
- **`Path must stay within the MCP workspace`:** the requested path is outside the server's working directory. Use the project configuration and pass a path inside the project.
- **Tools missing after a change:** toggle the server off and on, or reload the window.

## Remove

1. Delete the `ship-safe` entry from `.cursor/mcp.json` and/or `~/.cursor/mcp.json`. Delete the file only if it contains no other servers.
2. Reload Cursor and confirm `ship-safe` no longer appears in the MCP list.

To pause it without removing it, toggle the server off in Cursor's MCP settings.

## Tested versions

TODO: record Cursor / Node / Ship Safe versions after a real smoke test.

| Cursor surface | Cursor version | Node.js version | Ship Safe version | OS | Result |
| --- | --- | --- | --- | --- | --- |
| IDE | TODO | TODO | TODO | TODO | TODO |
