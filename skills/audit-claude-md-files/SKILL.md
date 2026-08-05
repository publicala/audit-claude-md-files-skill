---
name: audit-claude-md-files
description: >
  Audits every loaded CLAUDE.md line by line and prunes what a session can derive on its own, with every cut backed by evidence: enforcement checks, codebase counts, and low-effort agent panels. Completes the loop with feed-claude-md-files (adds rules) and bake-claude-md-files (converts rules to tooling).
user-invocable: true
disable-model-invocation: true
---

Read every CLAUDE.md the project loads: root, nested directories, and the referenced docs they point to. Judge each line against one question: **what mistake does a fresh session make without it?** If the answer is "none", the line is inventory, not instruction.

Every resident line is paid for in every session (tokens ≈ characters / 4, label all figures "est."). The goal is not a shorter file. The goal is a file where every line changes behavior.

## The audit, in order

### 1. Inventory

List each resident file with its est. token cost. Referenced docs load on demand and are close to free, so they are never a cut target on size alone. They enter the audit only through their pointers (step 6).

### 2. Derivability pass

Cut what a fresh session reconstructs with a few tool calls:

- Setup commands (`composer install`, `npm install`) and standard CLI usage
- Stack and version inventories (lockfiles are the source of truth)
- Directory layouts and file listings
- Generic best practices ("write tests", "validate inputs", "use clear names")

Test: delete the line and name the mistake a session now makes. No mistake, no line.

### 3. Enforcement verification

Never trust a claim (yours or the file's) that "the linter handles this". Open the formatter config, the architecture tests, and the CI pipeline, and confirm. Then classify each rule by its feedback loop:

- **Auto-fixed at format time**: the formatter rewrites violations silently, so the prose costs context and prevents nothing. Cut it.
- **Fails at suite time only** (architecture test, CI check): the prose can still pay for itself by preventing a write, fail, rewrite roundtrip. Keep it only when step 4 shows the surrounding code teaches the wrong pattern.
- **Unenforced**: decide entirely on step 4. Consider proposing `bake-claude-md-files` for it.

Expect surprises in both directions: rules believed enforced that are not, and rules believed prose-only that a formatter already fixes.

### 4. Code-gradient measurement

The codebase teaches conventions whether or not the doc repeats them. Count occurrences before judging:

- **Convention followed nearly everywhere**: the code teaches it. A new session copies its neighbors correctly without being told. Cut the line.
- **Counter-examples are common, or the dominant pattern is the banned one**: neighbors teach the wrong thing, so the line is the only corrective. Keep it.

Grep counts decide ("125 files import the banned class, 38 the right one"), impressions do not.

### 5. Capability-floor panel

Docs serve the weakest model that reads them, not the strong one auditing them. For each borderline block, spawn one independent low-effort agent and ask: "You are working in this codebase. Does this block change the code you produce? Answer KEEP or CUT with one reason." One agent per block, no shared context between them. Accept the panel's verdicts unless one contradicts hard evidence from steps 3 and 4.

While judging examples, verify every symbol they reference against the real codebase. An example that calls a method that does not exist is worse than absence: it teaches a wrong API. Cut it regardless of the panel, and flag it, because that drift means nobody has checked the examples in a while.

### 6. Pointer and description audit

A pointer carries exactly two things: the trigger (when to read) and the path. Never a content summary. Summarizing the target loads its vocabulary into every session, which defeats the deferral.

Discriminator: does this phrase help decide WHEN to read the file (routing key, keep) or does it describe what you WILL LEARN there (content summary, cut)?

```markdown
<!-- Cut: content summary, the target's contents leak into every session -->
See docs/payments.md for the retry flow, webhook signatures, refund windows, and the idempotency-key format.

<!-- Keep: trigger + path -->
Read docs/payments.md before touching payment or refund code.
```

The same rule governs skill frontmatter descriptions: triggers and routing keywords stay, mechanics and step lists move to the body. Safety and precedence clauses stay in the description ("never mutates remote state", "OVERRIDES the global skill") because they change the invocation decision itself.

### 7. Deduplication

State each fact once, at the smallest scope that covers its readers. When a rule repeats across files, keep the copy closest to where it applies, keep the load-bearing identifier resident (the helper name, the command), and defer the rationale to one referenced doc.

## Never cut

- Safety prohibitions ("never force-push", "never run the seeder against a shared database")
- Gotchas that contradict appearances (the call that silently no-ops, the flag that looks optional but is not)
- Conventions that differ from the framework or tool default
- Precedence and routing clauses
- Read-triggers for referenced docs

## Extract, don't delete

Content needed only in a specific situation moves verbatim to a referenced file, leaving a one-line read-trigger behind ("read X before doing Y"). Deferral keeps the knowledge and drops the always-resident cost. Deletion is only for content that fails step 2 outright.

## Approval and apply

Present the full report before editing anything. Per finding: the verdict (cut, keep, move, defer), the exact text affected, and the evidence (config line, grep count, panel verdict). Only edit after approval.

- **Checked-in files**: granular commits, one concern per commit, on a branch with a PR, so reviewers judge each cut in isolation.
- **Local files** (user-level memory, `CLAUDE.local.md`): edit directly, back up first.

Close the report with before and after est. token totals per file.

## Pairs with feed and bake

- `feed-claude-md-files` adds rules from observed patterns
- `bake-claude-md-files` converts crystallized rules into tooling and removes the prose
- `audit` prunes and verifies what remains

Run `audit` when CLAUDE.md files have grown without review, and after a long stretch of `feed` runs.
