# Week of 2026-01-12

- **Sentiment:** 4/5 _(1=rough week, 3=normal, 5=great week)_
- **Primary Focus:** User Dashboard Redesign — Activity Feed Component
- **Scope & Level Check:**
  - Team: All authored PRs in acme-org/web-app (Frontend team)
  - Multi-Team: Reviewed analytics schema PR in acme-org/data-pipeline repo (cross-team contribution)
  - Developer Tooling: Introduced AI coding assistant config and safety guardrails with potential multi-team applicability

---

## 1. Deliveries & Strategic Impact

- **Project:** User Dashboard Redesign -- _Business Impact_
  - **PR: [frontend] add ActivityFeed with events grouped by category to Dashboard** ([#287](https://github.com/acme-org/web-app/pull/287))
    - **What:** Added the ActivityFeed component to Dashboard pages in both desktop and mobile views. Events are grouped by category (e.g., Social, Billing, Settings) when the dashboard-v2 feature flag is enabled. Includes new `groupEventsByCategory` and `transformActivityTimeline` utilities, `useActivityFeedCopy` hook, and unit tests. +702/-163 across 16 files.
    - **Why / Business Impact:** Gives users a consolidated view of recent activity before navigating deeper, supporting the dashboard redesign rollout for the consumer product. Directly tied to #287 and #291.
    - **Level Evidence:**
      - Current Level: Owned end-to-end feature implementation spanning both desktop and mobile surfaces, including component architecture decisions (shared component location), utility design, and test coverage.
      - Target Level: N/A
  - **PR: [frontend] remove duplicate dashboard header components** ([#291](https://github.com/acme-org/web-app/pull/291))
    - **What:** Removed duplicate "Welcome back" headings from Dashboard screens. Cleaned up redundant heading components in Dashboard.tsx and DashboardSummary.tsx. +11/-58 across 2 files.
    - **Why / Business Impact:** Fixes a UI inconsistency on the dashboard, improving the experience for users landing on their home page.
    - **Level Evidence:**
      - Current Level: Identified and resolved UI polish issue as part of broader dashboard redesign work.
      - Target Level: N/A
  - **Scope:** Team (Frontend)
  - **Related Issues:** [#287](https://github.com/acme-org/web-app/issues/287), [#291](https://github.com/acme-org/web-app/issues/291)

- **Project:** Code Health & Shared Hooks -- _Engineering Excellence_
  - **PR: [frontend] delete unused DebugOverlay component** ([#302](https://github.com/acme-org/web-app/pull/302))
    - **What:** Removed the unused DebugOverlay component and its import from DevToolsPanel. +0/-74 across 2 files.
    - **Why / Business Impact:** Dead code removal reduces cognitive overhead and bundle size. Identified during code review on a related PR.
    - **Level Evidence:**
      - Current Level: Proactively cleaned up dead code spotted during review.
      - Target Level: N/A
  - **Scope:** Team (Frontend)
  - **Related Issues:** [#298](https://github.com/acme-org/web-app/issues/298) (migrate useDeleteSession hook to shared — closed this week)

- **Project:** AI Developer Tooling -- _Engineering Excellence_
  - **PR: [frontend] add CLAUDE.md and AGENTS.md for AI coding assistants** ([#305](https://github.com/acme-org/web-app/pull/305))
    - **What:** Added CLAUDE.md with project context (dev commands, code quality guidelines, architecture, tech stack, key patterns) and AGENTS.md for AI coding assistants working in web-app. +101/-0 across 2 files.
    - **Why / Business Impact:** Improves developer productivity by providing structured context for AI-assisted coding in the monorepo. Reduces onboarding friction for AI tools.
    - **Level Evidence:**
      - Current Level: Introduced new developer tooling practice for the team.
      - Target Level: Potential multi-team impact if adopted as a pattern across web-app teams. _(flag for review)_
  - **PR: [frontend] add git-guardrails-claude-code agent skill** ([#303](https://github.com/acme-org/web-app/pull/303))
    - **What:** Added git-guardrails-claude-code agent skill with PreToolUse hook and block-dangerous-git.sh script to prevent dangerous git commands from AI agents. Also added write-a-skill skill. +118/-0 across 4 files.
    - **Why / Business Impact:** Safety guardrail preventing AI coding assistants from executing destructive git operations (force push, reset --hard, etc.) in the repository.
    - **Level Evidence:**
      - Current Level: Proactive risk mitigation for AI tooling adoption.
      - Target Level: Reusable guardrail pattern applicable across teams using AI coding assistants. _(flag for review)_
  - **PR: [frontend] add CLAUDE.local.md to gitignore** ([#280](https://github.com/acme-org/web-app/pull/280))
    - **What:** Added CLAUDE.local.md to .gitignore. +3/-0 across 1 file.
    - **Why / Business Impact:** Prevents local AI assistant configuration from being committed to the repository.
    - **Level Evidence:**
      - Current Level: Housekeeping for AI tooling setup.
      - Target Level: N/A
  - **Scope:** Team (Frontend), with potential multi-team applicability

---

## 2. Code Reviews Given

| PR                                                                                                                      | Author | Repo                      | Type     | Feedback? |
| ----------------------------------------------------------------------------------------------------------------------- | ------ | ------------------------- | -------- | --------- |
| [Add event schema v2 for unified tracking](https://github.com/acme-org/data-pipeline/pull/118) | asmith | acme-org/data-pipeline | APPROVED | No        |

---

## 3. Documents & RFCs

- Frontend Ways of Working -- [Google Doc](https://docs.google.com/document/d/1ExampleDocumentIdReplaced00000000000000000/edit?usp=drivesdk) -- Last modified: 2026-01-16

---

## Weekly Metrics

| Metric                      | Count     |
| --------------------------- | --------- |
| PRs authored                | 6         |
| PRs merged                  | 4         |
| Lines +/-                   | +935/-295 |
| Reviews given (others' PRs) | 1         |
| Issues                      | 3         |
