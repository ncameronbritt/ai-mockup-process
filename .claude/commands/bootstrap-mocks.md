---
name: bootstrap-mocks
description: One-time setup of design mockup infrastructure (MockChrome, registry, routing, auth bypass, CLAUDE.md, example mock). Checks whether it has already run before redoing expensive analysis. React-specific for now.
model: opus
---

Set up the design mockup infrastructure for this project, or verify what already
exists. Do NOT create or modify any files until STEP 2 has been reviewed and
explicitly confirmed.

━━ STEP 0 — CHECK FOR EXISTING BOOTSTRAP ━━━━━━━━━━━━━━━━━━━━
Before doing anything else, search the repo for signs bootstrap has already run:

- A file named `MockChrome.tsx` (or clearly equivalent) anywhere under `src/`.
- A `CLAUDE.md` living in the same directory.
- A registry file (e.g. `mocks.registry.ts`) in that directory.

If NONE of these exist, skip straight to STEP 1.

If ANY of them exist, do not re-run the analysis. Instead report:

- File paths of every artifact found.
- Last-modified date and size for each.
- For MockChrome.tsx specifically: locate the real shell components it appears
  to be modeled on (header, nav, layout) and compare modification dates against
  it. Flag explicitly if a real component has changed more recently than
  MockChrome.tsx — that is a concrete drift signal, not just a guess.

Then stop and ask:
"Mock infrastructure already exists at [path]. [Nothing looks stale / The real
shell has changed since MockChrome.tsx was last updated on DATE]. Do you want
to (a) leave it as-is, (b) reconcile MockChrome.tsx against the current real
components, or (c) do a full re-bootstrap?"

Do not proceed past this point without an answer.

━━ STEP 1 — ANALYSIS (read-only, no files created) ━━━━━━━━━
Check whether a project-context.md file exists at the project root (or
anywhere in the repo). If it does, read it first — it may already contain
accurate answers to many of the questions below. Then verify or supplement
that information with your own codebase analysis. If no project-context.md
exists, look for AGENTS.md or a root CLAUDE.md instead.

Analyze the codebase and answer the following. Be specific — include actual
file paths, component names, and code snippets where useful.

1. COMPONENT LIBRARY
   - What UI library is used (MUI, Ant Design, Radix, Chakra, shadcn, etc.)?
   - How is theming configured? Find the theme provider and any custom token
     extensions.
   - What is the primary import pattern?

2. APP SHELL
   - What components make up the outer chrome: top bar/header, main
     navigation (rail, sidebar, or tabs), and the content area?
   - Exact file paths for each shell component.
   - What do they depend on (Redux state, context providers, hooks)?
   - What would need to be replaced with static values to use them as
     standalone presentation components?

3. ROUTING
   - What router library is used?
   - Where are routes defined? Show the pattern for adding a new route.
   - Are there protected/private route wrappers? What do they check?
   - Where could dev-only routes be added without affecting production builds?

4. BACKEND / AUTH BYPASS
   - What gates access to the app (session check, auth guard, gateway
     connection)?
   - What is the cleanest way to bypass this for a dev-only mock environment?
     Prefer an environment variable approach.
   - What environment variable system does the project use?

5. DESIGN TOKENS
   - Beyond standard library colors, are there custom theme extensions
     (e.g. theme.color.xxx, theme.palette.custom)?
   - Are there utility functions for accessing tokens (e.g. a get() helper)?

6. EXISTING PATTERNS
   - Find 2–3 representative, well-structured page components and note any
     patterns to match in mockups.
   - Are there existing prototype or sandbox pages? If so, read them — they
     are the most useful reference.

This process currently assumes a React app. If the codebase is not React,
stop after this step and say so explicitly rather than improvising an
unfamiliar framework's equivalents.

End with:
- "Proposed file locations": where MockChrome, the mock registry, and
  individual mock files should live.
- "Decisions needed": naming conventions, route prefix, or anything else
  the user should confirm before files are created.

Do NOT create any files yet. Stop and ask:
"Does this analysis look right, and do the proposed decisions above work for
you? Reply to confirm before I create the infrastructure."

━━ STEP 2 — CREATE INFRASTRUCTURE ━━━━━━━━━━━━━━━━━━━━━━━━━━━
Only after STEP 1 is explicitly confirmed. If a project-context.md file
exists, follow all conventions documented there in every file created —
especially mandatory file headers (copyright notices), import path patterns,
and documented anti-patterns.

Create all five artifacts in this order:

1. MOCK CHROME
   Create a shared file (e.g. src/pages/DesignSystemMocks/shared/MockChrome.tsx)
   that replicates the app shell as closely as possible using only
   static/local state — no Redux, no API calls, no context providers from the
   real app. Export named components with a composable API: at minimum
   MockPageShell (full-viewport layout), a secondary nav component if the app
   has one, and a tab strip component if the app uses tab-based content areas.
   Read the real shell components to get exact dimensions, colors, icons, and
   layout right. Use hardcoded values for username, product name, and nav items.

2. BACKEND BYPASS
   Add a dev-only bypass to skip the auth/session check: an environment
   variable check in the guard component that, when set, skips the real check
   and renders children directly. Create .env.development.local.example
   documenting the variable. Do NOT commit the actual
   .env.development.local — verify .gitignore covers it. Apply the bypass
   only to the guard, not globally.

3. MOCKS REGISTRY AND ROUTING
   Create mocks.registry.ts — a typed array where each entry has slug, label,
   description, and a lazily-loaded component. Create DevMocksIndex.tsx — a
   browsable page listing all mocks as clickable links. Wire up routes: a
   base route and dynamic child routes, wrapped in a condition that excludes
   them from production builds.

4. CLAUDE.MD
   Create the CLAUDE.md next to MockChrome (Claude Code loads directory
   CLAUDE.md files automatically). Document: the 2-step process for adding a
   new mock with a registry-entry code snippet; the MockChrome API with a
   usage example per exported component; a directory tree; the bypass
   environment variable and how to set it; project-specific design token
   patterns; and a Visual Fidelity section.

   The Visual Fidelity section must include these six principles verbatim
   (adapt only if the component library or design language is materially
   different):

   1. The goal is clarity, not visual interest.
   2. Visual treatments should earn their place by improving clarity.
   3. Default to restraint. Flag ideas separately.
   4. Color signals status, not structure.
   5. In body content, typography hierarchy comes from weight, not size.
   6. When establishing a new pattern, iteration is the process.

   Then add a Concrete Patterns subsection (6–8 items) by reading the
   component library and any existing mock files — chip/badge usage, section
   header conventions, icon usage, font family overrides. This list should
   grow later via pattern harvests from audit-mock, not speculatively now.

5. FIRST EXAMPLE MOCK
   Create ExampleMock.tsx: uses MockChrome for the shell, displays a heading,
   short paragraph, and a small table or card grid using the real component
   library — enough to verify shell, theming, and typography all look
   correct. Register it in mocks.registry.ts with slug "example".

━━ WHEN DONE ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Report:
1. The dev URL to open the mock index.
2. Any manual steps required (creating the .env file, npm install, etc.).
3. Any assumptions made that should be verified.
