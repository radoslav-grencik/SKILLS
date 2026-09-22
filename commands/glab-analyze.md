---
description: Analyze GitLab issue(s), related discussions, attachments, merge requests, and repository history, then produce a concise issue summary and a detailed solution proposal for approval.
agent: plan
---

Analyze the GitLab issue references in `$1` and write the final report in `$2`.

Input rules:

- `$1` is one or more GitLab issue references: issue numbers, comma-separated numbers, or full GitLab issue URLs. If multiple references are needed, the user may pass them as one quoted, space-separated argument.
- `$2` is the output language. If it is missing, empty, or unsupported, use Slovak (`slovenčina`). Preserve technical names, code symbols, URLs, and issue titles where translating them would reduce precision.
- If a reference is only a number, resolve it against the current GitLab project when possible. If the project cannot be determined, ask one concise clarification question instead of guessing.

Workflow:

1. Parse and deduplicate all issue references. Never silently drop an invalid reference; report it and continue with valid references.
2. Study each issue using the available GitLab tools. Read the title, description, state, labels, milestone, assignees, author, timestamps, linked issues, related issues, all relevant comments/discussions, and any available uploaded images or other attachments. Inspect images when they contain useful evidence.
3. Follow related work: linked and mentioned merge requests, commits, branches, discussions, and other issues. For merge requests, inspect their description, discussions, changed files, commits, and pipeline/status information when available. Do not stop at the first linked item.
4. Inspect the local repository and Git history for relevant code and unresolved context. Use search, file reading, `git log`, `git blame`, and focused diffs where useful. Look for existing implementations, recent regressions, TODOs, and code paths implicated by the issue. Do not invent repository facts that were not verified.
5. Reconcile the evidence. Separate confirmed facts from assumptions, conflicting statements, and unanswered questions. If access is missing or an attachment cannot be read, state exactly what could not be verified and how that limits the proposal.
6. Do not edit files, create commits, modify GitLab issues, open merge requests, or make any other mutation. This command is for analysis and approval of a proposed solution only.

Final report:

- Start with a short summary of what the issue(s) concern and the user-visible impact.
- For multiple issues, identify shared scope and conflicts, then give a clearly separated result for each issue.
- Include a detailed, concrete solution proposal suitable for approval: root cause, affected areas/files or likely locations, implementation steps in order, data/API/UX implications, edge cases, risks, and a focused validation/test plan.
- Include relevant evidence with issue/MR/commit/file references and links where available.
- Mark each statement as confirmed, inferred, or unresolved when that distinction matters.
- End with a short list of decisions or clarifications needed before implementation and a direct approval question.
- Keep the summary short, but make the solution proposal detailed enough that implementation can start after approval.
