# Bootstrap Playbook: Golden Prompts

Five prompts that cover the full mockup lifecycle.
Prompts 1 and 2 are run once per project (ideally in the same session).
Prompts 3a and 3b are used together for every new design exploration —
3a produces a written plan and stops for review; 3b builds once you approve.
Prompt 4 is a fidelity audit to run after an iterative design session
has settled — before sharing with stakeholders or handing off to engineers.


| Prompt             | When to use                                               | Produces                                                              | Creates files?    |
| ------------------ | --------------------------------------------------------- | --------------------------------------------------------------------- | ----------------- |
| 1 — Analysis       | Start of a new project                                    | Written codebase summary + decisions checklist                        | No                |
| 2 — Bootstrap      | Immediately after Prompt 1 (or standalone)                | MockChrome, registry, routing, bypass, CLAUDE.md, example mock        | Yes               |
| 3a — Plan          | Every new design exploration                              | Requirements summary + proposed layout — stops and waits for approval | No                |
| 3b — Build         | After approving the 3a plan                               | A working mock at a dev URL                                           | Yes               |
| 4 — Fidelity Audit | After design iteration settles; before sharing or handoff | Structured deviation report; fixes on confirmation                    | Only if confirmed |


---

## Prompt 1 — Codebase Analysis

> **Read-only · no files created**

Run this at the start of a fresh session on any new project.
It forces the AI to discover and document all the facts needed before
touching files — component library, app shell, routing, auth bypass
strategy, and design tokens. Review the output before running Prompt 2.

```
You are a Senior UX Engineer helping set up a design mockup environment for this project.

Before creating any files, check whether a project-context.md file exists at the project
root (or anywhere in the repo). If it does, read it first — it may already contain
accurate answers to many of the questions below. Then verify or supplement that
information with your own codebase analysis.

Analyze the codebase and answer the following questions.
Be specific — include actual file paths, component names, and code snippets where useful.

1. COMPONENT LIBRARY
   - What UI library is used (MUI, Ant Design, Radix, Chakra, shadcn, etc.)?
   - How is theming configured? Find the theme provider and any custom token extensions.
   - What is the primary import pattern (e.g. import { Button } from '@mui/material')?

2. APP SHELL
   - What components make up the outer chrome: top bar/header, main navigation
     (rail, sidebar, or tabs), and the content area?
   - What are the exact file paths for each shell component?
   - What do they depend on (Redux state, context providers, hooks)?
   - What would need to be replaced with static values to use them as
     standalone presentation components?

3. ROUTING
   - What router library is used (React Router, TanStack Router, Next.js, etc.)?
   - Where are routes defined? Show the relevant file and the pattern for adding
     a new route.
   - Are there protected/private route wrappers? What do they check?
   - Where could dev-only routes be added without affecting production builds?

4. BACKEND / AUTH BYPASS
   - What gates access to the app (session check, auth guard, gateway connection)?
   - What is the cleanest way to bypass this for a dev-only mock environment?
     Prefer an environment variable approach.
   - What environment variable system does the project use (Vite import.meta.env,
     CRA process.env.REACT_APP_*, Next.js, etc.)?

5. DESIGN TOKENS
   - Beyond standard library colors, are there custom theme extensions
     (e.g. theme.color.xxx, theme.palette.custom)?
   - Are there any utility functions for accessing tokens (e.g. a get() helper)?

6. EXISTING PATTERNS
   - Find 2–3 representative, well-structured page components and note any
     patterns I should match in mockups.
   - Are there any existing prototype or sandbox pages? If so, read them —
     they are the most useful reference.

Produce a structured summary organized by the 6 areas above. End with:
- "Proposed file locations": where should MockChrome, the mock registry,
  and individual mock files live?
- "Decisions needed": any choices the user should confirm before you start
  creating files (naming conventions, route prefix, etc.).

Do NOT create any files yet.
```

---

## Prompt 2 — Bootstrap

> **Creates 5 artifacts**

Run this immediately after Prompt 1 in the same session (so the AI
carries the analysis forward), or paste it alone into a new session
— it will re-analyze before acting. The output is a fully wired
mockup environment ready for Prompt 3.

**What gets created:**

- `MockChrome.tsx` — static app shell replica
- `.env` bypass — skip auth in dev
- `mocks.registry.ts` — central mock list
- `DevMocksIndex.tsx` + routes — browsable index
- `DesignSystemMocks/CLAUDE.md` — mock conventions
- `ExampleMock.tsx` — working validation mock

**If running in a new session:**
The prompt opens with "using the analysis above or re-analyzing if starting fresh" — so it's safe to run standalone. The AI will re-read the codebase before creating files. Expect it to take a few more turns than when running right after Prompt 1.

