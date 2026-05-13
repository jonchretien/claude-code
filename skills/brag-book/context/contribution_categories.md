# Contribution Categories for Brag Book Classification

Customize these categories to match your organization's framework (OKRs, KPIs, pillars, etc.). Add or remove categories as needed.

Use these categories to classify each contribution. Look for the **signals** in PR titles, bodies, Jira ticket descriptions, and labels to determine the best fit.

---

## 1. Engineering Excellence

**Focus:** Internal engineering health, developer productivity, and technical excellence.

**Signals:**
- Tech debt reduction, deprecation, decommissioning
- Architecture improvements, refactoring, migrations
- Test coverage additions or improvements
- CI/CD pipeline changes, build tooling
- Internal libraries, shared tooling, SDKs
- Performance optimization (latency, memory, CPU)
- Dependency upgrades, security patches
- Documentation of internal systems

**Example measurable-impact language:**
- "Reduced build time by X% by migrating to ..."
- "Eliminated N deprecated dependencies, unblocking upgrade to ..."
- "Added integration tests covering N critical paths, reducing regression rate"
- "Refactored X module from N lines to M lines, improving maintainability"
- "Migrated N services from X to Y, enabling ..."

---

## 2. Operational Excellence

**Focus:** Reliability, incident response, monitoring, and operational maturity.

**Signals:**
- Incident response, postmortems, on-call work
- Alerting and monitoring improvements
- Runbook creation or updates
- SLO/SLA definition or tracking
- Capacity planning
- Support queue tickets
- Rollback procedures, feature flag cleanup
- Disaster recovery, failover testing

**Example measurable-impact language:**
- "Resolved P1 incident affecting N users within X minutes"
- "Added monitoring for X, catching Y issue before customer impact"
- "Created runbook for X, reducing MTTR from N minutes to M minutes"
- "Resolved N support tickets, unblocking X team/customers"
- "Cleaned up N feature flags, reducing configuration complexity"

---

## 3. Business Impact

**Focus:** User-facing features, revenue impact, product launches, and business outcomes.

**Signals:**
- New feature implementation or enhancement
- A/B tests, experiments
- Revenue-impacting changes
- User experience improvements
- Product launches, GA releases
- Partner or third-party integrations
- Analytics or tracking instrumentation
- Accessibility improvements

**Example measurable-impact language:**
- "Launched X feature to N% of users, driving Y metric improvement"
- "Implemented A/B test for X, resulting in Y% lift in Z metric"
- "Integrated with X partner, enabling Y new use case"
- "Improved page load time by X%, impacting Y daily active users"
- "Added analytics tracking for X, enabling product team to measure Y"

---

## Classification Guidelines

1. **When a contribution spans multiple categories**, choose the primary intent. A performance optimization for a user-facing page is **Business Impact** (user impact), not Engineering Excellence (tech improvement).
2. **Support tickets** default to **Operational Excellence** unless they involve feature work.
3. **RFCs and design docs** inherit the category of the work they describe.
4. **Code reviews** are not categorized individually -- they roll up under the reviewer's overall contribution.
5. If unclear, mark as **Engineering Excellence** as a safe default and flag for manual review.
