# VideoAmp

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
