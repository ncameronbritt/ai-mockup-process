Ask the user for the context needed to plan this mock, one question at a time as plain conversational messages — wait for their answer before asking the next. Do not use a structured question/choice tool for these; they need to be free-form chat replies so screenshots can be attached inline where relevant.

Ask, in this order:

1. **Feature name** — short name, e.g. "Gateway Audit Events"
2. **What it is** — 1–2 sentences describing the feature and who uses it
3. **Docs** — URL to documentation, or "none"
4. **Existing UI** — brief description of what the current experience looks like, if this replaces something (or "none"). If it replaces existing UI, ask them to attach screenshots along with this answer.
5. **Key interactions to show** — the specific behaviors that must be demoable; be specific
6. **Representational interactions** — controls that should look real but not need to work (saves, exports, dialogs)

Once all six are answered, proceed:

1. UNDERSTAND FIRST
   Read any documentation or screenshots provided. If this replaces existing UI, read the referenced existing components to understand the current implementation before designing the new version.

   Read the project conventions file at the project root (project-context.md, AGENTS.md, or root CLAUDE.md — whichever exists) and the CLAUDE.md in the mocks directory. These document project-wide conventions that apply to mock files: mandatory file headers, TypeScript rules, anti-patterns, component patterns. Key things to carry forward: copyright header on every file, no hardcoded color values, no !important overrides on component library variants, and any documented anti-patterns.

2. REQUIREMENTS SUMMARY
   Produce a written plan covering:
   - What the current experience does (if replacing existing UI).
   - 2–3 UX improvement opportunities given the patterns already in this codebase.
   - Proposed layout and key components.
   - Which interactions will be functional vs. representational, and why.
   - Any assumptions or decisions that are hard to reverse once building starts.

   Do NOT write any code. Stop after the plan and ask:
   "Does this plan look right, or would you like to adjust anything before I build?"
