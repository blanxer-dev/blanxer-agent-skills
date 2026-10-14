# Blanxer agent skills

Skills that let an AI agent build and run a website for a Blanxer store (BlanxerAPI).

## Claude Code

```
/plugin marketplace add blanxer-dev/blanxer-agent-skills
/plugin install blanxer-website@blanxer
```

The `blanxer-website` plugin brings:
- the **blanxer-website** skill (step-by-step: find the store, design, scaffold from the starter, check, deploy to the merchant's Cloudflare account, link back to Blanxer);
- the **Blanxer MCP** server (`https://mcp.blanxer.com/mcp`, sign in to Blanxer in the browser) so the agent can fix SEO, products and other store data while it builds.

Then tell Claude: *"Build my Blanxer store website. My store is `<subdomain>.blanxer.io`."*

## Other agents (Cursor, Codex, …)

Point the agent at https://developers.blanxer.com/llms.txt and add `https://mcp.blanxer.com/mcp` as a remote MCP server. The same steps are in the starter's `AGENTS.md`.

## Layout

```
.claude-plugin/marketplace.json          marketplace "blanxer"
plugins/blanxer-website/
  .claude-plugin/plugin.json
  .mcp.json                              Blanxer MCP (http, OAuth)
  skills/blanxer-website/SKILL.md        the procedure + 13 hard rules
  skills/blanxer-website/references/     checkout, images, deploy, mcp
```

Validate after edits: `claude plugin validate .`
