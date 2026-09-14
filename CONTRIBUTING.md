# Contributing

Thanks for your interest! This project is intentionally small — one reusable
workflow, one caller template, docs — so contributions are easy to review.

## Great first contributions

- **Test an untested provider** (Kimi Code, DeepSeek, or anything else with an
  Anthropic-compatible endpoint) and report back — we'll update the README
  table.
- **Prompt improvements**: better review instructions in
  `.github/workflows/pr-review.yml`, with before/after examples.
- **Docs**: translations, screenshots, setup gotchas you hit.

## How to contribute

1. Open an issue first for anything bigger than a typo — it saves everyone time.
2. Fork, branch, and open a PR against `main`.
3. Your PR will be reviewed by the workflow itself (dogfooding) plus a human.
4. Keep the caller template and README in sync with any workflow change.

## Committing and releases

- Only maintainers push tags. Releases follow semver: `v1.0.0`, `v1.1.0`, …
  with the major tag (`v1`) floated forward.
- Never commit secrets. The workflows reference secrets by name only.

## Code of conduct

Be kind, be specific, assume good intent. That's the whole policy for now.
