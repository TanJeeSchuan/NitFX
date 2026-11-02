# NitFX

PR review bot for GitHub Actions. It runs [fx](https://fx.sh) against any OpenAI-compatible endpoint and posts one comment per PR, updated on every push.

It only reviews PRs opened by the repo owner. Drafts are skipped.

## Usage

`.github/workflows/nitfx.yml`:

```yaml
on:
  pull_request:
    types: [opened, synchronize, ready_for_review]

jobs:
  review:
    permissions:
      contents: read
      pull-requests: write
    uses: TanJeeSchuan/NitFX/.github/workflows/review.yml@main
    with:
      base_url: https://api.example.com/v1
      model: your-model-id
    secrets:
      LLM_API_KEY: ${{ secrets.LLM_API_KEY }}
```

`.github/nitfx.md`, the review prompt (the PR diff is appended):

```md
Review this PR diff. Read surrounding code if needed.
Report only real bugs, with file:line. If nothing is wrong, say LGTM.
```

## Inputs

| Input | Default | |
|---|---|---|
| `base_url` | required | API prefix; fx appends `/chat/completions` |
| `model` | required | Model id |
| `effort` | unset | Passed to `fx --effort` (e.g. `low`, `medium`, `high`) |
| `prompt_file` | `.github/nitfx.md` | Path in the reviewed repo |
| `fx_version` | `0.0.16` | |

Secret `LLM_API_KEY` is sent as a bearer token.
