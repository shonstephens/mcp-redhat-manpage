# mcp-redhat-manpage

An [MCP](https://modelcontextprotocol.io/) server for RHEL man pages. Lets AI assistants look up configuration parameter defaults, syntax, and supported options from the actual man pages shipped with each RHEL major version.

## Tools

| Tool | Description |
|------|-------------|
| `getManPage` | Get the full content of a man page for a specific RHEL version |
| `searchManPages` | Search across all man pages for a keyword or regex pattern, with context lines |
| `compareVersions` | Compare a man page between two RHEL versions to detect parameter changes |
| `listManPages` | List available man pages, optionally filtered by name pattern |

## Prerequisites

- Node.js 18+

Man pages for RHEL 8, 9, and 10 are included via the [mcp-redhat-manpage-data](https://github.com/shonstephens/mcp-redhat-manpage-data) dependency. No container runtime or manual extraction required.

## Configuration

No authentication required. The server reads man pages bundled in the data package.

### Gemini CLI

Add to `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "redhat-manpage": {
      "command": "npx",
      "args": ["-y", "mcp-redhat-manpage"]
    }
  }
}
```

### watsonx Orchestrate

```bash
orchestrate toolkits import --kind mcp \
  --name redhat-manpage \
  --description "RHEL Man Pages" \
  --command "npx -y mcp-redhat-manpage" \
  --tools "*"
```

### Claude Code

Add to `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "redhat-manpage": {
      "command": "npx",
      "args": ["-y", "mcp-redhat-manpage"]
    }
  }
}
```

### VS Code / Cursor

Add to `.vscode/mcp.json` in your workspace:

```json
{
  "mcpServers": {
    "redhat-manpage": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "mcp-redhat-manpage"]
    }
  }
}
```

To override the bundled man pages with a custom directory, set the `MANPAGES_DIR` environment variable.

## Related MCP Servers

- [mcp-redhat-manpage-data](https://github.com/shonstephens/mcp-redhat-manpage-data) - Man page data (bundled as a dependency)
- [mcp-redhat-knowledge](https://github.com/shonstephens/mcp-redhat-knowledge) - Knowledge Base search
- [mcp-redhat-support](https://github.com/shonstephens/mcp-redhat-support) - Support case management
- [mcp-redhat-account](https://github.com/shonstephens/mcp-redhat-account) - Account management
- [mcp-redhat-subscription](https://github.com/shonstephens/mcp-redhat-subscription) - Subscription management

## License

MIT
