# Privacy

This describes what the accessibility review app does with your code. It covers the app's own behaviour, which you can verify against the source in this repository — every claim below names the file that implements it.

> **Operators must complete this before listing.** Lines marked **`[OPERATOR]`** depend on how a particular deployment is hosted and contracted, and cannot be answered from the source. They are left blank deliberately rather than filled with a plausible guess.

## In short

- Your source is read from GitHub to analyse it. Only files you changed, only extensions you configure.
- **If contextual AI analysis is on, short snippets of your source are sent to a third-party model provider** — the line a finding sits on, plus five lines either side. You can turn this off per repository.
- The app has no database. It stores nothing about your code between webhook deliveries.
- The durable output lives in your own repository, as check runs and pull request comments you control.

## What the app reads

When a pull request is opened or updated, the app asks GitHub for:

- the list of changed files, and each one's diff ([`diff-analyzer.ts`](packages/github-app/src/services/diff-analyzer.ts))
- the full content of changed files whose extension you configured ([`file-fetcher.ts`](packages/github-app/src/services/file-fetcher.ts))
- existing comments on the pull request, so it does not post the same finding twice ([`comment-fingerprint.ts`](packages/review/src/comment-fingerprint.ts))
- `.a11y-agent.yml` from your default branch, if present

It requests `contents: read`, `pull_requests: write`, `issues: write`, `checks: write` and `metadata: read` ([`app.yml`](app.yml)). It does not read files you did not change, branches other than the pull request's, or any repository it is not installed on.

Findings are reported only for lines the pull request changed ([`pull-request.ts`](packages/github-app/src/handlers/pull-request.ts)) — pre-existing violations elsewhere in a file are not reported.

## What is sent to a model provider

**Only when contextual analysis is enabled.** It requires two independent things to be true: the operator has configured a provider, and your repository has not opted out. Either one off means static analysis only — no source leaves the app.

When it is on, then **per finding**, the app sends:

| Sent | From |
|---|---|
| The single source line the finding sits on | `elementSource` |
| The five lines either side of it | `surroundingCode` |
| The repository-relative file path and line number | `filename`, `line` |
| The rule's id and its message — the app's own text, not yours | `ruleId`, `message` |
| For a proposed fix: the model's own explanation from the previous step | `explanation` |

See [`analysis-context.ts`](packages/review/src/violation-evaluator/analysis-context.ts) for the window, and [`prompts/`](packages/review/src/violation-evaluator/prompts/) for the exact text of every request.

**It does not send** whole files, files without findings, your git history, your diff, branch or commit messages, your repository name beyond the file path, any contributor's name or email, or any GitHub token.

At most **25 findings per review** are sent by default ([`contextual-analyzer.ts`](packages/review/src/contextual-analyzer.ts)); beyond that, remaining findings are reported from static analysis alone.

Snippets are sent to **`[OPERATOR]` — name the provider and model this deployment uses** (the app supports Anthropic and OpenAI; the operator configures one). What that provider then does with the data is governed by its own terms, not by this document:

- Anthropic: https://www.anthropic.com/legal/privacy and https://www.anthropic.com/legal/commercial-terms
- OpenAI: https://openai.com/policies/privacy-policy and https://openai.com/policies/api-data-usage-policies

**`[OPERATOR]`** — state whether this deployment has a zero-retention or no-training agreement with its provider. Do not assume one: API data is typically retained by the provider for a limited period for abuse monitoring unless separately agreed.

### Turning it off

In `.a11y-agent.yml` at the root of your repository:

```yaml
llm:
  enabled: false
```

From then on the app runs static analysis only, and no source is sent anywhere outside GitHub and this app. Every rule still runs and findings are still reported; you lose the contextual commentary and the AI-proposed suggestions. The switch is checked before any provider is constructed, so a repository that opts out has no model client in the process at all ([`pull-request.ts`](packages/github-app/src/handlers/pull-request.ts)).

## What is logged

The app writes operational logs. By default these contain **metadata only**:

- repository and pull request number, commit SHA
- file paths, line numbers and rule ids of findings
- for each model call: which response shape was requested, how long it took, token counts, the model's verdict and confidence, and whether a rewrite was proposed — **not the rewrite, and not the explanation** ([`logging.ts`](packages/llm/src/decorators/logging.ts))

**Your source code is not logged by default.** One deployment setting changes that: `A11Y_LLM_LOG_BODIES=true` logs full prompts and responses, which contain your source. It is off by default, and the code says why — on a hosted installation it would push other people's private source into the operator's log aggregation, where it is retained and searchable. The app logs a warning on every review when it is on.

**`[OPERATOR]`** — confirm `A11Y_LLM_LOG_BODIES` is unset in production, and state the hosting platform's log retention period and who can read those logs.

## What is stored

**Nothing.** The app has no database, no cache and no file storage. Its dependencies are a parser, two model SDKs, a schema validator, a YAML reader and the GitHub framework — there is no persistence layer to inspect, because there is none.

Findings exist in memory for the duration of one webhook delivery and are gone when it finishes. Nothing is carried from one review to the next: a re-run re-reads your code from GitHub and re-analyses it from scratch.

The output is durable, but it is **yours, in your repository**:

- the check run on the commit
- the summary comment on the pull request
- inline review comments on changed lines

These are written through the GitHub API, stored by GitHub under your repository, and subject to GitHub's own privacy policy and your retention settings.

**`[OPERATOR]`** — if this deployment adds anything stateful (an error tracker, metrics, an analytics service), disclose it here. As the source stands, there is nothing.

## Uninstalling

Removing the app from a repository or organisation immediately revokes its access: GitHub stops sending webhooks and the installation token stops working. No further code is read.

**There is no stored data to delete,** because nothing was stored. That is the whole deletion story on the app's side — not a promise to delete on request, but an absence of anything to delete.

What remains is in your repository, and is yours to remove:

- Comments and check runs the app already posted. They stay unless you delete them; the app cannot remove them after its access is revoked. Delete them as you would any comment.
- Snippets already sent to the model provider, which are subject to that provider's retention. **`[OPERATOR]`** — state the provider's deletion process, or that snippets are retained per the provider's policy for a stated period.
- Operational log entries already written, subject to the retention period above.

If you want the app's comments gone before uninstalling, do it in this order: delete the comments while the app still has access, then uninstall. Reversing it leaves the comments with no way for the app to help.

## Questions, and requests about your data

**`[OPERATOR]`** — supply a contact (an email or an issue tracker), the legal entity operating this deployment, its jurisdiction, and how to make a data request. Under GDPR or UK GDPR, also state the lawful basis for processing and whether a data processing agreement is available.

---

*This document describes the app's behaviour as implemented in this repository. Where it says what the code does, you can check it — the files are named throughout. Where a fact depends on how a deployment is hosted or contracted, it is marked `[OPERATOR]` rather than guessed at.*
