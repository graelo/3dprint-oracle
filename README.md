<!-- mcp-name: io.github.graelo/3dprint-oracle -->

<p align="center">
  <a href="https://www.npmjs.com/package/@graelo/3dprint-oracle"><img src="https://img.shields.io/npm/v/@graelo/3dprint-oracle.svg" alt="npm version"></a>
  <a href="https://www.npmjs.com/package/@graelo/3dprint-oracle"><img src="https://img.shields.io/npm/dm/@graelo/3dprint-oracle.svg" alt="npm downloads"></a>
  <a href="https://github.com/graelo/3dprint-oracle/releases"><img src="https://img.shields.io/github/v/release/graelo/3dprint-oracle.svg?label=data%20release" alt="latest data release"></a>
  <a href="https://nodejs.org"><img src="https://img.shields.io/badge/node-%3E%3D18-brightgreen.svg" alt="Node.js 18+"></a>
  <a href="https://modelcontextprotocol.io"><img src="https://img.shields.io/badge/MCP-compatible-purple.svg" alt="MCP Compatible"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License"></a>
</p>

# 3dprint-oracle

3D printing filament and materials knowledge MCP server. Gives LLMs authoritative access to 9,700+ filament listings and curated material science knowledge.

> **This fork** ([graelo/3dprint-oracle](https://github.com/graelo/3dprint-oracle)) keeps the embedded database automatically in sync with upstream [SpoolmanDB](https://github.com/Donkie/SpoolmanDB) — see [Automatic data refresh](#automatic-data-refresh) below.

## Features

- **9,760 filament listings** from SpoolmanDB (87 manufacturers, 82 material types)
- **Material science knowledge** — properties, troubleshooting, recommendations
- **8 MCP tools** — search, lookup, compare, recommend, diagnose
- **Embedded data** — no API keys, no network calls at runtime
- **Automatic data refresh** — daily check for SpoolmanDB updates, rebuild + release on change

## Installation

The fork is published to npm as [`@graelo/3dprint-oracle`](https://www.npmjs.com/package/@graelo/3dprint-oracle) with a new semver version per data refresh — `npx` checks the registry on every start and auto-installs new versions, so you always run the latest data. Add to `~/.config/pi/agent/mcp.json` (same shape for any MCP client):

```json
{
  "mcpServers": {
    "3dprint-oracle": {
      "command": "npx",
      "args": ["-y", "@graelo/3dprint-oracle"],
      "exposure": "codemode",
      "description": "Search 3D printing filament materials database"
    }
  }
}
```

### Precompiled tarball (no build step)

Every data refresh also publishes a prebuilt npm tarball — dist files, database embedded, runtime dependencies excluded — under a stable, version-free URL:

```
https://github.com/graelo/3dprint-oracle/releases/latest/download/graelo-3dprint-oracle-latest.tgz
```

## Automatic data refresh

The [SpoolmanDB Sync workflow](.github/workflows/spoolmandb-sync.yml) runs daily (06:00 UTC) and compares upstream [SpoolmanDB](https://github.com/Donkie/SpoolmanDB) `main` HEAD against the SHA recorded in [`.spoolmandb-head`](.spoolmandb-head). When upstream changes, it:

1. Rebuilds `src/data/3dprint.sqlite` (`npm run fetch-data`)
2. Runs the full test suite
3. Bumps the npm version (`npm version patch`) so the refresh can be published
4. Commits the refreshed database + provenance SHA + version bump to `main`
5. Publishes a `data-<date>` GitHub Release with the upstream SpoolmanDB commit SHA in the release notes (provenance), plus `graelo-3dprint-oracle-latest.tgz` (stable-name copy of the npm tarball) and `graelo-3dprint-oracle-<version>.tgz`
6. Publishes `@graelo/3dprint-oracle@<version>` to npm via [trusted publishing](https://docs.npmjs.com/trusted-publishers/) (GitHub Actions OIDC — no long-lived tokens; requires the workflow to be configured as a trusted publisher in the package settings on npmjs.com, with `npm publish` ticked under allowed actions)

Manual runs are available via `workflow_dispatch` (Actions → SpoolmanDB Sync → Run workflow), with a `force` option to rebuild even when upstream has not changed.

## Tools

| Tool | Description |
|------|-------------|
| `search_filaments` | Search filaments by name, material, manufacturer, color |
| `get_filament` | Get full specs for a specific filament |
| `list_manufacturers` | Browse manufacturers with filament counts |
| `list_materials` | Browse material types with counts |
| `get_material_profile` | Authoritative properties for a material type |
| `compare_materials` | Side-by-side comparison of 2-3 materials |
| `recommend_material` | Get ranked recommendations for your requirements |
| `diagnose_print_issue` | Troubleshoot print problems with material-specific fixes |

## Data Sources

- Filament data: [SpoolmanDB](https://github.com/Donkie/SpoolmanDB) (MIT license)
- Material knowledge: Hand-curated from authoritative 3D printing references

## License

MIT
