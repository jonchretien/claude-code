---
name: brag-book
description: "Generate a weekly brag book entry that captures GitHub PRs, Google Drive docs, Google Calendar meetings, and optionally Jira tickets and Slack mentions, then analyzes them for business impact and career-level evidence based on your configured career ladder. Use whenever the user wants to review their weekly accomplishments, track contributions for career growth, summarize what they shipped this week, or prepare evidence for promotion. Trigger on: \"brag book\", \"update brag book\", \"weekly brag\", \"what did I do this week\", \"weekly accomplishments\", \"track my contributions\", \"weekly summary of my work\", \"what did I ship this week\"."
user_invocable: true
trigger_phrases:
  - "update brag book"
  - "brag book"
  - "weekly brag"
  - "brag book entry"
  - "update my brag book"
  - "what did I do this week"
  - "weekly accomplishments"
  - "track my contributions"
  - "weekly summary of my work"
  - "what did I ship this week"
---

# Update Brag Book

Generate a weekly brag book entry by exporting contributions from GitHub, Google Drive, Google Calendar, and optionally Jira and Slack, then analyzing for impact and career-level evidence.

This skill generates files to disk — every step writes to `{output_dir}/data/`, and the final output is a Markdown file. Verify each file exists before moving on.

---

## Required Parameters

| Parameter | Required | Default | Example |
|-----------|----------|---------|---------|
| **username** | YES | - | `jdoe` |
| **email** | YES | - | `jdoe@example.com` |
| **output_dir** | NO | `~/bragbook-output/` | `~/Library/CloudStorage/GoogleDrive-{email}/My Drive/My Brag Book/` |
| **week_start_date** | NO | Monday of current week | `2026-02-09` |
| **week_end_date** | NO | Today | `2026-02-13` |

**If username or email is missing, use AskUserQuestion to get them. Do not proceed without both.**

**Date defaults:** Calculate Monday of the current week as `week_start_date` and today as `week_end_date` unless the user specifies otherwise.

---

## Prerequisites

- **GitHub CLI**: `gh` CLI installed and authenticated (`gh auth status`)
- **Atlassian MCP** (optional): Atlassian MCP server connected (provides Jira access via `mcp__atlassian-mcp__*` tools) -- non-blocking if unavailable
- **Google Drive MCP** (optional): Google Drive MCP server connected (provides `mcp__google-drive__*` tools) -- non-blocking if unavailable
- **Google Calendar MCP** (optional): Google Calendar MCP server connected (provides `mcp__google-calendar-mcp__*` tools) -- non-blocking if unavailable
- **Slack MCP** (optional): Slack MCP server connected (provides `mcp__claude_ai_Slack__*` tools for Slack search) -- non-blocking if unavailable

---

## Phase 1: Setup (Sequential)

### STEP 0: Verify GitHub Authentication

```bash
gh auth status
```

**IF AUTH FAILS:** Run `gh auth login --git-protocol https --web` and retry.

**This step is BLOCKING.** Do not proceed until authentication succeeds.

---

### STEP 1: Create Output Directory

```bash
mkdir -p {output_dir}/data
```

**VERIFY:** Directory `{output_dir}/data` exists before proceeding.

---

## Phase 2: Data Collection (Parallel)

Steps 2-7 gather data from independent sources with no dependencies between them. Execute all of them in parallel to minimize total latency.

### STEP 2: Export GitHub Contributions

Collect PRs authored, diff stats, and reviews given by running `gh` CLI commands, then process the output and write CSV files.

#### 2a. Fetch authored PRs (REST search)

```bash
gh search prs --author={username} --created=">={week_start_date}" --json number,title,state,createdAt,closedAt,repository,url --limit 500
```

Capture the JSON output. This returns all authored PRs (open, closed, merged) in the date range.

#### 2b. Fetch diff stats and PR details (GraphQL)

Run this GraphQL query to get additions, deletions, changed files, body text, and labels for all authored PRs:

