# Absol Privacy Policy

**Last updated:** 2026-11-26 · **Version:** 1.0

Absol is a GitHub App operated by Broja Labs (broja.dev), in Toronto, Ontario, Canada that reviews pull requests for accessibility issues. This policy describes what Absol reads, what it sends outside GitHub, what it keeps, and how to stop all of it.

## In short

- Absol reads your source from GitHub to analyse it: only files changed in the pull request, only the file extensions you configure.
- **If contextual AI analysis is on (the default), short snippets of your source are sent to Anthropic's Claude API.** A snippet is the line a finding sits on plus five lines either side. You can turn this off per repository.
- Absol has no database. It stores nothing about your code between pull request events.
- Everything durable it produces lives in your own repository, as check runs and pull request comments you control.

## What Absol reads

When a pull request is opened or updated, Absol asks GitHub for:

- the list of changed files and each one's diff
- the full content of each changed file whose extension you have configured for review (by default `.js`, `.jsx`, `.ts`, `.tsx`), including lines you did not change
- its own previous comments on the pull request, so it does not post the same finding twice and can retire findings you have fixed
- `.a11y-agent.yml` from the pull request's base branch, if present

To do this it requests the GitHub permissions `contents: read`, `pull_requests: write`, `issues: write`, `checks: write` and `metadata: read`. It does not read files you did not change, branches other than the pull request's head and base, human review comments, issues, or any repository it is not installed on. It never pushes commits.

Findings are reported only for lines the pull request changed. Pre-existing issues elsewhere in a changed file are not reported.

## What Absol writes

Review comments on changed lines, one summary comment per pull request (updated in place on each push), and a check run named **Absol** on each commit. Nothing else.

## What is sent to a model provider

**Only when contextual analysis is enabled.** Two things must both be true: the Absol service has a model provider configured, and your repository has not opted out. If either is false, Absol runs static analysis only and no source leaves GitHub and the Absol service.

When it is on, for each finding Absol sends:

| Sent | Purpose |
|---|---|
| The single source line the finding sits on | What the rule flagged |
| Up to five lines either side of it | Context to judge whether the finding is a real violation |
| The repository-relative file path and line number | So the model can refer to the location |
| The rule's identifier and message (Absol's own text) | What is being evaluated |
| For a proposed fix: the model's explanation from the evaluation step | Continuity between the two calls |

**It does not send** whole files, files without findings, your git history, the diff, branch or commit messages, your repository or organisation name, any contributor's name or email, or any GitHub token.

At most **25 findings per review** are sent. Beyond that, remaining findings are reported from static analysis alone.

Snippets are sent to **Anthropic, PBC** via the Claude API, model `claude-sonnet-4-5`. Absol's Anthropic account is governed by Anthropic's commercial terms, under which API inputs and outputs are not used to train Anthropic's models. Anthropic may retain API data for a limited period for trust-and-safety purposes as described in its documentation; Absol does not have a separate zero-retention agreement. Anthropic's handling of that data is governed by:

- https://www.anthropic.com/legal/privacy
- https://www.anthropic.com/legal/commercial-terms

### Turning contextual analysis off

In `.a11y-agent.yml` at the root of your repository, on the base branch:

```yaml
llm:
  enabled: false
```

From then on Absol runs static analysis only. Every rule still runs and findings are still reported with mechanical fixes where a rule has one; you lose the contextual commentary and AI-proposed suggestions. The switch is checked before any model client is created, so a repository that opts out has no model client in the process at all.

## What is logged

Absol writes operational logs containing **metadata only**:

- repository identifier, pull request number and commit SHA
- file paths, line numbers and rule identifiers of findings
- for each model call: the request type, duration, token counts, the model's verdict and confidence, and whether a fix was proposed. Not the proposed fix, and not the model's explanation.

**Your source code is not logged.** The service has a diagnostic setting that would log full prompts and responses; it is off in production and is never enabled for the shared service. If it were ever enabled to investigate a problem reported by a specific installation, that installation would be told in advance, the setting would be time-limited, and the resulting log entries deleted afterwards.

Logs are held on **Railway** infrastructure in **us-east1** and retained for **[30] days** per the hosting plan. They are readable only by broja labs operators. No third-party error tracker, metrics or analytics service receives any of this data.

## What is stored

**Nothing.** Absol has no database, no cache and no file storage. Findings exist in memory for the duration of one pull request event and are discarded when it finishes. A re-run re-reads your code from GitHub and analyses it from scratch.

The durable output is **yours, in your repository**: the check run on the commit, the summary comment, and the inline review comments. These are written through the GitHub API, stored by GitHub under your repository, and subject to GitHub's privacy policy and your own retention settings.

## Uninstalling

Removing Absol from a repository or organisation revokes its access immediately: GitHub stops sending events and the installation token stops working. No further code is read.

There is no stored data to delete on Absol's side, because nothing was stored. What remains:

- **Comments and check runs already posted.** They stay in your repository unless you delete them. Absol cannot remove them after its access is revoked. If you want them gone, delete them before uninstalling.
- **Snippets already sent to Anthropic**, subject to Anthropic's retention described above.
- **Operational log entries already written**, which expire on the retention schedule above. You may request earlier deletion at the contact below.

## Security

Webhook deliveries are verified with a per-App secret. API credentials are stored as encrypted environment variables on the hosting platform and are never committed to source. Model-proposed fixes are validated by Absol against the original code before they are shown, and can only be applied by a person clicking "Commit suggestion" in GitHub.

## Legal basis and your rights

Absol processes repository content on behalf of the organisation or user that installed it, in order to provide the review that installation requested. Where GDPR or UK GDPR applies, the lawful basis is performance of a contract (Art. 6(1)(b)) for the review itself and legitimate interests (Art. 6(1)(f)) for operational logging. broja labs is located in Canada and is subject to PIPEDA. A data processing agreement is **available on request**.

Contributor names and emails are not processed; the only personal data Absol handles is incidental, such as a name appearing in a source comment within a reviewed snippet.

## Changes

Material changes to this policy will be noted in Absol's release notes and reflected in the version and date at the top of this page.

## Contact

**ogoelumeze@gmail.com**
**Broja Labs, Toronto, Ontario, Canada**

Questions about this policy, data requests, and deletion requests go to the address above.
