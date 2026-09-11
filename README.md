# AI PR Review — bring your own subscription

Automated pull request reviews powered by the AI coding subscription you
**already pay for** — GLM Coding Plan (Z.ai), Kimi Code, or any
Anthropic-compatible endpoint — instead of a per-seat review bot.

Built on the official
[`anthropics/claude-code-action`](https://github.com/anthropics/claude-code-action),
so reviews are done by a real agent that can explore your repo, not a single
chat completion over a diff.

## Why

Every hosted AI review tool charges per seat or per token. Meanwhile many
teams already have flat-rate coding subscriptions (GLM Coding Plan, Kimi Code)
with unused quota. This workflow points the Claude Code Action at your
subscription's Anthropic-compatible endpoint so PR reviews burn quota you
have already paid for.

## Quickstart

**1. Add the secrets** (repo *Settings → Secrets and variables → Actions*, or
once at the organization level to share across repos):

| Secret | Example (GLM Coding Plan) | Example (Kimi Code) |
| --- | --- | --- |
| `ANTHROPIC_AUTH_TOKEN` | your Z.ai plan API key | your Kimi Code API key |
| `ANTHROPIC_BASE_URL` | `https://api.z.ai/api/anthropic` | `https://api.kimi.com/coding/` |
| `REVIEW_MODEL` | `glm-4.7` | `kimi-for-coding` |

**2. Add the caller workflow** to your repo at
`.github/workflows/ai-review.yml`:

```yaml
name: AI PR Review

on:
  pull_request:
    types: [opened, ready_for_review]

permissions:
  contents: read
  pull-requests: write
  issues: write
  id-token: write

jobs:
  review:
    if: github.event.pull_request.draft == false
    uses: MediaJel/ai-pr-review/.github/workflows/pr-review.yml@v1
    secrets: inherit
```

(Also in [`examples/ai-review.yml`](examples/ai-review.yml).)

**3. Open a PR.** The review lands as a single PR comment.

Switching providers later is a secrets change only — no code changes in any
repo.

## Quota and cost control

Coding subscriptions typically have rolling rate windows and weekly caps, and
stop dead at the cap — every PR review burns the key owner's quota. The
caller workflow is deliberately conservative:

- Runs on PR **open** (and draft → ready), not on every push
- Skips draft PRs
- Cancels superseded runs on the same PR (concurrency group)

To reduce burn further, gate on a label instead of running on every PR:

```yaml
on:
  pull_request:
    types: [labeled]
# ...
jobs:
  review:
    if: github.event.label.name == 'ai-review'
```

If you would rather not spend subscription quota at all, point the same
secrets at a cheap/free pay-as-you-go model (e.g. Z.ai's GLM Flash tier) —
the workflow does not care.

## Security notes

- **Prompt injection:** PR content is untrusted. The review prompt instructs
  the agent to treat everything in the PR as data, never as instructions, and
  to flag injection attempts as findings. Keep this in mind when customizing
  the prompt.
- **Forks:** secrets are not exposed to workflows triggered by fork PRs on
  public repos — reviews of external contributions simply won't run until a
  maintainer re-triggers them from a branch.
- **Secrets:** use organization-level secrets to rotate keys in one place.

## Versioning

Consumers should pin a major tag (`@v1`), not `@main`. Improvements ship as
new tags so one commit can never break every consumer at once.

## License

MIT — see [LICENSE](LICENSE).
