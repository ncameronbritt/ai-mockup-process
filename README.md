# AI Mockup Process

A workflow for building high-fidelity UX design mockups in a React codebase using Claude Code.

The goal is mockups that look and feel indistinguishable from the real product — same shell, same component library, same design tokens. This is achieved by having Claude read the actual production codebase rather than working from screenshots or descriptions alone. The prerequisite is a working local dev environment: Claude needs to be able to scan the code to understand the shell, theming, and component patterns, and to run the app so you can verify mocks in a browser at a real URL.

## Contents

- [What's here](#whats-here)
- [How to use this on a new project](#how-to-use-this-on-a-new-project)
- [The lifecycle](#the-lifecycle)
- [Two-layer structure](#two-layer-structure)
- [Session hygiene](#session-hygiene)
- [Tips and edge cases](#tips-and-edge-cases)
- [Sharing mocks](#sharing-mocks)
- [Adapting for other frameworks](#adapting-for-other-frameworks)

## What's here

- **`.claude/commands/bootstrap-mocks.md`** — sets up the mock environment for a project (`/bootstrap-mocks`). Checks whether it's already been run before redoing the (comparatively expensive) codebase analysis.
- **`.claude/commands/plan-mock.md`** — runs the planning step for a new mock interactively (`/plan-mock`).
- **`.claude/commands/audit-mock.md`** — runs a fidelity audit on an existing mock (`/audit-mock UserSettingsMock.tsx`).

There's no separate "build" skill — see [The lifecycle](#the-lifecycle) below for why.

## How to use this on a new project

1. **Install the skills** — copy `.claude/commands/` into your project root (or `~/.claude/commands/` to make them available in every project). Update the path references in each skill file to match your project's structure.

2. **Bootstrap the mock environment** — run `/bootstrap-mocks` in a Claude Code session. This creates the mock shell component, registry, routing, auth bypass, and a `CLAUDE.md` in the mocks directory. Safe to run again later — it detects an existing bootstrap and reports on it (including whether the shell looks stale) instead of redoing the analysis from scratch.

3. **Plan and build a mock** — run `/plan-mock` to work through requirements interactively. It stops after producing a written plan and asks whether it looks right. Once you confirm, ask Claude to proceed with the build in the same session — no separate skill is needed, since everything from planning is already in context.

   If you already have a design solution in mind, that's normal and good — the distinction that matters is *how* you share it. Put it in your answer to `/plan-mock`'s "key interactions" question as a preference with a reason ("I'd like to explore a side panel here because it keeps list context visible"), rather than prescribing implementation details upfront. Framing it as a goal or constraint lets Claude propose an approach you can react to; prescribing details causes it to skip the design reasoning and go straight to code, bypassing the review moment entirely.

4. **Audit before sharing** — run `/audit-mock YourMockFile.tsx` in a fresh session before sharing with stakeholders or handing off to engineers.

## The lifecycle

| Stage         | When to use                                               | Produces                                                              | Creates files?    | How it runs                                     |
| ------------- | ---------------------------------------------------------- | ---------------------------------------------------------------------- | ------------------ | ------------------------------------------------ |
| **Bootstrap** | Once per project                                            | Mock shell, registry, routing, auth bypass, `CLAUDE.md`, example mock  | Yes                 | `/bootstrap-mocks`                                |
| **Plan**      | Every new design exploration                                | Requirements summary + proposed layout — stops and waits for approval | No                  | `/plan-mock`                                      |
| **Build**     | Right after approving the plan                              | A working mock at a dev URL                                            | Yes                 | Informal continuation of the same `/plan-mock` session — just confirm and ask Claude to proceed |
| **Audit**     | After design iteration settles; before sharing or handoff  | Structured deviation report; fixes on confirmation                     | Only if confirmed  | `/audit-mock`                                     |

Build was deliberately never turned into its own skill: by the time a plan is approved, everything Claude needs is already in context from the `/plan-mock` conversation, and Claude naturally asks "ready to build?" at that point. A separate skill would add ceremony without adding anything a fresh invocation could do better.

## Two-layer structure

This repo contains the **general process**. Each project that uses it will have its own adapted layer:

- A `CLAUDE.md` in the mocks directory with project-specific conventions and concrete patterns (generated by `/bootstrap-mocks`, grown over time via the audit's pattern harvest step).
- Skill files with project-specific paths filled in.

When you improve the general process, update this repo. When you refine project-specific conventions, update that project's `CLAUDE.md`. Changes to the process flow from project → this repo; changes to conventions stay in the project.

## Session hygiene

How you manage session boundaries matters as much as the skills themselves.

### The builder's bias problem

A model that has been iterating on a design for many turns develops momentum. It remembers its own earlier decisions and is reluctant to question them, has internal consistency pressure (it wants the things it built to be correct), and applies each change incrementally — so it never sees accumulated drift the way a fresh perspective would. It will rationalize a deviation rather than flag it, because it made that decision two turns ago. This is why `/audit-mock` run in a fresh session is qualitatively different from running it at the end of the session that built the mock.

### One job per session

| Job                    | Session                              | End condition                                |
| ----------------------- | ------------------------------------- | --------------------------------------------- |
| Bootstrap environment   | Session 1                             | Dev URL working, example mock loads           |
| Plan + build Mock X     | Session 2                             | Design has settled, ready to share            |
| Audit Mock X            | Session 3 — fresh                     | Deviations fixed and confirmed                |
| Plan + build Mock Y     | Session 4 — independent from Mock X   | Never share a session across unrelated mocks  |

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

**Stay together: Bootstrap's analysis and creation steps**
`/bootstrap-mocks` deliberately keeps its read-only analysis and file-creation steps in one skill invocation rather than splitting them across sessions. The codebase analysis carries forward directly into creation — splitting them means re-analyzing from scratch, which is slower and may reach different conclusions.

**Stay together: Design iterations**
Build → feedback → iterate is one coherent job. Breaking it up loses the "why we tried X and moved to Y" context that prevents the model from re-suggesting rejected approaches. Stay in the session until the design has genuinely settled.

**Stay together: Small targeted fixes**
Renaming a label, removing a background color, tweaking spacing — no contextual depth needed, no reason to start fresh. These are safe to chain in a session without coherence risk.

**Stay together: Hard-won context**
If the model just read 15 reference files to understand a complex pattern, don't throw that away unless you're switching jobs. The cost of re-reading outweighs the benefit of a fresh session for the next iteration of the same task.

> **End sessions at natural breakpoints, not when you're stuck.**
> If a design is "good enough to share," that is the end of the build session — not when you run into a hard problem. Getting stuck mid-session is a signal to wrap up cleanly and approach the problem differently in a fresh session, not to keep pushing until something breaks.

## Tips and edge cases

#### If `/bootstrap-mocks` fails on the mock shell

The app shell is the hardest part to get right on the first pass. If it looks wrong, paste the real shell component file and ask the AI to compare its mock against it. One targeted correction pass usually resolves visual mismatches.

#### If the bypass doesn't work

Ask the AI to read the actual route guard or auth provider and trace exactly where the check happens. Some apps have multiple layers (route guard + context provider + API call). Each needs its own bypass check.

#### If routing is Next.js App Router

The registry pattern still works but routes are directory-based. The AI should create `app/dev/design-system-mocks/[slug]/page.tsx` that reads the slug and renders the matching component from the registry. The project's `CLAUDE.md` should document this variant.

#### Evolving MockChrome over time

When the real app shell changes (new nav items, new layout zones), paste both the changed real component and MockChrome into a session and ask the AI to reconcile them. Keep MockChrome changes small and backward-compatible so existing mocks don't break. `/bootstrap-mocks` will flag this automatically on a repeat run if the real shell components have a newer modification date than MockChrome — but the reconciliation itself is still a manual pass.

#### If the project has a project-context.md

Some projects maintain a `project-context.md` (or `AGENTS.md`) at the root that documents the full tech stack, coding conventions, and anti-patterns for AI agents. If it exists, it supersedes `/bootstrap-mocks`'s own codebase analysis for fact-finding and should be referenced in every `/plan-mock` and `/audit-mock` session. It typically covers mandatory file headers, import path rules, forbidden patterns, and TypeScript configuration — all of which apply equally to mock files.

> **Keeping the mocks-directory CLAUDE.md current is the highest-leverage maintenance task.**
> After each bootstrap, review the generated file and refine it. Every future `/plan-mock` and `/audit-mock` session reads it automatically — accurate conventions in the file mean less back-and-forth per exploration.

## Sharing mocks

Mocks live on a dedicated branch in the project repo. Sharing is as simple as pointing engineers or stakeholders to that branch and the local dev URL — no separate design tool or export step required. Engineers can read the code directly if they want implementation detail.

## Adapting for other frameworks

This process has only actually been exercised on React codebases so far. `/bootstrap-mocks` is written to discover the component library, router, and app shell from the codebase rather than hardcoding React assumptions, so it may well work on Angular, Vue, or another stack — but that's untested, not confirmed. It will stop and say so explicitly if it can't confirm the codebase is React, rather than improvising an unfamiliar framework's equivalents.

If you try this on a non-React project, treat the first run as a validation pass: review the analysis output carefully before letting it create files, and expect to correct assumptions that were implicitly React-shaped (e.g. `sx` prop patterns, MUI variant overrides in the generated `CLAUDE.md`'s Visual Fidelity section). Once a second framework has been run through successfully, this section should be updated with what actually needed to change.

Porting these skills to a different AI coding tool (not Claude Code) is a separate kind of adaptation: the plain-English instruction body in each `.claude/commands/*.md` file is portable, but tool-specific frontmatter (`model`, `allowed-tools`) and slash-command invocation are not — expect to manually rebuild those parts in whatever mechanism the other tool provides.
