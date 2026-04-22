---
description: >
  Search Azure DevOps for work-related contributions using ADO MCP tools
  (primary) and Azure CLI with DevOps extension as fallback. Gathers work
  items, pull requests, commits, code reviews, pipelines, and boards data,
  then maps them to core priorities and role accountabilities.
  USE FOR: ado evidence, azure devops contributions, work items completed,
  ado PRs, code reviews, pipeline runs, sprint delivery, boards evidence,
  engineering evidence, shipping evidence, bug fixes, features shipped.
---

# ADO Evidence Search

A structured methodology for gathering work-related evidence from Azure DevOps. Uses **ADO MCP tools** as the primary data source, with the **Azure CLI (`az devops`)** as a fallback when MCP tools are unavailable or return incomplete results.

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| ADO organization / projects | **Yes** | User-confirmed organization URL and project names |
| Core priorities | **Yes** | Confirmed core priorities from agent Phase 1 |
| Role description | **Yes** | Confirmed role accountabilities from agent Phase 1 |
| Lookback period | **Yes** | Start and end dates from agent Step 3 |
| Role family keywords | No | Role-dependent search terms from agent Step 1b |

## Ground Rules

- **Not all projects are work-related.** Only search projects the user explicitly confirms.
- **Impact over activity.** Focus on what was shipped, unblocked, or enabled — not raw work item counts or lines of code.
- **Month-by-month.** Search within monthly windows to ensure consistent coverage, matching the approach used in M365 evidence searches.
- **Never inflate.** Do not count automated commits, merge commits, or bot-created work items as user contributions.
- **Map to priorities.** Every evidence item should connect to a core priority or role accountability.
- **Retrieve neutrally, assess afterward.** Query for factual data (what was completed, merged, deployed). Do not ask for evaluative summaries.

---

## Step 1: Identify ADO Organization and Projects

Ask the user for their ADO organization and projects via an interactive prompt:

> _"Which Azure DevOps organization and projects should I search?_
> 1. _**Organization URL** — e.g., `https://dev.azure.com/myorg` or `myorg`_
> 2. _**Projects** — list specific projects, or say 'all' to search all projects I can access_
> 3. _**Your ADO identity** — your email or display name in ADO (for filtering contributions)"_

If the user says "all," list the available projects and ask for confirmation before searching — not all projects may be work-relevant.

---

## Step 2: Multi-Pass Evidence Gathering

For each confirmed project, run the following passes **month-by-month** within the lookback period.

### Pass 1 — Work Items (Highest Signal)

Work items (User Stories, Bugs, Tasks, Features, Epics) are the strongest evidence of delivered work.

**Using ADO MCP (primary):**
- Query work items assigned to or created by the user within the date range
- Filter by state changes: items moved to `Done`, `Closed`, or `Resolved` during the month
- Include item type, title, description, area path, iteration path, and tags

**Using Azure CLI (fallback):**
```
az boards work-item list --organization https://dev.azure.com/ORG --project PROJECT --query "SELECT [System.Id], [System.Title], [System.WorkItemType], [System.State], [System.ChangedDate], [System.AreaPath], [System.IterationPath] FROM workitems WHERE [System.AssignedTo] = 'USER_EMAIL' AND [System.ChangedDate] >= 'YYYY-MM-01' AND [System.ChangedDate] <= 'YYYY-MM-31' AND [System.State] IN ('Done', 'Closed', 'Resolved') ORDER BY [System.ChangedDate] DESC"
```

**Extract for each work item:**
- **Type and title** — what was delivered (Feature, User Story, Bug, Task)
- **Completion date** — when it was resolved or closed
- **Area path** — which component or team area
- **Iteration / sprint** — delivery cadence context
- **Tags and labels** — project, feature, or severity association
- **Parent work item** — link to Feature or Epic for broader context
- **Story points / effort** — as a scope indicator, not a quality metric

### Pass 2 — Pull Requests

PRs show code contributions and shipped changes.

**Using ADO MCP (primary):**
- Query PRs created by the user in the date range
- Filter by status: `completed` (merged)
- Include title, description, target branch, reviewers, and completion date

**Using Azure CLI (fallback):**
```
az repos pr list --organization https://dev.azure.com/ORG --project PROJECT --creator USER_EMAIL --status completed --query "[?closedDate >= 'YYYY-MM-01' && closedDate <= 'YYYY-MM-31']" --output json
```

**Extract for each PR:**
- **Title and description** — what was changed
- **Merge date** — when it shipped
- **Target branch** — main/release branch indicates production changes
- **Reviewers and votes** — who approved, any feedback received
- **Linked work items** — what user story or bug it addresses
- **Policies passed** — build validation, required reviewers (quality signal)

### Pass 3 — Code Reviews Given

Reviews the user gave to others demonstrate technical leadership and quality ownership.

**Using ADO MCP (primary):**
- Query PRs where the user was a reviewer during the date range
- Include the user's vote (Approved, Approved with suggestions, Wait for author, Rejected)

**Using Azure CLI (fallback):**
```
az repos pr list --organization https://dev.azure.com/ORG --project PROJECT --reviewer USER_EMAIL --status completed --query "[?closedDate >= 'YYYY-MM-01' && closedDate <= 'YYYY-MM-31']" --output json
```

**Extract:**
- **Count** of reviews per month
- **Notable reviews** — where the user caught significant issues or provided architectural guidance
- **Cross-team reviews** — reviews on repos outside the user's primary project

### Pass 4 — Commits (Supporting Signal)

Commits add depth but are lower signal than PRs. Use to fill gaps.

