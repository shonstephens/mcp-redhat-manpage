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
- Podman (or Docker) with network access to `registry.access.redhat.com`

## Setup

Man pages must be extracted locally before the server can serve them. The extraction script pulls UBI container images, installs target packages, renders all man pages to plain text, and stores them under `manpages/rhel{8,9,10}/`.

```bash
npm install

# Extract all versions
npm run extract

# Or a specific version
bash scripts/extract.sh 9
```

The extraction script installs packages from these categories: SSSD/identity (`sssd-common`, `sssd-ad`, `sssd-ldap`, `sssd-krb5`, `sssd-tools`, `sssd-client`), Kerberos (`krb5-workstation`, `krb5-libs`), AD integration (`adcli`, `realmd`), auth stack (`authselect`), crypto (`crypto-policies`, `crypto-policies-scripts`), system services (`chrony`, `systemd`), network (`NetworkManager`, `bind-utils`), core (`man-db`, `man-pages`, `coreutils`, `shadow-utils`), and PAM (`pam`). Edit the `PACKAGES` array in `scripts/extract.sh` to add more.

Extraction uses the `:latest` UBI image tag for each major version, which tracks the latest minor release. Man pages are indexed by major version only. UBI repos are a subset of full RHEL — unavailable packages are skipped with a warning. Re-run extraction periodically to pick up updates.

## Configuration

No authentication required. The server reads pre-extracted man pages from the local filesystem.

### Gemini CLI

Add to `~/.gemini/settings.json`:

```json
{
  "mcpServers": {
    "redhat-manpage": {
      "command": "npx",
      "args": ["-y", "mcp-redhat-manpage"],
      "env": {
        "MANPAGES_DIR": "/path/to/manpages"
      }
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

Set `MANPAGES_DIR` to the directory containing your extracted man pages.

### Claude Code

Add to `~/.claude/settings.json`:

```json
{
  "mcpServers": {
    "redhat-manpage": {
      "command": "npx",
      "args": ["-y", "mcp-redhat-manpage"],
      "env": {
        "MANPAGES_DIR": "/path/to/manpages"
      }
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
      "args": ["-y", "mcp-redhat-manpage"],
      "env": {
        "MANPAGES_DIR": "/path/to/manpages"
      }
    }
  }
}
```

If `MANPAGES_DIR` is not set, the server looks for a `manpages/` directory relative to the package installation.

## Related MCP Servers

- [mcp-redhat-knowledge](https://github.com/shonstephens/mcp-redhat-knowledge) - Knowledge Base search
- [mcp-redhat-support](https://github.com/shonstephens/mcp-redhat-support) - Support case management
- [mcp-redhat-account](https://github.com/shonstephens/mcp-redhat-account) - Account management
- [mcp-redhat-subscription](https://github.com/shonstephens/mcp-redhat-subscription) - Subscription management

## License

MIT
