---
description: Find and analyze GitLab issues and related work from issue numbers, URLs, or a text query, then produce a concise summary and a detailed solution proposal for approval.
agent: plan
---

Analyze the following GitLab references or search request and write the final report in the requested language:

<user-input>
$ARGUMENTS
</user-input>

Input rules:

- Accept one or more GitLab issue references (issue numbers, comma-separated numbers, or full GitLab issue URLs), a free-form text query describing what to find and analyze, or a mixture of references and text. Read the entire input; an unquoted multi-word query must not be truncated to its first word.
- Preserve the existing syntax of a reference or quoted input followed by an optional output language, for example `123 english` or `"chyba prihlasovania po zmene hesla" english`. For unquoted free-form text, treat the whole text as the search request unless the user explicitly requests an output language. If the language is missing, empty, or unsupported, use Slovak (`slovenčina`). Preserve technical names, code symbols, URLs, and issue titles where translating them would reduce precision.
- If a reference is only a number, resolve it against the current GitLab project when possible. If the project cannot be determined, ask one concise clarification question instead of guessing.
- For a text query, determine the GitLab project from explicit project context or the local repository's Git remote. Do not assume this configuration repository is the target project. If the intended project or search scope cannot be determined, ask one concise clarification question. If input is empty, ask for references or a description of what to search for.

Workflow:

1. Parse and deduplicate explicit issue references and identify any search intent in the text. Never silently drop an invalid reference; report it and continue with valid references. For text queries, infer the relevant keywords, identifiers, symptoms, and resource types (issues, merge requests, or related work), then search with the available GitLab tools in the resolved scope. Search issues across open and closed states with `scope: all`, and search merge requests across states when relevant; use supported search filters and pagination. Do not assume natural-language text is an issue number or that the GitLab search API understands a full sentence: try focused keyword variants when needed. Compare candidate titles and descriptions, inspect the strongest matches, and deduplicate them with explicit references. Analyze multiple relevant matches when they form one coherent topic; if unrelated candidates leave the intended target ambiguous, show a short candidate list with links and ask which to analyze. If no relevant matches are found, report the scope and queries tried and ask for a more specific clue instead of inventing a target.
2. Study each issue using the available GitLab tools. Read the title, description, state, labels, milestone, assignees, author, timestamps, linked issues, related issues, all relevant comments/discussions, and any available uploaded images or other attachments. Inspect images when they contain useful evidence.
3. Follow related work: linked and mentioned merge requests, commits, branches, discussions, and other issues. For merge requests, inspect their description, discussions, changed files, commits, and pipeline/status information when available. Do not stop at the first linked item.
4. Inspect the local repository and Git history for relevant code and unresolved context. Use search, file reading, `git log`, `git blame`, and focused diffs where useful. Look for existing implementations, recent regressions, TODOs, and code paths implicated by the issue. Do not invent repository facts that were not verified.
5. Reconcile the evidence. Separate confirmed facts from assumptions, conflicting statements, and unanswered questions. If access is missing or an attachment cannot be read, state exactly what could not be verified and how that limits the proposal.
6. Do not edit files, create commits, modify GitLab issues, open merge requests, or make any other mutation. This command is for analysis and approval of a proposed solution only.

Final report:

- Start with a short summary of what the issue(s) concern and the user-visible impact.
- When input includes a text query, briefly state the interpreted search intent, project/scope, queries used, and why the selected issues or merge requests match. If the primary match is a merge request, apply the same evidence and solution-proposal requirements to it even when no linked issue exists.
- For multiple issues, identify shared scope and conflicts, then give a clearly separated result for each issue.
- Include a detailed, concrete solution proposal suitable for approval: root cause, affected areas/files or likely locations, implementation steps in order, data/API/UX implications, edge cases, risks, and a focused validation/test plan.
- Include relevant evidence with issue/MR/commit/file references and links where available.
- Mark each statement as confirmed, inferred, or unresolved when that distinction matters.
- End with a short list of decisions or clarifications needed before implementation and a direct approval question.
- Keep the summary short, but make the solution proposal detailed enough that implementation can start after approval.