```bash
gh api graphql -f query='
query {
  search(query: "author:{username} is:pr created:{week_start_date}..{week_end_date}", type: ISSUE, first: 100) {
    edges {
      node {
        ... on PullRequest {
          number
          title
          body
          url
          state
          createdAt
          mergedAt
          additions
          deletions
          changedFiles
          labels(first: 10) {
            nodes {
              name
            }
          }
          repository { nameWithOwner }
        }
      }
    }
  }
}'
```

#### 2c. Fetch reviews given (GraphQL)

Run this GraphQL query to get reviews the user has given on other people's PRs:

```bash
gh api graphql -f query='
query {
  search(query: "reviewed-by:{username} is:pr created:>={week_start_date}", type: ISSUE, first: 100) {
    edges {
      node {
        ... on PullRequest {
          number
          title
          url
          author { login }
          repository { nameWithOwner }
          createdAt
          mergedAt
          state
          reviews(first: 50) {
            nodes {
              author { login }
              state
              submittedAt
              body
              comments(first: 10) {
                totalCount
              }
            }
          }
        }
      }
    }
  }
}'
```

#### 2d. Process and write `github_authored.csv`

Join the REST search results (step 2a) with the GraphQL stats (step 2b) by PR number. For each authored PR, produce a row with these columns:

`pr_number,title,repository,state,created_at,merged_at,additions,deletions,files_changed,url`

Processing rules:
- `state`: lowercase (`open`, `closed`, or `merged`)
- `created_at`: first 10 chars of `createdAt` (YYYY-MM-DD)
- `merged_at`: first 10 chars of `mergedAt` if the PR is merged, otherwise empty string
- `additions`, `deletions`, `files_changed`: from GraphQL stats; empty string if PR not found in GraphQL results
- `repository`: the `nameWithOwner` value (e.g., `org/repo`)
- Sort rows by `created_at` descending

Write the CSV to `{output_dir}/data/github_authored.csv`.

#### 2e. Process and write `github_reviews.csv`

From the reviews GraphQL response (step 2c), extract only reviews where `review.author.login` equals `{username}`. For each such review, produce a row with these columns:

`pr_number,title,repository,pr_author,review_type,reviewed_at,pr_state,pr_created_at,has_feedback,inline_comments,url`

Processing rules:
- `pr_author`: the PR's `author.login` (use `"unknown"` if null)
- `review_type`: the review's `state` (e.g., `APPROVED`, `CHANGES_REQUESTED`, `COMMENTED`)
- `reviewed_at`: first 10 chars of `submittedAt` (YYYY-MM-DD)
- `pr_state`: the PR's `state`
- `has_feedback`: `True` if the review has a non-empty body OR `comments.totalCount > 0`, otherwise `False`
- `inline_comments`: the review's `comments.totalCount`
- Sort rows by `reviewed_at` descending

Write the CSV to `{output_dir}/data/github_reviews.csv`.

**VERIFY:** Files `{output_dir}/data/github_authored.csv` and `{output_dir}/data/github_reviews.csv` exist.

---

### STEP 3: Write PR Details JSON

Using the GraphQL response already fetched in step 2b, build a JSON object keyed by PR number (as a string). Each value should contain:

```json
{
  "number": 123,
  "title": "PR title",
  "body": "PR body text",
  "url": "https://github.com/org/repo/pull/123",
  "state": "MERGED",
  "created_at": "2026-05-11",
  "merged_at": "2026-05-12",
  "additions": 45,
  "deletions": 12,
  "changed_files": 3,
  "labels": ["bug", "priority-high"],
  "repository": "org/repo"
}
```

Processing rules:
- `created_at`: first 10 chars of `createdAt`, empty string if null
- `merged_at`: first 10 chars of `mergedAt`, empty string if null
- `labels`: list of label name strings from `labels.nodes`
- `additions`, `deletions`, `changed_files`: integers (default 0)

Write to `{output_dir}/data/pr_details.json` with indented formatting.

