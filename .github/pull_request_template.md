<!--
Publica.la pull request template.
Reference copy: https://github.com/publicala/.github/blob/main/PULL_REQUEST_TEMPLATE.md

Delete every section that does not apply. Never leave a section holding "none" or "N/A".
The title carries more weight than anything below it. Imperative, complete sentence, understandable on its own in `git log`.
-->

Closes:

<!-- What this PR resolves and marks done: Linear issue, Nightwatch issue, Sentry issue, GitHub issue. One per line. A GitHub issue closes on merge only when the keyword sits on the reference itself, so write that line as `- Closes #123`. The heading alone does not close anything. Delete both lines when there is nothing to close. -->

-

Related:

<!-- Context this PR does not resolve: Slack thread, design doc, partially addressed Sentry or Nightwatch issue. Delete both lines when there is nothing to link. -->

-

## Summary

<!-- One sentence: what this does. -->

## Why

<!-- What the diff cannot show: the problem, why this approach over the alternatives, the trade-off or limitation you accepted. Keep the load-bearing fact inline (the error, the decision, the number) so the PR still reads when the link dies or the reader lacks access. Skip only for true one-liners. -->

## Solution

<!-- How it is solved. Skip for simple changes. -->

## Changes

<!-- What changed against the target branch, from a reviewer's perspective: files, classes, behavior. One idea per bullet, nested sub-bullets when a single item carries several. Do not narrate the diff, and do not use self-referential labels like "Setup overhaul" that name a goal instead of describing the change. -->

-

## Testing

```bash

```

<!-- Manual steps, when the change is not covered by tests. -->

## Deployment

<!-- Fill the rows that apply and delete the rest. Replace the whole block with "No special steps required" when none of them apply. -->

- [ ] Migrations:
- [ ] Cache or config clear:
- [ ] Env vars or feature flags:
- [ ] Monitor after release:
