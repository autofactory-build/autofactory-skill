# AutoFactory skill

Upload 3D models, get manufacturing quotes, and track orders with AutoFactory.
The agent handles the quote workflow; your user completes hosted checkout.

Start with [`SKILL.md`](skills/autofactory/SKILL.md). It links to the API reference
for each task, so agents can load what they need as they go.

## Install

```sh
npx skills add autofactory-build/autofactory-skill --skill autofactory
```

The public skill includes the current staging API URL:
`https://autofactory-api-staging.dowhilegeek.workers.dev`.
No separate API-host setup is needed. New API keys require an email and operator
activation.

## What's included

- [`skills/autofactory/`](skills/autofactory/SKILL.md): the skill and task references.
- [Workflow map](skills/autofactory/references/api-workflow.md): an index of API tasks.
- [Distribution notes](docs/distribution.md): distribution and landing integration.
- [`landing/llms.txt`](landing/llms.txt): the discovery index served by the landing page.

The skill is Markdown-only and uses the [Agent Skills format](https://agentskills.io/specification).
The examples are standalone curl commands with placeholders; any HTTP client works.

## Availability

The skill targets the staging API for early testing. Its examples use the included
API URL; tokens and resource IDs remain placeholders. Your user completes hosted
checkout. Update an already-installed copy to receive these instructions.

## Author

Joseph Rodrigues ([@DoWhileGeek](https://github.com/DoWhileGeek)), maintained under
[autofactory-build](https://github.com/autofactory-build).
