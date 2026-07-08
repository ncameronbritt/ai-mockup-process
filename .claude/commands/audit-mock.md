If $ARGUMENTS is empty, ask which mock file(s) to audit and wait for the
answer before doing anything else — do not guess or default to auditing
everything in the mocks directory. A plain filename or relative path is
enough; there's no need to use @-mention, since this file gets opened with
the Read tool either way.

Perform a fidelity audit on $ARGUMENTS (or the file(s) just named).

Do NOT make any changes yet. Complete steps 1–4 in full, then wait for confirmation.

Throughout, prefer the Read, Glob, and Grep tools over shell commands
(find/grep/cat/ls via Bash) for locating and opening files. Each distinct
Bash invocation is its own permission prompt; Read/Glob/Grep cover the same
ground with far fewer prompts over the course of an audit.

━━ STEP 1 — READ SOURCES OF TRUTH ━━━━━━━━━━━━━━━━━━━━━━━━━━━
Read these in order before examining the mock:

1. The project conventions file (project-context.md, AGENTS.md, or root CLAUDE.md —
   whichever exists) — project-wide code quality rules (copyright headers,
   TypeScript configuration, import conventions, anti-patterns).
2. The CLAUDE.md in the mocks directory — mock-specific conventions
   (token patterns, code quality rules for mocks, reusable components).
3. The real shell components the mock shell (MockChrome.tsx or equivalent) was
   modelled from. Compare them — shell drift affects every mock.
4. Any real production components the mock imitates. Identify these by
   reading the mock first, then locating the originals in the codebase
   (e.g. if the mock has a pagination footer, find CustomPagination.tsx
   and compare them side by side).

━━ STEP 2 — EXAMINE THE MOCK ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Read $ARGUMENTS carefully. For each UI pattern, control, or component
used, identify the corresponding real production component or rule it
should match.

━━ STEP 3 — REPORT FINDINGS ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Produce a structured report with exactly two sections:

  DEVIATIONS — things that concretely differ from project patterns.
  For each: what the mock does, what the production code/rule says it
  should do, and a severity label:
    · High   — visually wrong or breaks conventions that are enforced
    · Medium — code quality issue (hardcoded values, wrong tokens, etc.)
    · Low    — minor inconsistency (spacing, label wording)

  INFORMATIONAL — patterns that are intentional design decisions or
  are consistent with other mocks (no fix needed). This section is
  required even if empty — it shows the audit was thorough.

Keep the two sections clearly separated. Do not mix deviations and
informational findings into one list.

━━ STEP 4 — PATTERN HARVEST ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Review the INFORMATIONAL findings from Step 3. For each pattern listed
there, check whether it is already documented in the CLAUDE.md's
Concrete Patterns section.

For any pattern that is correct and consistent but NOT yet documented,
propose it as an addition to the Concrete Patterns section. For each
proposed addition include:
  - A short name (matching the Concrete Patterns style)
  - A one-sentence description of the pattern
  - A code example if one would make it clearer

If there are no undocumented patterns worth adding, state that explicitly.

━━ STEP 5 — CONFIRM BEFORE ACTING ━━━━━━━━━━━━━━━━━━━━━━━━━━━
After the report and harvest, ask:
  "Should I fix deviations, add the harvested patterns to the CLAUDE.md,
  both, or neither?"

Do not make any changes until the user responds.