**VERIFY:** File `{output_dir}/data/pr_details.json` exists and contains PR bodies.

---

### STEP 4 (Optional): Export Jira Contributions

**This step is NON-BLOCKING.** If Atlassian MCP tools are unavailable, skip this step and create a placeholder CSV.

Use the Atlassian MCP server to fetch Jira tickets directly.

1. Call `mcp__atlassian-mcp__search_issues_advanced` with:
   - `jql_query`: `"assignee = '{email}' AND updated >= '{week_start_date}'"`
   - `fields`: `"summary,description,parent,project,status,issuetype,updated"`
   - `max_results`: 50

2. If `total` in the response exceeds `maxResults`, paginate by calling again with incremented `start_at` (e.g., 50, 100, ...) until all results are fetched.

3. Extract CSV columns from the response:
   - `ticket_id`: from issue `key` (e.g., `PROJ-1234`)
   - `ticket_name`: from `fields.summary`
   - `project_code`: from `fields.project.key`
   - `epic_name`: from `fields.parent.fields.summary` (if parent exists, otherwise empty)
   - `description`: from `fields.description` (if returned as ADF JSON, recursively extract text from `content[].content[].text` nodes; if plain text, use directly; truncate to 500 chars)

4. Write results to `{output_dir}/data/jira.csv` with columns: `ticket_id,ticket_name,project_code,epic_name,description`

**IF MCP UNAVAILABLE:** Create an empty placeholder:
```bash
echo "ticket_id,ticket_name,project_code,epic_name,description" > {output_dir}/data/jira.csv
```

---

### STEP 5 (Optional): Export Google Drive Contributions

**This step is NON-BLOCKING.** If Google Drive MCP tools are unavailable, skip this step and note it in the output.

1. Call `mcp__google-drive__list_drive_files` with query=`{email}` and maxResults=50
2. For each file, call `mcp__google-drive__get_drive_file_metadata` to get ownership, MIME type, and timestamps
3. Filter to files where:
   - The user's email is in the "owners" field
   - MIME type is one of: `application/vnd.google-apps.document`, `application/vnd.google-apps.spreadsheet`, `application/vnd.google-apps.presentation`
   - `modifiedTime` is between `{week_start_date}` and `{week_end_date}`
4. For each included file, generate a 1-2 sentence synopsis:
   - **For Google Docs and Slides**: Call `mcp__google-drive__get_document_structure` first to get section headings and previews, then generate synopsis from the structure (more effective than raw preview for RFCs and long docs)
   - **For Google Sheets or docs where `get_document_structure` returns no useful structure**: Fall back to `mcp__google-drive__get_document_preview` and summarize from the first 1000 characters
5. Save results to `{output_dir}/data/gdrive.csv` with columns: last_modified, doc_type, doc_title, doc_link, synopsis

**IF MCP UNAVAILABLE:** Create an empty placeholder:
```bash
echo "last_modified,doc_type,doc_title,doc_link,synopsis" > {output_dir}/data/gdrive.csv
```

---

### STEP 6 (Optional): Export Google Calendar Meetings

**This step is NON-BLOCKING.** If Google Calendar MCP tools are unavailable, skip this step and create a placeholder CSV.

1. Call `mcp__google-calendar-mcp__list_calendar_events` with:
   - `calendarId`: `"primary"`
   - `timeMin`: `{week_start_date}T00:00:00Z`
   - `timeMax`: `{week_end_date}T23:59:59Z`
   - `maxResults`: 100
   - `singleEvents`: true

