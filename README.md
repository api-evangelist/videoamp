# VideoAmp

VideoAmp is a Los Angeles-based media measurement and advertising technology company, founded in
2014, certified by the U.S. Joint Industry Committee as a national TV currency provider alongside
Nielsen and Comscore. Its platform unifies linear TV, streaming and digital viewership onto one
dataset and identity spine (VALID) so agencies, brands and publishers can build audiences, plan and
optimize cross-screen media, and measure ad and content outcomes.

- Website: https://videoamp.com/
- Developer documentation: https://docs.videoamp.dev (Auth0-gated)
- GitHub: https://github.com/VideoAmp

## API surface

| Surface | Location | Notes |
|---|---|---|
| VideoAmp Public API | `https://api.videoamp.dev` | 118 operations, OAuth 2.0 bearer |
| VideoAmp MCP Server | `https://api.videoamp.dev/v1/mcp` | 101 tools, streamable HTTP, OAuth-protected |
| VideoAmp CLI | `github.com/VideoAmp/cli` | Single Go binary; also runs the MCP server locally |

## How this repo's contract was obtained

VideoAmp publishes **no anonymous OpenAPI**. `docs.videoamp.dev` 302-redirects every path to the
Auth0 universal login, `api.videoamp.dev` returns 404 for `/openapi.json`, `/swagger.json`,
`/v1/openapi.json`, `/api-docs`, `/docs` and `/redoc`, and an anonymous MCP `tools/list` returns 401.

The contract in `openapi/` was instead derived from the operation table that VideoAmp itself ships
inside its official CLI binary (GitHub release `v0.148.32`, `api_edition` 2026-07-31). Every path,
method, operationId, summary, description and parameter is reproduced verbatim from that binary's
`--help` output, and the 101 MCP tools come from `videoamp mcp list-tools`, which runs offline.
Request and response body schemas are not exposed on any anonymous surface and were deliberately
left unspecified rather than invented.

## Artifacts

- `openapi/` — OpenAPI 3.1, 118 operations across 84 paths and 14 tags
- `overlays/` — API Evangelist enhancements as an OpenAPI Overlay 1.0.0
- `mcp/` — MCP server manifest and the tool-to-operation crosswalk (101 tools, all bound)
- `cli/` — the `videoamp` command surface
- `skills/` — five packaged Agent Skills for the marquee flows
- `agentic-access/` — recommended `x-agentic-access` contracts for all 118 operations
- `authentication/`, `scopes/`, `well-known/` — Auth0 OIDC, RFC 8414 and RFC 9728 metadata
- `conventions/`, `errors/`, `lifecycle/`, `data-model/`, `conformance/` — runtime semantics
- `changelog/` — dated CLI release history
- `packages/` — registry survey (no first-party API client SDK exists)
- `security/` — domain security probe, trust center, and a verified-absent disclosure program
- `llms/` — generated llms.txt

## Notable gaps

- No anonymous OpenAPI, documentation or help-center content
- No first-party API client SDK in any language registry
- No A2A agent card, no `security.txt`, no `/.well-known/api-catalog`
- No published deprecation policy, SLA or status page
- No webhook, event or streaming surface — AsyncAPI is genuinely not applicable
- Errors are a proprietary `<AREA>_<NNNN>` code scheme, not RFC 9457
