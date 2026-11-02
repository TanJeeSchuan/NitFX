# NitFX

PR review bot for GitHub Actions. It runs [fx](https://fx.sh) against any OpenAI-compatible endpoint and keeps one review comment per PR, updated on each review.

Only the repo owner can trigger it: PRs they open (drafts skipped), or `/review` comments they post.

## Usage

`.github/workflows/nitfx.yml`:

```yaml
on:
  pull_request:
    types: [opened, synchronize, ready_for_review]
  issue_comment:
    types: [created]

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

## When it reviews

The `on:` block is the setting.

| Mode | `pull_request.types` |
|---|---|
| Never (only `/review`) | drop `pull_request` |
| On PR opened | `[opened, ready_for_review]` |
| On new pushes | `[opened, synchronize, ready_for_review]` |
| On all events | `[opened, synchronize, reopened, ready_for_review, edited]` |

Drop `issue_comment` to disable `/review`.

## `/review`

Comment `/review` on a PR to review it now. Text after the command is added to the prompt:

```
/review focus on the migration, ignore the tests
```

The bot reacts with 👀 when it starts. `issue_comment` workflows run from the default branch, so `/review` only works once the caller workflow is on `main`.

`/review` works on PRs from anyone, including forks. The review checks out the PR, so its `.github/nitfx.md` and `AGENTS.md` steer the model, and fx runs with full access next to `LLM_API_KEY`. Read a stranger's PR before you `/review` it.

## Inputs

| Input | Default | |
|---|---|---|
| `base_url` | required | API prefix; fx appends `/chat/completions` |
| `model` | required | Model id |
| `effort` | unset | Passed to `fx --effort` (e.g. `low`, `medium`, `high`) |
| `prompt_file` | `.github/nitfx.md` | Path in the reviewed repo |

fx is installed at its latest release on every run.

Secret `LLM_API_KEY` is sent as a bearer token.
