# AutoFactory customer agent skill

**Private human-review draft. Not publicly published or registered.** No license
has been selected; approve one before public release. This draft is not approval
for merging, public publication, deployment, payment or production actions.

## Contents

- [`skills/autofactory/SKILL.md`](skills/autofactory/SKILL.md): customer workflow and
  human-checkout boundary.
- [`references/api-workflow.md`](skills/autofactory/references/api-workflow.md):
  compact task map; load only the current task and missing prerequisites.
- Task-specific optional curl examples: [authentication](skills/autofactory/references/authentication.md),
  [catalogs](skills/autofactory/references/catalogs.md), [uploads](skills/autofactory/references/uploads.md),
  [RFQs](skills/autofactory/references/rfqs.md), [quotes](skills/autofactory/references/quotes.md),
  [checkout](skills/autofactory/references/checkout.md), and [orders](skills/autofactory/references/orders.md).
- [`references/troubleshooting.md`](skills/autofactory/references/troubleshooting.md):
  approval, polling, conflicts and uncertain-write recovery.
- [`docs/distribution.md`](docs/distribution.md): registry format and proposed
  one-time landing integration; not a deployment.
- [`landing/llms.txt`](landing/llms.txt): proposed discovery index for review only;
  its future publication links are not claimed to be live.

The skill payload is Markdown-only: no installed scripts, bundled executable code,
SDK or agent-specific tooling dependency. Its frontmatter follows the
[Agent Skills specification](https://agentskills.io/specification): the directory
matches `name: autofactory`, and metadata values are strings. This is format
compatibility, not a claim of registry acceptance or universal agent support.

## Private review

Review the local skill directory without executing its API examples. The draft
requires an operator-provided HTTPS origin and an approved key for that same
environment. It intentionally has no default production hostname and makes no
claim that production is reachable, tax handling is enabled, or live payments
are authorized. New-client onboarding issues an inactive key pending manual
operator approval; checkout remains a private, human-only action.

API requests were drafted against the repository OpenAPI contract and customer
handlers/tests. The workflow explicitly separates API authentication, signed raw
upload capabilities and signed browser-checkout capabilities. It does not expose
internal production operations or production artifacts. Pricing comes from the
returned itemized offer, not hardcoded staging/test amounts.

Before acceptance, independently validate frontmatter, local links, shell syntax,
request schemas, redaction and human-review boundaries. Any future live smoke test
requires separate operator authorization and must not create a payment session or
contact customers/printers without approval.

## Future installation — not currently available

After `autofactory-build/autofactory-skill` exists, is accessible, and the reviewed
skill is available there, the intended installation command is:

```sh
npx skills add autofactory-build/autofactory-skill --skill autofactory
```

Do not run this as proof of availability today. A private repository requires
repository access and configured Git authentication. Follow the upstream
[skills CLI command reference](https://github.com/vercel-labs/skills/blob/main/docs/commands.md)
for supported sources and authentication. No registry submission or public
listing has been performed; no public documentation URL is asserted here.

## Release gates

- Human approval of customer behavior, API accuracy and safety boundaries.
- License approval and public-content review (no credentials, actual customer IDs,
  private state or signed URLs).
- Authorized repository creation/access and review through the intended private PR.
- Separate approval for any later publication, registry listing or distribution.
