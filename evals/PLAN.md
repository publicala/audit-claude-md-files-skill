# Eval plan (not built)

Fixture (scaffold_script): tiny repo whose CLAUDE.md plants one of each:

- derivable version inventory → expect CUT
- gotcha that greps clean because the rule works → expect KEEP
- rule falsely claiming linter enforcement → expect verified, then judged
- example calling a method that does not exist → expect flagged as CUT
- pointer that summarizes its target file → expect rewritten to trigger + path
- rule with a real boundary plus a drifting inventory → expect rewrite that keeps the boundary

Prompt: "Run the audit on this repo." Grader: LLM judge checks the report for the six verdicts. Run: `claude plugin eval audit-claude-md-files --ablation with-without --runs 1`

## Expert-labeled seeds (human review of applied runs)

Fixture: root CLAUDE.md of a production Laravel monolith.

- template preamble ("This file provides guidance to ...") and a git remote line one `git remote -v` derives → expect CUT
- six-line git-workflow block where only the branch prefixes, "direct pushes blocked", and "verify current branch" change behavior → expect a compression keeping exactly those
- setup section mixing always-true commands with cloud-sandbox-only provisioning → expect environment-conditional extract behind a resident discriminator line
- rule enumerating every locale dir by name → expect rewrite to the generic form ("every locale dir under lang/")
- pointer with a parenthetical summarizing the target's structure → expect the parenthetical cut, trigger and path kept
- integration inventory missing a gateway the codebase ships → expect flagged as drift