> **Skill candidate:** Prompts 1 and 2 are a good candidate to convert to a Claude Code skill (`.claude/commands/bootstrap-mocks.md`). A skill would let you invoke it as `/bootstrap-mocks` instead of copy-pasting both prompts, and can check first whether the infrastructure already exists (MockChrome, registry, mocks CLAUDE.md) — reporting file paths and staleness signals instead of blindly redoing Prompt 1's analysis, which is the most expensive step in the whole lifecycle. The skill should keep the analysis → confirm → create gate between Prompt 1 and Prompt 2 intact rather than collapsing it into one unreviewed pass, and should run both steps in the same session per the Session Hygiene guidance below — splitting them across sessions means Prompt 2 re-analyzes from scratch.

```
Using the codebase analysis above (or re-analyzing the codebase if starting fresh),
create the design mockup infrastructure for this project.

If a project-context.md file exists at the project root, read it before creating any files.
Follow all conventions documented there in every file you create — especially mandatory file
headers (copyright notices), import path patterns, and documented anti-patterns.

Create all five artifacts in this order:

━━ 1. MOCK CHROME ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Create a shared file (e.g. src/pages/DesignSystemMocks/shared/MockChrome.tsx) that
replicates the app shell as closely as possible using only static/local state —
no Redux, no API calls, no context providers from the real app.

Export named components with a composable API. At minimum:
- MockPageShell — full-viewport layout (header + nav + content area).
- A secondary nav component if the app has one (side panel, secondary rail).
- A tab strip component if the app uses tab-based content areas.

Read the real shell components to get exact dimensions, colors, icons, and layout
right. Use hardcoded values for username, product name, and nav items.

━━ 2. BACKEND BYPASS ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Add a dev-only bypass to skip the auth/session check:
- Add an environment variable check to the guard component. When set,
  skip the real check and render children directly.
- Create .env.development.local.example documenting the variable name and value.
- Do NOT commit the actual .env.development.local — verify .gitignore covers it.
- Apply the bypass only to the guard, not globally.

━━ 3. MOCKS REGISTRY AND ROUTING ━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Create a centralized registry and automatic routing:
- mocks.registry.ts — a typed array where each entry has: slug (URL-safe string),
  label, description, and a lazily-loaded component. Adding a new mock should
  only require editing this file.
- DevMocksIndex.tsx — a browsable page listing all mocks as clickable links
  with label and description.
- Wire up routes in the app router: a base route (e.g. /dev/design-system-mocks)
  and dynamic child routes (/dev/design-system-mocks/:slug). Wrap in a condition
  that excludes them from production builds.

━━ 4. CLAUDE.MD ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Create src/pages/DesignSystemMocks/CLAUDE.md
(Claude Code automatically loads CLAUDE.md files in the directory — no configuration needed)

Document:
- The 2-step process for adding a new mock, with a code snippet showing the
  registry entry format.
- The MockChrome API with a short usage example for each exported component.
- A directory tree showing the file layout.
- The bypass environment variable name and how to set it.
- Any project-specific design token patterns (custom token access, import paths).
- A Visual Fidelity section (see below).

━━ 4a. VISUAL FIDELITY SECTION ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Every generated CLAUDE.md must include a Visual Fidelity section containing
two parts: the design sensibility principles (copy verbatim below, adapting
only if the project's component library or design language is materially
different) and a short set of concrete patterns specific to this project
(derived by reading the component library and any existing mock files).

Include the following design sensibility principles verbatim:

  1. The goal is clarity, not visual interest.
     The UI's job is to surface information efficiently and get out of the way.
     Visual design choices should be evaluated against whether they make
     information easier to find and understand — not against whether they make
     the page feel more polished or dynamic.

  2. Visual treatments should earn their place by improving clarity.
     Borders, background fills, size variations, and other visual treatments
     are appropriate when they help the user understand structure —
     establishing grouping, separating distinct regions, or indicating
     interactivity. They are not appropriate as decoration. If you cannot
     articulate what a visual treatment helps the user find or understand,
     leave it out.

  3. Default to restraint. Flag ideas separately.
     When uncertain about a visual choice — whether to add an icon, a colored
     label, a section divider, a box — omit it. If you see a genuine
     opportunity to improve clarity through a new pattern, call it out
     explicitly and let the user decide. Do not build speculative ideas into
     the implementation.

  4. Color signals status, not structure.
     Color appears in this UI to communicate state: success, error, warning,
     active/inactive. It is not used to differentiate sections, add emphasis
     to labels, or add visual dynamism. A section header that "needs" color
     to stand out is a layout problem, not a color problem.

  5. In body content, typography hierarchy comes from weight, not size.
     For form fields, table cells, panel sections, and dialog content:
     distinguish a label from its value with fontWeight: 600, not a different
     font size. Resist reaching for smaller or larger sizes to create variation
     within the body. Page-level and structural section headers are a separate
     context and naturally use larger type scales.

  6. When establishing a new pattern, iteration is the process.
     Restraint and reference-first thinking apply to visual decoration —
     colors, borders, typography choices — where an established pattern
     already exists and should be followed. When the task requires a new
     interaction model, dialog flow, or component that has no equivalent in
     the codebase, exploration is appropriate and valuable. Propose the
     simplest version that could work, build it, and let iteration surface
     the right answer. Converging too early on a new pattern produces
     something adequate but not considered. The goal of the mockup process
     is partly to discover what works — not just to execute what's already known.

Then add a Concrete Patterns subsection by reading the project's component
library and any existing mock or production components. Document the cases
most likely to drift — chip/badge usage, section header conventions,
icon usage, font family overrides — with code examples where helpful.
Keep this list short (6–8 items). It should grow only via a pattern harvest
after a successful mock session, not speculatively.

━━ 5. FIRST EXAMPLE MOCK ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Create ExampleMock.tsx in the DesignSystemMocks directory:
- Uses MockChrome for the shell.
- Displays a heading, short paragraph, and a small table or card grid using
  the real component library — enough to verify that shell, theming, and
  typography all look correct.
- Register it in mocks.registry.ts with slug "example".

━━ WHEN DONE ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Report:
1. The dev URL to open the mock index.
2. Any manual steps required (creating the .env file, npm install, etc.).
3. Any assumptions you made that should be verified.
```

