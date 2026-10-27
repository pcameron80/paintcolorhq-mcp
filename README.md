# PaintColorHQ — Cross-Brand Paint Color Match (MCP server)

A hosted [Model Context Protocol](https://modelcontextprotocol.io) server that finds the closest **real, purchasable paint colors across 13 brands** for any hex value, scored with **CIEDE2000** (Delta-E) over 26,000+ colors.

No install, no API key. It's a remote server — point any MCP client at the endpoint.

```
https://www.paintcolorhq.com/api/mcp
```

Transport: stateless **Streamable HTTP**. Auth: none.

## Tools

Three tools. An agent holding a color name calls `find_paint_color` for its hex, then `match_paint_color` for the equivalents in other brands.

### `find_paint_color`

Look up a paint color by name, number or both, optionally with the brand in the query.

| Argument | Required | Description |
|----------|----------|-------------|
| `query` | yes | Name and/or number, e.g. `Agreeable Gray`, `SW 7029`, `HC-154`, `Repose Gray sherwin williams`. |
| `brand` | no | Restrict to one brand slug from `list_brands`. |
| `limit` | no | Number of colors, 1–10 (default 5). |

Returns brand, color name, color number, hex, availability (current, retired or discontinued) and a link to the color's page.

```
find_paint_color { "query": "SW 7029" }
→ 1. Agreeable Gray — Sherwin-Williams 7029 (#d1cbc1)
```

### `list_brands`

No arguments. The 13 brands with the slug the other tools take, the brand name and its number of colors.

### `match_paint_color`

Find the closest cross-brand paint color equivalents for a hex value.

| Argument | Required | Description |
|----------|----------|-------------|
| `hex` | yes | 6-digit hex, with or without `#` (e.g. `7F9B8E`). |
| `brand` | no | Restrict to one brand slug (`sherwin-williams`, `benjamin-moore`, `behr`, `valspar`, `ppg`, `dunn-edwards`, …). |
| `limit` | no | Number of cross-brand matches, 1–10 (default 10). |

Returns a readable summary plus `structuredContent` — for each match: brand, color name, color number, hex, a closeness label, the Delta-E, and a link to the color's page. Answers questions like *"what's the Behr equivalent of Sherwin-Williams Repose Gray?"*

**Example**

```
match_paint_color { "hex": "7F9B8E", "limit": 3 }
→ 1. Sioux Falls — Benjamin Moore 705 (#80968f) — Very similar
  2. Eucalyptus — Behr QE-44 (#8c9f97) — Very similar
  3. Phoenix Tears — Valspar 8003-35D (#8da898) — Very similar
```

## Connect

**Claude Code**
```bash
claude mcp add --transport http paintcolorhq https://www.paintcolorhq.com/api/mcp
```

**Clients that accept a remote URL** (Claude Desktop connectors, Cursor, etc.)
```json
{
  "mcpServers": {
    "paintcolorhq": {
      "type": "http",
      "url": "https://www.paintcolorhq.com/api/mcp"
    }
  }
}
```

**stdio-only clients** (via [`mcp-remote`](https://www.npmjs.com/package/mcp-remote))
```json
{
  "mcpServers": {
    "paintcolorhq": {
      "command": "npx",
      "args": ["mcp-remote", "https://www.paintcolorhq.com/api/mcp"]
    }
  }
}
```

## Verify

```bash
npx @modelcontextprotocol/inspector
# Transport: Streamable HTTP → URL: https://www.paintcolorhq.com/api/mcp → Connect → Tools

# Or a raw round-trip:
curl -s https://www.paintcolorhq.com/api/mcp -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"match_paint_color","arguments":{"hex":"7F9B8E"}}}'
```

## How the matching works

Every color is converted to CIE LAB space, and the server computes the CIEDE2000 Delta-E — the perceptual color-difference standard used in paint and coatings manufacturing — between your color and 26,000+ catalog colors across 13 brands, returning the closest in each. A screen match can still read differently on a wall under your lighting, so confirm a final choice with a physical sample.

## Source & links

- Server source lives in the main site repo: [pcameron80/paintcolorhq](https://github.com/pcameron80/paintcolorhq) → `src/app/api/mcp/route.ts` (docs: `docs/mcp-server.md`)
- Human docs: <https://www.paintcolorhq.com/api>
- Free REST endpoint (no key): <https://www.paintcolorhq.com/api/color-match?hex=7F9B8E>
- Free embeddable widget: <https://www.paintcolorhq.com/embed>
- Paid tier (batch + color-science fields) on [RapidAPI](https://rapidapi.com/support-weGRmXmTU/api/paint-color-match-api1)

## License

MIT — see [LICENSE](./LICENSE). Paint color names and numbers are trademarks of their respective manufacturers; this project is not affiliated with or endorsed by any paint brand.
