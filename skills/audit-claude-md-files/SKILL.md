---
name: audit-claude-md-files
description: >
  Prunes loaded CLAUDE.md files down to the lines a fresh session cannot derive on its own, every cut backed by evidence. Use when CLAUDE.md files have grown without review, or after a stretch of feed-claude-md-files runs (feed adds rules, bake-claude-md-files bakes them into tooling, audit prunes what remains).
user-invocable: true
disable-model-invocation: true
---

Judge every line of every in-scope CLAUDE.md against one question: **what mistake does a session make without it?** If the answer is "none", the line is inventory, not instruction. The goal is not a shorter file. The goal is a file where every line changes behavior.

Throughout the audit, "cut" means recording a cut verdict in the report. No file changes before approval, no exceptions.

## Scope and load model

Default scope is the current project: its root CLAUDE.md, nested CLAUDE.md files, and the rule files they point to. User-level and ancestor-directory files load in every project, so include them only when the user asks, and evaluate them against a session in an arbitrary project (codebase greps and CI checks do not apply to them). Exclude vendored code, build output, and worktree copies everywhere: from the inventory and from every grep.

Not everything called "referenced" is deferred. Classify each file before judging it:

- **Always resident**: the root CLAUDE.md, user-level and ancestor files, and anything they pull in with `@path` imports. An `@path` import costs full price in every session. Moving text behind one saves nothing.
- **Scope-triggered**: nested CLAUDE.md files and path-scoped rule files load only for sessions working in their subtree. Judge their lines against a fresh session working there, not against every session.
- **Deferred**: a plain markdown pointer loads nothing until an agent chooses to read the target. This is the only class where content is close to free.

## Never cut

Check every line against this list before anything else. A match is a KEEP and skips the rest of the audit, with one exception: a rule the formatter rewrites silently may still fall in step 3, because the tool guarantees it.

- Safety prohibitions ("never force-push", "never run the seeder against a shared database")
- Gotchas that contradict appearances (the call that silently no-ops, the flag that looks optional but is not). Greps come back clean precisely because the line works, so step 4 would misread these as derivable.
- Conventions that differ from the framework or tool default
- Precedence and routing clauses
- Read-triggers for referenced docs

## The audit, in order

### 1. Inventory

List each in-scope file with its load class and est. token cost (label every figure "est."). Include equivalent rule files other agents consume (`.cursor/rules`, `AGENTS.md` and the like) in the inventory and the dedup pass, even though edits target CLAUDE.md files. Migrating a foreign-format rule file into the project's native format is never part of an audit apply: record it as a proposal and act only on explicit user approval.

### 2. Derivability pass

Cut what a fresh session reconstructs with a few tool calls:

- Setup commands (`composer install`, `npm install`) and standard CLI usage
- Stack and version inventories (lockfiles are the source of truth)
- Directory layouts and file listings
- Generic best practices ("write tests", "validate inputs", "use clear names")
- Self-referential document metadata and biography: version stamps ("**Version**: 2026-08-07"), "last updated" lines, rename history, drift-tracking clauses between files ("if the spine moves forward and this file doesn't..."), and rules phrased against the past ("previously X, now Z"). Git is the history. When the history carries a load-bearing constraint, reframe it present tense: that is a rewrite, not a cut. A date survives only when it is itself the instruction a session must apply, not a stamp about the document.

Test: delete the line and name the mistake a session in the file's scope now makes. No mistake, no line. A line that fails the test but matters in one identifiable situation moves instead of dying (see "Extract, don't delete").

The five classes above cut on sight. A cut for any other derivability claim needs a clean-context probe first: give one fresh low-effort agent a task the line governs in the file's scope, without the line, and record whether it derives the fact or makes the mistake. The probe's outcome is the verdict. The loaded auditor has read the line and cannot un-read it, so its own guess at what a fresh session derives is not evidence.

A probe is only valid when the probed line is absent from the probe agent's context. When the harness injects the audited file into every subagent (resident files always are), no probe can run clean: record that, and send the candidate to the capability-floor panel instead.

### 3. Enforcement verification