---

## Prompts 3a + 3b — New Mock

> **Two steps · always used together**

Every new design exploration uses both prompts in sequence.
3a produces a written plan and explicitly stops — giving you the
chance to read, react, and redirect before a line of code is written.
3b is the build instruction sent after you approve (or amend) the plan.
Keeping them separate is what makes the review step real rather than
just a preamble the AI races past.

### Prompt 3a — Plan

Fill in the bracketed placeholders and paste. Attach screenshots
of the existing UI when replacing something — that context
consistently produces the best fidelity.

**Placeholders to fill in:**


| Placeholder                   | What to put here                                                              |
| ----------------------------- | ----------------------------------------------------------------------------- |
| `[FEATURE NAME]`              | Short name, e.g. "Gateway Audit Events"                                       |
| `[1–2 sentences…]`            | What the feature does and who the primary user is                             |
| `[URL to documentation]`      | Link to product docs, Confluence, etc. — or "none"                            |
| `[Attach screenshots…]`       | Upload images of the current UI in your IDE, or describe it in words          |
| Key interactions to show      | List the specific behaviors that must be demoable — be specific               |
| Representational interactions | Controls that should look real but not need to work (saves, exports, dialogs) |


> **A note for designers who already have a solution in mind.**
> Having a design solution before you start is normal and good — that's your job.
> The distinction that matters is *how* you share it. Framing your thinking as
> goals and constraints ("the details should feel contextual, not like a separate page")
> lets the AI propose an approach you can react to in the plan step. Prescribing
> implementation details upfront causes it to skip the design reasoning and go straight
> to code — bypassing the review moment entirely.
>
> **The practical rule:** put your solution thinking in the Key Interactions
> field as a preference with a reason — "I'd like to explore a side panel here because
> it keeps the list context visible while viewing details." If the plan comes back
> proposing something different, that's the right moment to redirect — before anything
> is built. If it matches your thinking, you've validated your approach with an
> independent read of the problem.

```
I want to create a new design mockup for [FEATURE NAME].

Context:

- What it is:
  [1–2 sentences describing the feature and who uses it]

- Docs:
  [URL to documentation, or "none"]

- Existing UI:
  [Attach screenshots if this replaces existing UI, or describe briefly]

- Key interactions to show:
  [e.g. "clicking a row opens a detail side panel"]
  [e.g. "filters narrow the list in real time"]
  [e.g. "histogram updates when a different time range is selected"]

- Interactions that can be representational (not functional):
  [e.g. "save and export buttons", "settings dialogs"]

Follow this process:

1. UNDERSTAND FIRST
   Read any documentation or screenshots provided. If this replaces existing UI,
   read the referenced existing components to understand the current implementation
   before designing the new version.

   Also read project-context.md if it exists in the project root. It documents
   project-wide conventions (mandatory file headers, TypeScript rules, anti-patterns,
   component patterns) that apply to mock files just as much as production code.
   Key things to carry forward: copyright header on every file, no hardcoded hex
   colors, no !important overrides on MUI variants, and any documented anti-patterns.

2. REQUIREMENTS SUMMARY
   Produce a written plan covering:
   - What the current experience does (if replacing existing UI).
   - 2–3 UX improvement opportunities given the patterns already in this codebase.
   - Proposed layout and key components (describe the structure you plan to build).
   - Which interactions you will make functional vs. representational, and why.
   - Any assumptions or decisions that are hard to reverse once building starts.

   Do NOT write any code yet. Stop after the requirements summary and wait.
   Ask: "Does this plan look right, or would you like to adjust anything before I build?"
```