2. **Filter events** -- exclude events that match any of: _(edit these defaults to match your organization's meeting conventions)_
   - Title contains: standup, daily, 1:1, 1-on-1, focus time, OOO, out of office, lunch, coffee
   - All-day events
   - Events the user declined

3. **Include events** that match any of: _(edit these defaults to match your organization's meeting conventions)_
   - User is the organizer
   - 5+ attendees
   - Title contains impact keywords: review, RFC, design, architecture, planning, retro, demo, launch, incident, postmortem, all-hands, guild

4. For each included event, call `mcp__google-calendar-mcp__get_calendar_event` with `calendarId: "primary"` and the event ID to get attendee details.

5. Save results to `{output_dir}/data/calendar.csv` with columns: `date,time,title,organizer,is_organizer,attendee_count,duration_minutes,description_snippet`

**IF MCP UNAVAILABLE:** Create an empty placeholder:
```bash
echo "date,time,title,organizer,is_organizer,attendee_count,duration_minutes,description_snippet" > {output_dir}/data/calendar.csv
```

---

### STEP 7 (Optional): Export Slack Mentions

**This step is NON-BLOCKING.** If Slack MCP tools are unavailable, skip this step and create a placeholder CSV.

**Limitation:** `mcp__claude_ai_Slack__slack_search_public` provides keyword search across public Slack channels. Results are treated as supplementary evidence in the analysis step, not primary contributions.

1. Run 2-3 searches using `mcp__claude_ai_Slack__slack_search_public`:
   - **Search A:** query = `"{username}"` (e.g., `"jdoe"`)
   - **Search B:** query = user's full name (if known)
   - **Search C** (optional): query = primary project name (if identifiable from earlier steps)

2. **Deduplicate** results by URL or content similarity.

3. Extract from each result: `date,channel,snippet,source_url,search_query`
   - Flag results without verifiable dates as `"date unverified"`

4. Save results to `{output_dir}/data/slack_mentions.csv`

**IF MCP UNAVAILABLE:** Create an empty placeholder:
```bash
echo "date,channel,snippet,source_url,search_query" > {output_dir}/data/slack_mentions.csv
```

---

## Phase 3: Analysis & Output (Sequential — depends on Phase 2)

### STEP 8: Analyze and Group Contributions

1. **Read all exported data files:**
   - `{output_dir}/data/github_authored.csv`
   - `{output_dir}/data/github_reviews.csv`
   - `{output_dir}/data/pr_details.json`
   - `{output_dir}/data/jira.csv`
   - `{output_dir}/data/gdrive.csv`
   - `{output_dir}/data/calendar.csv`
   - `{output_dir}/data/slack_mentions.csv`

2. **Group PRs by project:**
   - Cross-reference Jira project codes (ticket prefixes like `PROJ-`, `INFRA-`)
   - Group by repository name
   - Group by PR title prefixes or common themes

3. **Cross-reference calendar events with PRs/tickets:**
   - Match calendar events to projects (e.g., "Design Review: Project X" aligns with Project X PRs)
   - Identify meetings that provide context for code contributions

4. **Analyze Slack mentions for visibility and recognition signals:**
   - Look for recognition patterns: "thanks to", "kudos", "shoutout", mentions in other teams' channels
   - Cross-reference mentioned projects with PR/ticket data

5. **Read the career level evidence framework from `~/.claude/skills/brag-book/context/career_levels.md`**
   - If the file is empty or missing, skip career-level evidence classification.

6. **For each PR, auto-generate draft content:**
   - **What:** Synthesize from PR body text (`pr_details.json`). If body is empty or minimal, fall back to PR title + diff stats (e.g., "+120/-45 across 8 files").
   - **Why / Business Impact:** Infer from PR body (look for "why", "context", "motivation" sections), Jira ticket descriptions, and any measurable language. If unclear, draft a placeholder and flag for review.
   - **Level Evidence:** For each career level defined in `context/career_levels.md`, classify the contribution using that level's signals. Use **N/A** for a level if no relevant signals are present.

---

### STEP 9: Infer Subjective Fields

Determine these fields from the data:

1. **Primary Focus:** The project with the most PRs and/or tickets this week.

2. **Scope & Level Check:** Classify using the scope definitions from `~/.claude/skills/brag-book/context/career_levels.md`:
   - Calendar signals: organized cross-team meetings, ran design reviews = Multi-Team/Guild scope
   - Slack signals: mentioned in channels outside own team = cross-team visibility
   - List all applicable scopes.

3. **Sentiment (1-5):**
   - Draft based on: PR merge rate, feature-vs-support ratio, number of items completed
   - Account for meeting-heavy weeks: fewer PRs but high leadership activity (organizing meetings, running reviews) should not lower sentiment
   - **1** = Blocked/frustrating (many open PRs, mostly support/bugs)
   - **3** = Normal steady week (default if ambiguous)
   - **5** = High-impact shipping week (multiple features merged, large scope)
   - **Always flag sentiment for user confirmation** in the output summary.

---

### STEP 10: Categorize Contributions

1. Read the classification guidance from `~/.claude/skills/brag-book/context/contribution_categories.md`
2. For each project group from Step 8, assign a contribution category:
   - **Engineering Excellence:** Tech debt, architecture, test coverage, tooling
   - **Operational Excellence:** Incidents, support, monitoring, runbooks
   - **Business Impact:** Feature delivery, revenue, user impact, launches
3. Use the signals and classification guidelines from the context file.

---

### STEP 11: Generate Output File

1. Read the template from `~/.claude/skills/brag-book/context/brag_book_template.md`
2. Generate the brag book entry to `{output_dir}/brag_book_week_of_{week_start_date}.md`
3. Populate all sections:
   - **Sentiment, Primary Focus, Scope & Level Check** from Step 9
   - **Deliveries & Strategic Impact** from Steps 8 and 10 (grouped by project, each PR with What/Why/Evidence)
   - **Code Reviews Given** table from `github_reviews.csv`
   - **Documents & RFCs** from `gdrive.csv`
   - **Key Meetings & Collaboration** from `calendar.csv`
   - **Slack Mentions & Discussions** from `slack_mentions.csv`
   - **Weekly Metrics** computed from the data files (including meeting and Slack mention counts)

**VERIFY:** File `{output_dir}/brag_book_week_of_{week_start_date}.md` exists and contains all sections.

---

### STEP 12: Display Summary and Flag for Review

1. Display the file path: `{output_dir}/brag_book_week_of_{week_start_date}.md`

2. Show a summary:
   ```
   Weekly Brag Book Summary:
   - PRs authored: N (M merged)
   - Lines: +X/-Y
   - Reviews given: N
   - Issues: N
   - Key meetings: N (M organized)
   - Slack mentions: N
   - Primary Focus: {project}
   - Sentiment: {N}/5 (NEEDS CONFIRMATION)
   - Scope: {Team/Multi-Team/Guild/Org}
   ```

3. **Flag sentiment for user confirmation:**
   > "I've drafted sentiment as {N}/5 based on {reasoning}. Does this feel right, or would you like to adjust?"

4. Ask: "Would you like me to adjust any section or add more detail to a specific project?"

---

## Common Mistakes

- **Chat-only output:** Every step must write files to `{output_dir}/data/`. If you find yourself showing results in chat without creating files, stop and go back to Step 0.
- **Ungrouped PRs:** Always group PRs by project/theme in Step 8. Never list them individually without grouping.
- **Unconfirmed sentiment:** Never finalize sentiment without asking the user to confirm. Always flag it in Step 12.
- **Slack as primary evidence:** Slack mentions from keyword search are supplementary only — they corroborate other contributions, not replace them. Always note "date unverified" for results without clear timestamps.

---

## Example Usage

**Example 1**: "Update my brag book" (on a Friday)

You should:
1. Default username/email from context or ask
2. Calculate Monday of current week as start, today as end
3. Execute all steps, generate `{output_dir}/brag_book_week_of_2026-02-09.md`

**Example 2**: "Brag book entry for last week"

You should:
1. Calculate Monday of previous week as start, Friday of previous week as end
2. Execute all steps with those dates

---

## Notes

- PR body text is stored as JSON (not CSV) because bodies contain commas and newlines