**Using ADO MCP (primary):**
- Query commits by the user in each repo within the date range

**Using Azure CLI (fallback):**
```
az repos commit list --organization https://dev.azure.com/ORG --project PROJECT --repository REPO --author USER_EMAIL --from-date YYYY-MM-01 --to-date YYYY-MM-31 --output json
```

**Extract:**
- **Commit themes** — group by feature/project, not individual commits
- **Shipping evidence** — commits to release or main branches indicating deployments
- **Do not count** merge commits, auto-generated commits, or trivial formatting changes

### Pass 5 — Pipelines and Deployments (If Applicable)

Pipeline runs show delivery velocity and operational ownership.

**Using ADO MCP (primary):**
- Query pipeline runs triggered by or associated with the user during the date range
- Include pipeline name, result (succeeded/failed), and target environment

**Using Azure CLI (fallback):**
```
az pipelines runs list --organization https://dev.azure.com/ORG --project PROJECT --query "[?requestedBy.uniqueName == 'USER_EMAIL' && finishedDate >= 'YYYY-MM-01']" --output json
```

**Extract:**
- **Deployments to production** — releases shipped, environments deployed to
- **Pipeline improvements** — changes to CI/CD pipelines the user made (new pipelines, reliability improvements)
- **Build fix contributions** — pipeline failures the user resolved

### Pass 6 — Boards and Sprint Delivery (If Applicable)

Sprint-level data shows consistent delivery and team contribution patterns.

**Using ADO MCP (primary):**
- Query iteration (sprint) data for the user's team
- Look for sprint burndown, velocity, and the user's contribution within each sprint

**Using Azure CLI (fallback):**
```
az boards iteration list --organization https://dev.azure.com/ORG --project PROJECT --team TEAM --output json
```

**Extract:**
- **Sprint completion rate** — items committed vs. delivered
- **Consistent delivery** — evidence of reliable delivery across sprints
- **Sprint goals achieved** — alignment between sprint goals and what shipped

---

## Step 3: Consolidate and Map

### Deduplication

A single work item may appear across multiple passes (linked to a PR, referenced in commits, shown in sprint data). Deduplicate by treating the **work item as the primary record** and linking related PRs, commits, and pipeline runs to it.

### Priority Mapping

For each evidence item, map to:
- **Core priority** — which priority does this contribution support?
- **Role accountability** — which accountability area does it demonstrate?
- **Impact type** — what was the business/customer/team impact?

If a contribution doesn't map to any priority or accountability, include it under a "General Engineering" category but deprioritize it in the final output.

### Impact Framing

Transform raw ADO data into impact statements. Frame as outcomes, not activity:

| ❌ Raw Data | ✅ Impact Statement |
|------------|-------------------|
| Closed 15 work items in Sprint 12 | Shipped the customer onboarding module on schedule, enabling 3 enterprise pilots to begin in March |
| Merged 8 PRs to main branch | Delivered the API security hardening ahead of the compliance deadline, passing all penetration test requirements |
| Resolved 6 bugs tagged Sev-1 | Resolved 6 critical production bugs within 48 hours, restoring service availability for 2,000+ affected users |
| Reviewed 10 PRs across 2 projects | Provided cross-team code review on the data pipeline migration, catching a data loss risk before production deployment |

---

## Step 4: Output

Return evidence as a structured list grouped by month, with each item containing:

- **Date** (month)
- **Type** (Work Item / PR / Review / Commit / Pipeline / Sprint)
- **Title / Description** — what was done
- **Project / Repo** — which ADO project and repository
- **Impact** — framed as outcome, not activity
- **Maps to** — core priority and/or role accountability
- **Source link** — ADO URL for the work item/PR/commit
- **Confidence:** Strong (completed work item with clear impact) | Partial (contribution clear, impact inferred) | Weak (minor or impact unclear)

---

## Self-Critique Gate

Before returning results, verify:

- [ ] Did I only search **user-confirmed ADO projects**?
- [ ] Did I search **month-by-month**, not the entire period in one query?
- [ ] Did I check for **code reviews given** (not just PRs authored)?
- [ ] Did I check for **work items completed** (not just created)?
- [ ] Did I **map evidence to priorities**, not just list raw activity?
- [ ] Did I **frame as impact**, not raw counts?
- [ ] Did I **exclude automated commits and bot-created items**?
- [ ] Did I try the **fallback (az devops CLI)** for any pass where MCP returned empty or incomplete results?

If any answer is "no" — rerun the relevant pass.

---

## Error Handling

| Failure | Fallback |
|---------|----------|
| ADO MCP tools unavailable | Use `az devops` CLI commands instead |
| `az devops` CLI not installed or not authenticated | Note in output; ask user to provide ADO evidence manually |
| Project access denied (permissions) | Skip project; note in output which projects were inaccessible |
| No activity found in a project | Note the gap; do not fabricate activity |
| Rate limiting or API throttling | Slow down queries; prioritize highest-signal projects first |
| WIQL query fails | Simplify the query; fall back to broader filters |

---

## Anti-Patterns to Avoid

- ❌ Counting lines of code or work item counts as a quality metric — focus on what shipped
- ❌ Including projects the user didn't confirm as work-related
- ❌ Treating automated commits or bot-created work items as user contributions
- ❌ Listing work items without mapping to priorities or framing impact
- ❌ Searching the entire date range in one query — always month-by-month
- ❌ Ignoring code reviews — they demonstrate leadership and quality ownership
- ❌ Giving up after MCP fails without trying `az devops` CLI fallback
- ❌ Asking evaluative questions — retrieve factual data, assess impact afterward
