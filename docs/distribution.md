# Distribution and landing-page review

**Review draft only.** No public repository, registry submission, service change,
or landing deployment is authorized by this branch. Publication and merging need
Joe's approval. The intended repository is `autofactory-build/autofactory-skill`.

## Format

Use the [Agent Skills specification](https://agentskills.io/specification): a
`skills/autofactory/` directory with `SKILL.md` and Markdown references. The skill
uses the shared format, not a Hermes plugin, MCP server, executable installer, or
SDK. Optional metadata values are strings. No platform-specific manifest is
required for this draft.

Git storage and the shared format make the content portable; each registry still
has its own submission, licensing, validation, and indexing requirements. This
branch does not claim acceptance by any registry.

The [skills CLI reference](https://github.com/vercel-labs/skills/blob/main/docs/commands.md)
documents repository and local-path installation. Once this repository exists
and the reader has access, the intended command is:

```sh
npx skills add autofactory-build/autofactory-skill --skill autofactory
```

This is an optional agent-manager command, not a dependency of using the HTTP API.
Private-repository installation requires the reader's own authorized Git access.
Do not invoke an external installer merely to validate this review draft.
Installed copies may need an agent-manager update; editing Git does not magically
update every already-installed local copy.

## Proposed landing integration — not deployed

The proposed `landing/llms.txt` follows the [llms.txt proposal](https://llmstxt.org/).
It is a concise agent-facing index, not an access-control file or a promise that
search engines or agents will automatically discover the business.

After approval and public release, the landing deployment can add these static
Cloudflare Pages redirects once:

```text
/skill.md https://raw.githubusercontent.com/autofactory-build/autofactory-skill/main/skills/autofactory/SKILL.md 302
```

`/llms.txt` can link directly to the raw Markdown reference files so agents do not
have to infer reference URLs from a redirect. Follow redirects only for these
public documentation reads; do not attach an AutoFactory API credential.

The canonical skill lives on the reviewed `main` branch in GitHub. Later merged
skill edits require no landing release, subject to normal GitHub/CDN caching.
Changing the landing index or redirect itself still requires a landing release.
Do not expose a private-repository token through the public site to make the
private draft readable.

Add a visible `For AI agents` link to `/skill.md`. An optional HTML discovery hint
is `<link rel="describedby" href="/llms.txt">`. Do not claim that a currently
unpublished API reference/OpenAPI URL is live: approve and verify its real URL
before adding it to the index.

## Approval and release checklist

- [ ] Joe approves the skill, workflow examples, and customer-facing wording.
- [ ] Verify the launch API origin, public documentation URLs, and payment mode.
- [ ] Approve the content license before public distribution.
- [ ] Open/review the private PR; do not auto-merge it.
- [ ] Explicitly approve making the repository public.
- [ ] Verify unauthenticated access to the exact published Markdown files.
- [ ] Validate actual selected registry requirements before any submission.
- [ ] Separately approve/deploy the landing index and redirect, then check their
      real HTTP responses, content types, links, and cache behavior.

The review workflow must never require public access to private operational code,
credentials, real customer identifiers, checkout capabilities, or production
artifacts.
