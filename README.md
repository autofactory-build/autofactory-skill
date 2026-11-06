# AutoFactory skill

Upload 3D models, get manufacturing quotes, and track orders with AutoFactory.
The agent handles the quote workflow; the buyer completes hosted checkout.

Start with [`SKILL.md`](skills/autofactory/SKILL.md). It links to the API reference
for each task, so agents can load what they need as they go.

## Install

```sh
npx skills add autofactory-build/autofactory-skill --skill autofactory
```

This is a private draft on a review branch. The command above is for the reviewed
release; it does not install this draft from `main`. Private access requires Git
credentials with repository access.

## What's included

- [`skills/autofactory/`](skills/autofactory/SKILL.md): the skill and task references.
- [Workflow map](skills/autofactory/references/api-workflow.md): an index of API tasks.
- [Distribution notes](docs/distribution.md): release checklist and proposed landing integration.
- [`landing/llms.txt`](landing/llms.txt): a draft discovery index, not yet deployed.

The skill is Markdown-only and uses the [Agent Skills format](https://agentskills.io/specification).
The curl examples are optional; any HTTP client works.

## Review status

API examples follow the AutoFactory OpenAPI contract and handlers. Offline checks
cover frontmatter, local links and shell syntax; they do not establish live API
or payment availability. The launch API endpoint still needs to be selected.

Content approval, a license choice and publication approval are pending. The PR
remains private and unmerged; registry submission and landing changes are separate
release steps. See the [release checklist](docs/distribution.md#release-checklist).

## Author

Joseph Rodrigues ([@DoWhileGeek](https://github.com/DoWhileGeek)), maintained under
[autofactory-build](https://github.com/autofactory-build).
