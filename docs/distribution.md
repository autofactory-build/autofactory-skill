# Distribution

The repository is private and PR #1 is the content review. Merging, public
publication, registry submission and landing deployment remain pending approval.

## Package

The skill uses the [Agent Skills format](https://agentskills.io/specification):
`skills/autofactory/SKILL.md` with Markdown references. It has no scripts, SDK or
platform-specific runtime dependency. Each registry has its own license,
submission and validation requirements.

The intended installation command for the reviewed release is:

```sh
npx skills add autofactory-build/autofactory-skill --skill autofactory
```

The draft is still on a review branch rather than `main`. Private installation
requires repository access. See the [skills CLI reference](https://github.com/vercel-labs/skills/blob/main/docs/commands.md)
for source and authentication options. Installed copies need their normal update
step to pick up later changes.

## Landing integration (proposed)

After public release, add a visible `For AI agents` link to `/skill.md` and serve
`landing/llms.txt` at `/llms.txt`. The index follows the [llms.txt proposal](https://llmstxt.org/).
An optional HTML discovery hint is `<link rel="describedby" href="/llms.txt">`.

A Cloudflare Pages redirect can point at the source:

```text
/skill.md https://raw.githubusercontent.com/autofactory-build/autofactory-skill/main/skills/autofactory/SKILL.md 302
```

The redirect targets the raw file so its relative references resolve beside it.
The index also links to raw references directly. Fetch these as public documents,
without API credentials. Verify reference resolution with the actual reader at
release time.

Later skill edits need only a merge to GitHub; changing the index or redirect still
needs a landing release. GitHub/CDN caches and installed copies may update later.

## Release checklist

- [ ] Approve the skill content and API examples.
- [ ] Select and verify the launch API endpoint and payment mode.
- [ ] Choose a license.
- [ ] Approve merging PR #1 and making the repository public.
- [ ] Verify unauthenticated access to the exact published Markdown files.
- [ ] Check each selected registry's requirements before submitting.
- [ ] Approve and deploy landing discovery; verify responses, reference links,
      content types and caching.

Public API documentation links can join the index once their locations are
verified. The landing page must not need private GitHub credentials.
