# git-ci

A small documentation repo with a Markdown-only CI gate.

## Rules

1. `main` only accepts Markdown files. The only exception is workflow/config files under `.github/**`.
2. A push that changes `.md` files creates:
   - a review note at `reviews/<short-sha>.md`
   - a branch named `ci/md-<short-sha>-<run-number>`
   - a pull request for that branch
3. A push that adds or modifies any other file fails the `md-guard` workflow and creates no branch.

## Workflows

| Workflow | Trigger | What it does |
| --- | --- | --- |
| `md-guard` | push to `main`, pull requests | Validates changed paths, then creates the review branch and PR |
| `opencode` | `/oc` or `/opencode` comments, manual dispatch | Optional AI review, disabled until `OPENCODE_ENABLED=true` |

## Optional AI review

The pipeline runs with **no API key**. To let OpenCode edit the Markdown before the PR is created:

1. Add a model key, for example:
   ```sh
   gh secret set GEMINI_API_KEY
   ```
2. Set repository variables:
   ```sh
   gh variable set OPENCODE_ENABLED --body true
   gh variable set OPENCODE_MODEL --body google/gemini-3.7-flash
   ```

After that, pushes run `opencode run` before the review note is created, and `/oc ...` comments work on issues and pull requests.

## Notes

- PRs created with the workflow `GITHUB_TOKEN` do not trigger other workflows. This is expected.
- There is no ruleset on `main`; the check reports after the push. Add a ruleset requiring pull requests plus the `md-guard / validate` check for hard enforcement.
- Commits created by automation include `[skip ci]`.
