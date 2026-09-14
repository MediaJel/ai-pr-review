# AI PR Review — bring your own subscription

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Release](https://img.shields.io/github/v/release/MediaJel/ai-pr-review)](https://github.com/MediaJel/ai-pr-review/releases)
[![AI PR Review](https://github.com/MediaJel/ai-pr-review/actions/workflows/ai-review.yml/badge.svg)](https://github.com/MediaJel/ai-pr-review/actions/workflows/ai-review.yml)

**Agentic code review for GitHub pull requests, powered by the AI coding
subscription you already pay for** — GLM Coding Plan (Z.ai), Kimi Code, or any
Anthropic-compatible endpoint. No per-seat pricing, no per-review fees, no new
vendor.

If your team has coding-plan subscriptions sitting with unused quota, this
turns them into a review bot that costs you **$0 per review**.

## See it in action

This repo reviews its own PRs with the workflow it ships — open any
[pull request](https://github.com/MediaJel/ai-pr-review/pulls) to see a live,
unedited review.

Real excerpts from production use at MediaJel:

> **On a breadcrumb navigation change:**
> "Verified end to end: sitemap keys exist… target route exists… the regex is
> correctly anchored and requires an id segment, so the new-campaign flow is
> untouched, and lookalike segments like `edits/…` don't match."

> **Reviewing its own installation PR**, it caught that our triggers never
> re-reviewed new pushes — and that our caller pinned a *movable* tag while
> handing it org secrets:
> "`ai-pr-review` is a public repo and `v1` is a movable annotated tag…
> Whatever that ref points to at run time receives all this repo's secrets.
> Pinning to the full commit SHA bounds the blast radius."

Both findings were correct. We fixed them before shipping.

## Why not a hosted review bot?

| | Hosted bots (CodeRabbit, Copilot, …) | **AI PR Review** |
| --- | --- | --- |
| Price | $15–30 / dev / month | $0 marginal — uses your existing plan |
| Review depth | Varies | Full agent: explores files, traces routes, checks consumers |
| Provider | Locked to theirs | GLM, Kimi, DeepSeek — swap via one secret |
| Data path | Their servers | Straight from GitHub runner to your provider endpoint |
| Infra to run | None (theirs) | None (GitHub Actions) |

Under the hood it's the official
[`anthropics/claude-code-action`](https://github.com/anthropics/claude-code-action)
pointed at your provider's Anthropic-compatible endpoint — so reviews are done
by a real agent that can read your repo, not a single chat completion over a
diff.

## Quickstart (2 minutes)

**1. Add the secrets** (repo *Settings → Secrets and variables → Actions*, or
once at the organization level to share across repos):

| Secret | Example (GLM Coding Plan) | Example (Kimi Code) |
| --- | --- | --- |
| `ANTHROPIC_AUTH_TOKEN` | your Z.ai plan API key | your Kimi Code API key |
| `ANTHROPIC_BASE_URL` | `https://api.z.ai/api/anthropic` | `https://api.kimi.com/coding/` |
| `REVIEW_MODEL` | `glm-5.3` | `kimi-for-coding` |

**2. Add the caller workflow** to your repo at
`.github/workflows/ai-review.yml`:

```yaml
name: AI PR Review

on:
  pull_request:
    types: [opened, synchronize, ready_for_review, reopened]

permissions:
  contents: read
  pull-requests: write
  issues: write

concurrency:
  group: ai-pr-review-${{ github.repository }}-${{ github.event.pull_request.number }}
  cancel-in-progress: true

jobs:
  review:
    if: github.event.pull_request.draft == false
    uses: MediaJel/ai-pr-review/.github/workflows/pr-review.yml@v1
    secrets: inherit
```

(Also in [`examples/ai-review.yml`](examples/ai-review.yml). For maximum
supply-chain safety, pin `uses:` to a full commit SHA instead of the floating
major tag — see [Versioning](#versioning).)

**3. Open a PR.** The review lands as a single PR comment. Switching providers
later is a secrets change only — no code changes in any repo.

## Supported providers

| Provider | Endpoint | Status |
| --- | --- | --- |
| GLM Coding Plan (Z.ai) | `https://api.z.ai/api/anthropic` | ✅ Production-tested (`glm-5.3`, `glm-4.7`) |
| Kimi Code | `https://api.kimi.com/coding/` | 🔌 Endpoint-compatible, untested — reports welcome |
| DeepSeek | `https://api.deepseek.com/anthropic` | 🔌 Endpoint-compatible, untested — reports welcome |

Anything that speaks the Anthropic Messages API should work. Open an issue or
PR to add your provider to the table.

A note on plan terms: reviews burn the key owner's subscription quota. Coding
plans are generally licensed for interactive use in supported tools — check
your plan's terms before pointing CI at it, or use a pay-as-you-go key instead
(Z.ai's Flash tier is currently free and works fine for smaller diffs).

## Quota and cost control

Coding subscriptions typically have rolling rate windows and weekly caps, and
stop dead at the cap — every PR review burns the key owner's quota. The caller
workflow is deliberately conservative:

- Skips draft PRs
- Cancels superseded runs on the same PR (concurrency group), so rapid pushes
  only ever leave one review running

It re-reviews on every push to an open PR (`synchronize`) — that is where most
of your quota will go. To reduce burn, remove `synchronize` from the caller's
trigger list, or gate on a label instead of running on every PR:

```yaml
on:
  pull_request:
    types: [labeled]
# ...
jobs:
  review:
    if: github.event.label.name == 'ai-review'
```

## Security notes

- **Prompt injection:** PR content is untrusted. The review prompt instructs
  the agent to treat everything in the PR as data, never as instructions, and
  to flag injection attempts as findings. The agent's tool allowlist only
  permits `gh` commands, which can only reach `api.github.com`.
- **Forks:** secrets are not exposed to workflows triggered by fork PRs on
  public repos — reviews of external contributions simply won't run until a
  maintainer re-triggers them from a branch.
- **Secrets:** use organization-level secrets to rotate keys in one place.
  GitHub secrets are write-only and masked in logs.
- **Caller trust:** the caller hands the reusable workflow your secrets and
  write-scoped tokens. If you call a workflow you do not control, pin `uses:`
  to a full commit SHA — tags are movable.

## Versioning

Consumers should pin a major tag (`@v1`) or a full commit SHA, not `@main`.
Releases are immutable semver tags (`v1.0.0`, …); the major tag floats to the
latest release in that major line. Pinning a SHA is the strongest option.

## Contributing

Issues and PRs welcome — see [CONTRIBUTING.md](CONTRIBUTING.md). Good first
contributions: testing a provider marked "untested" above, prompt
improvements, and language/framework-specific review heuristics.

## License

MIT — see [LICENSE](LICENSE).

---

Built by [MediaJel](https://github.com/MediaJel). If this saves your team
money, a star helps others find it.
