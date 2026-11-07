# Vibe SEO Plugin

SEO analysis for [Mistral Vibe](https://docs.mistral.ai/vibe/code/overview) — 26 skills and 19 subagents covering technical SEO, content quality, schema markup, sitemaps, local SEO, backlinks, GEO/AI search, and the Google APIs.

A native **Agent Plugins 1.0** port of [AgriciDaniel/claude-seo](https://github.com/AgriciDaniel/claude-seo). Every skill and subagent is preserved; the Claude-format agent files and the Claude manifest are not shipped.

## Install

The plugin must live at `~/.vibe/plugins/seo`. Fifty-one files reference that absolute path, so do not rename the directory.

```bash
git clone https://github.com/danieldannhofer-gif/vibe-seo-plugin ~/.vibe/plugins/seo
```

Then reload plugins in a running session:

```
/reload-plugins
```

`/plugins` shows what loaded. A healthy install reports `skills=26 agents=19 issues=0`.

## What's inside

| Component | Count | Location |
|---|---|---|
| Skills | 26 | `skills/<name>/SKILL.md` |
| Subagents | 19 | `ai.mistral.vibe/agents/*.toml` |
| Python runtime | — | `skills/seo/scripts/`, provisioned on first use |

Each subagent carries an `enabled_tools` allowlist derived from its upstream `tools:` frontmatter, so an agent only gets the tools it was written for. `seo-image-gen`, for example, cannot write files; `seo-technical` can.

## Usage

Every skill is `user-invocable`, so it appears in the `/` menu as `seo:<skill>`:

| Group | Commands |
|---|---|
| Hub | `/seo:seo` |
| Audit and report | `/seo:seo-audit` `/seo:seo-page` `/seo:seo-technical` `/seo:seo-content` `/seo:seo-schema` `/seo:seo-images` `/seo:seo-sitemap` `/seo:seo-sxo` `/seo:seo-drift` |
| Content | `/seo:seo-content-brief` `/seo:seo-cluster` `/seo:seo-plan` `/seo:seo-programmatic` `/seo:seo-competitor-pages` |
| Local | `/seo:seo-local` `/seo:seo-maps` |
| Platform | `/seo:seo-google` `/seo:seo-dataforseo` `/seo:seo-backlinks` `/seo:seo-ecommerce` `/seo:seo-hreflang` `/seo:seo-geo` `/seo:seo-agentic` `/seo:seo-flow` `/seo:seo-image-gen` |

Anything after the command is passed to the skill as plain text — Vibe has no `$ARGUMENTS` templating:

```
/seo:seo-audit https://example.com
```

The model also loads these skills on its own when a request matches a description, and spawns the subagents when a task warrants one. You do not have to invoke anything explicitly.

## Requirements

- Python 3.11 to 3.14 on `PATH`. The launcher creates an isolated virtual environment on first run and installs `skills/seo/requirements.txt` into it, including Playwright.
- Optional: `playwright install chromium` for SPA rendering. Without it the fetcher falls back to raw HTML.
- Optional credentials, only for the skills that need them: DataForSEO API key, Google Search Console / GA4 service account or ADC, PageSpeed Insights key.

## Live data (optional)

Two skills reach outside the bundled scripts. Both are optional; without them the rest of the plugin works normally.

**Search Console.** If your session exposes a Google Search Console connector, `seo:seo-google` prefers it and skips the bundled scripts. Otherwise the scripts handle it with a service account or ADC configured in `~/.config/claude-seo/google-api.json`. PageSpeed, CrUX, the Indexing API, GA4, and Keyword Planner always use the scripts.

**DataForSEO.** `seo:seo-dataforseo` is written against a DataForSEO MCP server, and five other skills enrich their output when those tools are present. Add the server to your own Vibe config:

```bash
vibe mcp add dataforseo --transport stdio \
  --command "npx -y dataforseo-mcp-server" \
  --env DATAFORSEO_USERNAME=you@example.com \
  --env DATAFORSEO_PASSWORD=yourpassword
```

This has to live in your config rather than the plugin's `mcp.json`. A plugin `mcp.json` can only expand `${PLUGIN_ROOT}` and `${PLUGIN_DATA}`, and Vibe passes stdio servers a fixed allowlist of OS variables (`PATH`, `HOME`, `TEMP`, and similar) rather than your full environment. There is no way for a plugin to forward an API key, so a server shipped there could never authenticate.

## Notes

- Connectors do not propagate to subagents. A subagent sees the built-in `file_system`, `process`, and `self` tools plus whatever its allowlist grants. Web access goes through `bash` and the bundled `render_page.py`, which is what the skill instructions already prescribe.
- The runtime writes its virtual environment and state outside the plugin, under `%LOCALAPPDATA%/claude-seo` on Windows or `~/.local/share/claude-seo` elsewhere. Set `CLAUDE_SEO_DATA_DIR` to move it.
- `skills/seo/scripts/release_sign.py`, `verify_release.py`, and `consistency_check.py` are upstream maintainer tooling. They are not loaded by Vibe.

## Attribution

Original plugin by [AgriciDaniel](https://github.com/AgriciDaniel), MIT licensed. This port keeps the upstream skill text, scripts, and reference data, and replaces the Claude-format packaging with the Vibe Agent Plugins 1.0 layout.

## License

MIT. See [LICENSE](LICENSE).
