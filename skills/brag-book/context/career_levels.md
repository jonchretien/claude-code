# Career Level Evidence Framework

Customize the levels below to match your organization's career ladder (e.g., L5/L6/L7, IC3/IC4/IC5, Mid/Senior/Staff, or any other leveling system). Add or remove levels as needed — the skill will produce one evidence entry per level defined here.

---

## Level 1: Current Level

**Focus:** Team-scope ownership, technical decisions, quality improvements.

**Signals:**
- End-to-end feature ownership within a team
- Technical design decisions (component architecture, API design, data modeling)
- Quality improvements (test coverage, dead code removal, error handling)
- Proactive identification and resolution of tech debt
- Code review quality (catching bugs, suggesting improvements)
- On-call and incident response within team scope

**Example evidence language:**
- "Owned end-to-end implementation spanning both desktop and mobile surfaces"
- "Made architectural decision to use shared component location for reuse"
- "Proactively cleaned up dead code spotted during review"
- "Introduced testing patterns for new component category"

---

## Level 2: Target Level

**Focus:** Cross-team or cross-org influence, architectural patterns, mentoring, RFC authorship.

**Signals:**
- PRs or reviews across multiple repos
- RFC authorship or significant contribution to design docs
- Shared library or SDK contributions
- Organized or led cross-team meetings (design reviews, guild meetings)
- Established patterns or tooling adopted beyond own team
- Mentoring signals (pairing sessions, onboarding docs, knowledge sharing)
- Incident leadership for cross-team issues
- Mentioned in channels outside own team (cross-team visibility)

**Example evidence language:**
- "Authored RFC for cross-service authentication pattern"
- "Created shared library adopted by 3 teams"
- "Led cross-team design review with 12 attendees from 4 teams"
- "Established guardrail pattern applicable across teams using AI coding assistants"

**Use N/A for a level if no relevant signals are present.**

---

## Scope Indicators

| Scope | Definition |
|-------|-----------|
| **Team** | All PRs in same repo/project area; work contained within one team |
| **Multi-Team** | PRs across multiple repos; organized cross-team meetings; mentioned in channels outside own team |
| **Guild/Org** | RFCs, shared libraries, cross-org PRs, design docs, ran design reviews or guild meetings |

**Calendar signals for scope:**
- Organized cross-team meetings → Multi-Team
- Ran design reviews or guild meetings → Guild/Org

**Slack signals for scope:**
- Mentioned in channels outside own team → Multi-Team or higher

List all applicable scopes when multiple apply.
