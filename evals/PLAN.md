# Eval plan (not built)

Fixture (scaffold_script): tiny repo whose CLAUDE.md plants one of each:

- derivable version inventory → expect CUT
- gotcha that greps clean because the rule works → expect KEEP
- rule falsely claiming linter enforcement → expect verified, then judged
- example calling a method that does not exist → expect flagged as CUT
- pointer that summarizes its target file → expect rewritten to trigger + path
- rule with a real boundary plus a drifting inventory → expect rewrite that keeps the boundary

Prompt: "Run the audit on this repo." Grader: LLM judge checks the report for the six verdicts. Run: `claude plugin eval audit-claude-md-files --ablation with-without --runs 1`