### Prompt 3b — Build

Send this after reviewing the 3a plan. If you want adjustments,
fill in the optional line — otherwise delete it and send as-is.
Everything the AI needs is already in the session from 3a, so
this prompt is intentionally short.

```
Proceed with building the mock as planned.
[Optional: list any adjustments to the plan here, or delete this line]

As you build:
- Use the DesignSystemMocks infrastructure (MockChrome, mocks.registry.ts).
- Check the CLAUDE.md and project-context.md for conventions.
- All data should be static and inline — no API calls, no network requests.
- Register the mock in mocks.registry.ts.

Report the dev URL when done.
```

---

## Prompt 4 — Fidelity Audit

> **Report-first · fixes on confirmation**

Run this after an iterative design session has settled — before sharing
with stakeholders or handing off to engineers. It reads sources of truth
(project-context.md, the CLAUDE.md, real production components) and
produces a structured deviation report, then waits for your sign-off
before touching anything.

**Why the report-first gate matters:**
Iterative sessions accumulate drift in predictable ways: quick fixes that
bypass proper patterns, new components copied from other mocks that
themselves had drift, and "I'll fix this later" items that don't get fixed.
The audit's value is the triage, not just the mechanical fixes — the gate
lets you decide which deviations matter for your use case (e.g. intentional
terminal colors are not a deviation; a hardcoded hex on a status chip is).

**Placeholder to fill in:**
Replace `[MOCK FILE(S)]` with the filename(s) to audit, e.g. "UserSettingsMock.tsx"
or "UserSettingsMock.tsx and DashboardMock.tsx". Auditing
one file per session produces more focused findings than batching many files at once.

**When to run it:**

- ✓ Before sharing a mock with stakeholders
- ✓ Before engineering handoff
- ✓ After a long iterative session (3+ back-and-forth turns)
- ✓ When a mock has been used as copy-paste source for a new one
- ✗ Not needed on brand-new mocks (run after first iteration round instead)

> **Skill candidate:** Prompt 4 is a good candidate to convert to a Claude Code skill (`.claude/commands/audit-mock.md`). A skill would let you invoke it as `/audit-mock UserSettingsMock.tsx` instead of copy-pasting. The skill file would contain this prompt verbatim with `$ARGUMENTS` substituted for the filename placeholder. Session hygiene (run in a fresh session) would remain a manual step.

```
Perform a fidelity audit on [MOCK FILE(S) — e.g. UserSettingsMock.tsx].

Do NOT make any changes yet. Complete steps 1–4 in full, then wait for confirmation.

━━ STEP 1 — READ SOURCES OF TRUTH ━━━━━━━━━━━━━━━━━━━━━━━━━━━
Read these in order before examining the mock:

1. project-context.md — project-wide code quality rules (copyright headers,
   TypeScript configuration, import conventions, anti-patterns).
2. DesignSystemMocks/CLAUDE.md — mock-specific conventions
   (token patterns, code quality rules for mocks, reusable components).
3. The real shell components MockChrome.tsx was modelled from. Compare
   MockChrome against them — shell drift affects every mock.
4. Any real production components the mock imitates. Identify these by
   reading the mock first, then locating the originals in the codebase
   (e.g. if the mock has a pagination footer, find CustomPagination.tsx
   and compare them side by side).

━━ STEP 2 — EXAMINE THE MOCK ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Read [MOCK FILE(S)] carefully. For each UI pattern, control, or component
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

This captures patterns that were applied correctly during the build
session — whether they were planned from the start or arrived at through
iteration — so they are available to the AI in future sessions.

If there are no undocumented patterns worth adding, state that explicitly.

━━ STEP 5 — CONFIRM BEFORE ACTING ━━━━━━━━━━━━━━━━━━━━━━━━━━━
After the report and harvest, ask:
  "Should I fix deviations, add the harvested patterns to the CLAUDE.md,
  both, or neither?"

Do not make any changes until the user responds.
```