Never trust a claim (yours or the file's) that "the linter handles this". Inspect every enforcement surface: formatter and linter configs, static analysis, architecture or convention tests, git hooks, CI workflows. Record which surfaces you checked per rule. Then classify each rule by its feedback loop:

- **Auto-fixed at format time**: confirm the tool's file globs cover the affected paths and that it runs before code lands (hook or CI), then cut the prose. Violations get rewritten silently, so the line prevents nothing.
- **Fails at suite time only** (architecture test, CI check): the prose can still pay for itself by preventing a write, fail, rewrite roundtrip. Leave the verdict open. Step 4 closes it: keep when neighbors teach the wrong pattern, cut when they teach the right one.
- **Unenforced**: step 4 decides, and the rule is a candidate for `bake-claude-md-files`.

Expect surprises in both directions: rules believed enforced that are not, and rules believed prose-only that a formatter already fixes.

### 4. Code-gradient measurement

The codebase teaches conventions whether or not the doc repeats them. Grep before judging, scoped to the subtree the audited file governs and to first-party code only, and report numerator, denominator, and the exclusions used:

- **Compliance at 90% or above**: the code teaches the convention, a session copies its neighbors correctly. Cut.
- **Compliance at 70% or below**: neighbors teach the wrong pattern, so the line is the only corrective. Keep. (Example: 38 files import dates the required way, 125 the banned way. 23% compliant, KEEP.)
- **In between**: borderline. Step 5 decides.

### 5. Capability-floor panel

Docs serve the weakest model that reads them, not the strong one auditing them. A borderline block is a heading section or standalone bullet that steps 3 and 4 left undecided. Panel only those, never the whole file.

Spawn three independent low-effort agents per block. Each gets the repo path, the verbatim block, and one concrete task the block would govern, and answers: "Would this block change what you produce for this task? KEEP or CUT, one reason." Majority wins, ties are KEEP, the verdict is final.

Tell each panelist to answer the verdict only and never perform the task: a floor model handed a block will otherwise start executing it.

Separately, verify every first-party symbol an example references against the real codebase. An example that calls a method that does not exist teaches a wrong API and is worse than absence. Record it as a cut whatever the panel says, and flag it, because that drift means nobody has checked the examples in a while.

### 6. Pointer and description audit

Resolve every pointer's target first and flag the broken ones. Then judge the phrasing: a pointer carries exactly two things, the trigger (when to read) and the path. Never a content summary. Summarizing the target loads its vocabulary into every session, which defeats the deferral. Record these as rewrites.

Discriminator: does this phrase help decide WHEN to read the file (routing key, keep) or does it describe what you WILL LEARN there (content summary, cut)?

```markdown
<!-- Cut: content summary, the target's contents leak into every session -->
See docs/payments.md for the retry flow, webhook signatures, refund windows, and the idempotency-key format.

<!-- Keep: trigger + path -->
Read docs/payments.md before touching payment or refund code.
```

Exception: a fact stays in the pointer when the session needs it to pick WHICH target applies. A discriminator is routing, not summary.

The same rule governs skill frontmatter descriptions: triggers and routing keywords stay, mechanics and step lists move to the body. Safety and precedence clauses stay in the description ("never mutates remote state", "OVERRIDES the global skill") because they change the invocation decision itself.

### 7. Deduplication

State each fact once, at the smallest scope that covers its readers. When a rule repeats across files, keep the copy closest to where it applies, keep the load-bearing identifier resident (the helper name, the command), and defer the rationale to one referenced doc.

## Precise but generic

Phrasing gets judged only after a line earns its keep. Every specific detail in a kept line (a class inventory, an enumerated list, a count) is either load-bearing, meaning generalizing it would change what a session does, or a liability that drifts as the code moves. When a kept line carries non-load-bearing specifics, record a **rewrite**: drop those specifics, keep the load-bearing identifiers verbatim, and change nothing else. A rewrite never widens scope, weakens the boundary, or adds advice, and every identifier it keeps gets verified against the codebase like any example symbol.

Two senses of "generic" live in this skill. Step 2 cuts generic best practices because they pin no boundary. Generic here means phrased at the pattern level while still pinning one. When dropping the driftable specifics would leave nothing a session cannot derive, the line was inventory all along: cut, not rewrite. Precision earns the keep, genericity makes it last.

```markdown
<!-- Rewrite: the inventory drifts as validators are added, the boundary does not -->
Our validators are EmailValidator, PhoneValidator, and VatValidator. Never write a new validator for a rule one of these already covers.

<!-- After: same boundary, load-bearing path kept, inventory dropped -->
Never write a new validator for a rule one in app/Validators already covers.
```

## Extract, don't delete

Content needed only in a specific situation moves verbatim to a referenced file, leaving a one-line read-trigger behind ("read X before doing Y"). A move must beat the pointer it leaves behind: content no longer than its read-trigger stays resident. Reuse the project's existing referenced-doc location (detect it from current pointers) instead of inventing a new one. Deletion is only for content that fails step 2 outright.

Environment-conditional content is its own extract class: sentences that bind only in some execution environments (a cloud sandbox, CI, containerized local dev). No load mechanism triggers on environment, so path scoping cannot help. The pattern is one resident discriminator line per environment naming the trigger and the doc ("cloud session? read X before running anything"), with everything conditional moved to that doc verbatim. Classifying each sentence by the environments it governs is judgement the loaded auditor shortcuts: hand the section to one clean-context agent with the single question "which execution environments does each sentence govern", and treat every section with mixed answers as an extract candidate.

## Report delivery

Before the inventory, ask one AskUserQuestion: does the user want the report as an interactive artifact or as plain text? When they pick the artifact, publish it as a live doc (`capabilities: {artifact: {}}`) so their in-page decisions persist and the session reads them back, and build it so they can decide while reading:

- Every finding names its file by repo-relative path, never a shorthand or a directory nickname.
- Each verdict row carries an approve checkbox, grouped per file, checked by default for recommended verdicts.
- Each file to be edited shows a diff of the proposed result against the current content. Draft the proposed files outside the repo (scratchpad) to generate these; the drafts double as the apply source.
- A copy-decisions control as the fallback path back into the session.

## Building the decision artifact

The artifact is a decision surface, not a document. The user often decides from a phone, so every item renders as a compact card: path, a one-sentence claim, one evidence line. Compact governs your own prose and never the source: text the user is approving (the line being cut, the rule being written) appears verbatim and whole, however long it runs.

- **Show the line, don't cite it.** Any claim about text the reader cannot see gets a collapsed "See the lines" block quoting the offending line verbatim in diff-removed styling, with the replacement (where one exists) in diff-added styling. The reader must never need the repo open to decide. Abridge very long lines with `[...]` and point at the full diff.
- **A note field on every row.** Every decision row carries a collapsed free-text "Add note", including near-certain items. The user may have an opinion or question anywhere.
- **Open-in-editor links.** Every file reference gets a small open link built on the user's editor URL scheme, detected from the machine (`$EDITOR`, installed apps): `zed://file{abs}:{line}`, `vscode://file{abs}:{line}`, `cursor://file{abs}:{line}`, `phpstorm://open?file={abs}&line={n}`. Always absolute paths, so the links hold from any worktree, and percent-encode them (keeping `/` in the path, encoding the whole PhpStorm query value) so a `#`, `?`, `%`, or `&` inside a path cannot truncate the target. Evidence references carrying file:line link the same way. Use `target="_blank"`, and stop click propagation on these links in a capture-phase handler, since they sit inside the checkbox label and a click must never toggle the row. The link opens on the machine running the browser, so emit these only when the session runs on the user's own machine: a remote session (SSH, devcontainer, cloud sandbox) reads a different filesystem and a different set of installed editors, so omit the links there rather than pointing an absent editor at a path that does not exist.
- **Word-level diffs.** Pair adjacent removed/added runs (SequenceMatcher on tokens, similarity at or above 0.4) and highlight only the changed tokens. Render each diff line as a `display:block` span and join the spans with no separator: a newline between block spans inside a `<pre>` renders as a phantom blank line. Lint-enforced docs keep whole paragraphs on one physical line, so wrap with `white-space:pre-wrap`, `overflow-wrap:anywhere`, and a hanging indent.
- **Questions as options.** Render each open question as radio options with a "recommended" chip plus a free-text field, mirroring AskUserQuestion. A bare textarea is only for questions with no concrete options.
- **Sticky decision bar.** Section links, a changed-from-default counter, and the copy-decisions control stay reachable while scrolling.
- **A fallback that carries everything.** The copy-decisions control serializes every control on the page: each checkbox, the selected option of each question, and each free-text field. The pasted export is an accepted approval path, so any state it drops is a decision the session then applies wrongly.
- **Triage chips.** Label each diff row by the review effort it needs (cuts-only rows are skimmable, rewrites are worth opening, adds carry new lines) and provide an expand-all-diffs control.
- **Mobile pass.** Verify the page at around 390px width before publishing. Collapse method and other secondary sections by default.
- **Republish safety.** Viewer decisions reach the session only because the page is a live doc (`capabilities: {artifact: {}}`), which saves the DOM a viewer's gesture changed back into the served document. Before any republish, fetch the live artifact and compare its state against defaults. Carry any non-default state into the rebuilt HTML, or do not republish: republishing over live decisions destroys them. Publish without that capability and the state never leaves the viewer's browser, where the session cannot read it: then the clipboard export is the only path back, and republishing is off the table once the user starts deciding.
- **Generate, don't hand-edit.** Build the page from a data-plus-template script in the scratchpad so every iteration regenerates it whole.
- **End with the trigger.** Close the page by telling the user the exact phrase that resumes the session, such as "read the artifact and apply".

## Approval and apply

Present the full report before editing anything. Per finding: the verdict (cut, keep, rewrite, move, defer), the exact text affected, and the evidence (surfaces checked, grep ratio, panel vote). Approval arrives item by item: from the artifact's saved state (or its pasted export) when the report is an artifact, from AskUserQuestion otherwise. Only edit after approval, and approval to edit is not approval to publish: confirm separately before creating commits, branches, or PRs.

- **Checked-in files**: granular commits, one concern per commit, on a branch cut from the default branch with a clean tree (stop and ask if the tree is dirty), with a PR, so reviewers judge each cut in isolation.
- **Local files** (user-level memory, `CLAUDE.local.md`): edit directly, back up first.

After applying, re-resolve every pointer you touched. Close the report with before and after est. token totals per file, listing deferred (moved) tokens separately from deleted ones.

## The quartet

- `feed-claude-md-files` adds rules from observed patterns
- `bake-claude-md-files` converts crystallized rules into tooling and removes the prose
- `audit-claude-md-files` prunes and verifies what remains
- `split-claude-md-files` moves what remains to the scope that reads it

Run `feed` after a working session, `bake` once enough rules have accumulated to be worth automating, `audit` when CLAUDE.md files have grown without review, and `split` after an audit leaves a resident file carrying rules that govern one area.
