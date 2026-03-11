# mcp-redhat-manpage

MCP server providing version-indexed RHEL man pages for configuration parameter verification.

## Why

Red Hat Knowledge Base articles and product documentation cover procedures and known issues, but they don't always document the exact default values, supported options, and syntax for every configuration directive. Man pages are the canonical source for that detail — and they vary between RHEL major versions.

This server extracts man pages from official Red Hat UBI container images, renders them to plain text, and serves them over MCP so that AI assistants can verify configuration claims against the actual documentation shipped with each RHEL release.

## Prerequisites

- **Podman** (or Docker) — used to pull UBI images and extract man pages
- **Node.js** 18+ — runs the MCP server
- **Network access** to `registry.access.redhat.com` — for pulling UBI images during extraction

## Setup

```bash
# Install dependencies
npm install

# Extract man pages from RHEL UBI containers
npm run extract            # All versions (8, 9, 10)
bash scripts/extract.sh 9  # Specific version only

# Verify extraction
ls manpages/rhel9/         # Should contain .txt files
```

Extraction pulls UBI images for each RHEL major version, installs target packages (SSSD, Kerberos, adcli, realmd, authselect, chrony, systemd, etc.), renders all man pages to plain text, and stores them locally under `manpages/rhel{8,9,10}/`.

### What gets extracted

The extraction script installs these package groups and captures all their man pages:

| Category | Packages |
|---|---|
| SSSD / Identity | sssd-common, sssd-ad, sssd-ldap, sssd-krb5, sssd-tools, sssd-client |
| Kerberos | krb5-workstation, krb5-libs |
| AD Integration | adcli, realmd |
| Auth Stack | authselect |
| Crypto | crypto-policies, crypto-policies-scripts |
| System Services | chrony, systemd |
| Network | NetworkManager, bind-utils |
| Core | man-db, man-pages, coreutils, shadow-utils |
| PAM | pam |

To add more packages, edit the `PACKAGES` array in `scripts/extract.sh` and re-run extraction.

### Container images

Extraction uses the `:latest` tag for each major version's UBI image, which tracks the latest minor release:

| Version | Image |
|---|---|
| RHEL 8 | `registry.access.redhat.com/ubi8/ubi:latest` |
| RHEL 9 | `registry.access.redhat.com/ubi9/ubi:latest` |
| RHEL 10 | `registry.access.redhat.com/ubi10/ubi:latest` |

Man pages are indexed by major version only (e.g., `rhel9`). Re-run extraction periodically to pick up minor release updates.

### UBI repository limitations

UBI images provide a subset of the full RHEL package repository. Some packages may not be available in UBI repos. The extraction script handles this gracefully — unavailable packages are skipped with a warning.

## MCP Configuration

Add to your Claude Code MCP settings (`~/.claude/settings.json` or project-level):

```json
{
  "mcpServers": {
    "redhat-manpage": {
      "command": "node",
      "args": ["/path/to/mcp-redhat-manpage/src/index.js"]
    }
  }
}
```

Or with a custom man pages directory:

```json
{
  "mcpServers": {
    "redhat-manpage": {
      "command": "node",
      "args": ["/path/to/mcp-redhat-manpage/src/index.js"],
      "env": {
        "MANPAGES_DIR": "/custom/path/to/manpages"
      }
    }
  }
}
```

## Tools

### getManPage

Get the full content of a man page for a specific RHEL version.

```
page: "sssd.conf"     # Man page name (required)
section: "5"           # Section — default "5" (config files), "8" (admin commands), "1" (user commands)
rhelVersion: "9"       # RHEL major version — default "9"
```

Returns the complete rendered man page text. If the page isn't found, suggests similar pages.

### searchManPages

Search across all man pages for a keyword or regex pattern.

```
query: "ldap_network_timeout"   # Search term or regex (required)
rhelVersion: "9"                # Default "9"
maxResults: 20                  # Default 20
contextLines: 3                 # Lines of context around each match — default 3
```

Returns matching lines with surrounding context, identifying which man page and line number each match comes from.

### compareVersions

Compare a man page between two RHEL versions to detect parameter changes.

```
page: "sssd-ad"        # Man page name (required)
section: "5"           # Default "5"
version1: "8"          # First version — default "8"
version2: "9"          # Second version — default "9"
```

Shows line counts, whether the pages are identical, and samples of lines added or removed between versions.

### listManPages

List all available man pages, optionally filtered by name pattern.

```
rhelVersion: "9"       # Default "9"
filter: "sssd"         # Optional name pattern (regex)
```

## Use cases

- **Verify configuration defaults**: "What is the default value of `ldap_network_timeout` in RHEL 9?"
- **Check parameter availability**: "Does `ad_gpo_implicit_deny` exist in RHEL 8?"
- **Detect version differences**: "What changed in sssd-ad(5) between RHEL 8 and RHEL 9?"
- **Find documentation**: "Which man page documents the `krb5_store_password_if_offline` parameter?"
- **Validate tool names**: "Is `msktutil` shipped with RHEL?" (It's not — no man page exists.)

## Data storage

Man pages are stored locally as plain text files:

```
manpages/
├── rhel8/
│   ├── sssd.conf.5.txt
│   ├── sssd-ad.5.txt
│   └── ...
├── rhel9/
│   └── ...
└── rhel10/
    └── ...
```

The `manpages/` directory is gitignored — each user extracts their own copy. This ensures man pages match the latest available UBI image content.

## License

MIT