---

## Tips and Edge Cases

#### If Prompt 2 fails on MockChrome

The app shell is the hardest part to get right on the first pass.
If it looks wrong, paste the real shell component file and ask the
AI to compare its mock against it. One targeted correction pass
usually resolves visual mismatches.

#### If the bypass doesn't work

Ask the AI to read the actual route guard or auth provider and
trace exactly where the check happens. Some apps have multiple
layers (route guard + context provider + API call). Each needs
its own bypass check.

#### If routing is Next.js App Router

The registry pattern still works but routes are directory-based.
The AI should create `app/dev/design-system-mocks/[slug]/page.tsx`
that reads the slug and renders the matching component from the
registry. The CLAUDE.md should document this variant.

#### Evolving MockChrome over time

When the real app shell changes (new nav items, new layout zones),
paste both the changed real component and MockChrome into a session
and ask the AI to reconcile them. Keep MockChrome changes small and
backward-compatible so existing mocks don't break.

#### If the project has a project-context.md

Some projects maintain a `project-context.md` (or `AGENTS.md`) at the root
that documents the full tech stack, coding conventions, and anti-patterns
for AI agents. If it exists, it supersedes Prompt 1 for fact-finding and
should be referenced in every Prompt 3 session. It typically covers
mandatory file headers, import path rules, forbidden patterns, and
TypeScript configuration — all of which apply equally to mock files.

> **Keeping the CLAUDE.md current is the highest-leverage maintenance task.**
> After each Bootstrap session, review the generated file and refine it.
> Every future Prompt 3 session reads it automatically — accurate conventions
> in the file mean less back-and-forth per exploration.

---

## Session Hygiene

How you manage session boundaries matters as much as the prompts themselves.

### The builder's bias problem

A model that has been iterating on a design for many turns develops momentum.
It remembers its own earlier decisions and is reluctant to question them,
has internal consistency pressure (it wants the things it built to be correct),
and applies each change incrementally — so it never sees accumulated drift the
way a fresh perspective would. It will rationalize a deviation rather than flag
it, because it made that decision two turns ago. This is why Prompt 4 run in a
fresh session is qualitatively different from Prompt 4 run at the end of the
session that built the mock.

### One job per session


| Job                   | Session                             | End condition                                |
| --------------------- | ----------------------------------- | -------------------------------------------- |
| Codebase analysis     | Session 1                           | Summary reviewed, decisions confirmed        |
| Bootstrap environment | Session 2 (or same as Analysis)     | Dev URL working, example mock loads          |
| Design + build Mock X | Session 3                           | Design has settled, ready to share           |
| Audit Mock X          | Session 4 — fresh                   | Deviations fixed and confirmed               |
| Design + build Mock Y | Session 5 — independent from Mock X | Never share a session across unrelated mocks |


### Signals that a session has run too long

**Coherence signals:**

- The model contradicts a decision it made earlier in the same session
- A fix introduces a regression in something that worked before
- The model "remembers" things that weren't explicitly stated
- The session summary is very long — the model is now working from summaries of summaries

**Task signals:**

- You're switching between fundamentally different task types (building → auditing)
- You're starting a new deliverable (Mock A is done, starting Mock B)
- The design has "settled" and the next step is review, sharing, or handoff
- You're stuck and considering a different approach — start fresh instead of pivoting mid-session

### When to stay in the same session

**Stay together: Analysis → Bootstrap**
Prompts 1 and 2 are explicitly designed to run in sequence. The codebase analysis context
carries forward directly into Bootstrap — splitting them means Prompt 2 re-analyzes from
scratch, which is slower and may produce different conclusions.

**Stay together: Design iterations**
Build → feedback → iterate is one coherent job. Breaking it up loses the "why we tried X
and moved to Y" context that prevents the model from re-suggesting rejected approaches.
Stay in the session until the design has genuinely settled.

**Stay together: Small targeted fixes**
Renaming a label, removing a background color, tweaking spacing — no contextual depth
needed, no reason to start fresh. These are safe to chain in a session without
coherence risk.

**Stay together: Hard-won context**
If the model just read 15 reference files to understand a complex pattern, don't throw
that away unless you're switching jobs. The cost of re-reading outweighs the benefit of
a fresh session for the next iteration of the same task.

> **End sessions at natural breakpoints, not when you're stuck.**
> If a design is "good enough to share," that is the end of the build session —
> not when you run into a hard problem. Getting stuck mid-session is a signal
> to wrap up cleanly and approach the problem differently in a fresh session,
> not to keep pushing until something breaks.

