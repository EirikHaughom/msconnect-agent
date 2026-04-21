---
description: >
  Search GitHub for work-related contributions using GitHub MCP tools (primary)
  and GitHub CLI as fallback. Gathers commits, PRs, issues, code reviews,
  and discussions, then maps them to core priorities and role accountabilities.
  USE FOR: github evidence, github contributions, code contributions,
  PRs merged, commits, code reviews, issues closed, github activity,
  open source contributions, engineering evidence, shipping evidence.
---

# GitHub Evidence Search

A structured methodology for gathering work-related evidence from GitHub. Uses **GitHub MCP tools** as the primary data source, with the **GitHub CLI (`gh`)** as a fallback when MCP tools are unavailable or return incomplete results.

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| Work-related repos/orgs | **Yes** | User-confirmed list of repositories or organizations that are work-related |
| Core priorities | **Yes** | Confirmed core priorities from agent Phase 1 |
| Role description | **Yes** | Confirmed role accountabilities from agent Phase 1 |
| Lookback period | **Yes** | Start and end dates from agent Step 3 |
| Role family keywords | No | Role-dependent search terms from agent Step 1b |

## Ground Rules

- **Not all repos are work-related.** Only search repos the user explicitly confirms.
- **Impact over activity.** Focus on what was shipped, unblocked, or enabled — not raw commit counts or lines of code.
- **Month-by-month.** Search within monthly windows to ensure consistent coverage, matching the approach used in M365 evidence searches.
- **Never inflate.** Do not count bot commits, merge commits, or automated PRs as user contributions.
- **Map to priorities.** Every evidence item should connect to a core priority or role accountability.

---

## Step 1: Identify Work-Related Repositories

Offer the user two approaches via an interactive prompt:

> _"How would you like to identify your work-related GitHub repos?_
> 1. _**Auto-detect** — I'll scan your recent GitHub activity (commits, PRs) to find repos you've contributed to, then show you the list for confirmation. I'll filter out obvious non-work repos (personal forks, hobby projects) based on org ownership and activity patterns._
> 2. _**Manual** — You provide the repo or org names directly."_

### Option 1: Auto-Detect (Default)

Use GitHub MCP tools to discover repos the user has contributed to during the lookback period:

**Using GitHub MCP (primary):**
- Use `search_pull_requests` with query `author:USERNAME created:>YYYY-MM-DD` to find all PRs
- Use `search_issues` with query `author:USERNAME created:>YYYY-MM-DD` to find all issues
- Extract the unique list of repos from the results

**Using GitHub CLI (fallback):**
```
gh search prs --author @me --created ">YYYY-MM-DD" --json repository --jq '.[].repository.nameWithOwner' | Sort-Object -Unique
```

**Filter for work-relevance using these signals:**
- ✅ **Likely work:** repos owned by the user's employer org (e.g., `microsoft/`, `Azure/`, or the team's org), repos with multiple collaborators from the same company, repos matching core priority topics
- ⚠️ **Uncertain:** personal account repos with work-sounding names, small orgs that could be work or personal
- ❌ **Likely personal:** repos under personal account with hobby/learning topics, tutorial forks, dotfiles, personal websites

Present the auto-detected repos to the user grouped by signal:

> _"I found these repos with your contributions in the lookback period:_
>
> _**Work repos** (high confidence):_
> - _`microsoft/project-alpha` — 12 PRs, 3 issues_
> - _`contoso-team/analytics-api` — 8 PRs_
>
> _**Uncertain** (need your confirmation):_
> - _`your-username/customer-demo` — 2 PRs_
>
> _**Excluded** (appear personal):_
> - _`your-username/dotfiles` — 4 commits_
>
> _Want to adjust this list? Add or remove any repos, then confirm."_

### Option 2: Manual

Accept user-provided repos:
- Specific repos (`microsoft/project-alpha`, `contoso/analytics-platform`)
- Entire organizations (`microsoft`, `Azure`)
- Wildcards within an org ("all repos in `my-team` org")

## Step 2: Multi-Pass Evidence Gathering

For each confirmed repo/org, run the following passes **month-by-month** within the lookback period.

### Pass 1 — Pull Requests (Highest Signal)

PRs are the strongest evidence of shipped work. For each month:

**Using GitHub MCP (primary):**
- Use `search_pull_requests` to find PRs authored by the user in the date range
- Use `pull_request_read` with method `get` for PR details (title, description, merged status)
- Use `pull_request_read` with method `get_files` to understand scope of change
- Use `pull_request_read` with method `get_reviews` to capture review feedback received

**Using GitHub CLI (fallback):**
```
gh pr list --author @me --state merged --search "merged:YYYY-MM-01..YYYY-MM-31" --repo org/repo --json number,title,body,mergedAt,additions,deletions,changedFiles
```

**Extract for each PR:**
- **Title and description** — what was delivered
- **Merge date** — when it shipped
- **Size** — files changed, additions/deletions (as a scope indicator, not a quality metric)
- **Review comments** — any praise or recognition in reviews
- **Labels/milestones** — project or feature association
- **Linked issues** — what problem it solved

### Pass 2 — Code Reviews Given

Reviews the user gave to others demonstrate technical leadership, mentorship, and quality ownership.

**Using GitHub MCP (primary):**
- Use `search_pull_requests` to find PRs where the user is a reviewer (query: `reviewed-by:USERNAME`)
- Use `pull_request_read` with method `get_reviews` for review details

**Using GitHub CLI (fallback):**
```
gh pr list --search "reviewed-by:@me merged:YYYY-MM-01..YYYY-MM-31" --repo org/repo --json number,title,mergedAt
```

**Extract:**
- **Count** of reviews per month
- **Notable reviews** — where the user caught significant bugs, provided architectural guidance, or unblocked others
- **Cross-team reviews** — reviews on repos outside the user's primary team (One Microsoft)

### Pass 3 — Issues

Issues show problem identification, project management, and customer responsiveness.

**Using GitHub MCP (primary):**
- Use `search_issues` to find issues created or assigned to the user
- Use `issue_read` with method `get` for issue details

**Using GitHub CLI (fallback):**
```
gh issue list --author @me --state all --search "created:YYYY-MM-01..YYYY-MM-31" --repo org/repo --json number,title,state,createdAt,closedAt,labels
gh issue list --assignee @me --state closed --search "closed:YYYY-MM-01..YYYY-MM-31" --repo org/repo --json number,title,closedAt,labels
```

**Extract:**
- **Issues filed** — bugs found, feature requests, improvements proposed
- **Issues resolved** — problems fixed, customer-reported issues addressed
- **Labels** — severity, priority, customer-impact tags

### Pass 4 — Commits (Supporting Signal)

Commits add depth but are lower signal than PRs. Use to fill gaps.

**Using GitHub MCP (primary):**
- Use `list_commits` with author filter and date range per repo

**Using GitHub CLI (fallback):**
```
gh api repos/org/repo/commits --jq '.[].commit | {message: .message, date: .author.date}' -X GET -f author=USERNAME -f since=YYYY-MM-01T00:00:00Z -f until=YYYY-MM-31T23:59:59Z
```

**Extract:**
- **Commit themes** — group by feature/project, not individual commits
- **Shipping evidence** — commits that indicate releases, deployments, or milestones
- **Do not count** merge commits, bot commits, or trivial formatting changes

### Pass 5 — Discussions and Community (If Applicable)

For repos with GitHub Discussions enabled, or for open-source contributions:

**Using GitHub MCP (primary):**
- Use `search_code` or `get_file_contents` to check for CONTRIBUTORS files, release notes mentioning the user, or README acknowledgments

**Using GitHub CLI (fallback):**
```
gh api repos/org/repo/releases --jq '.[].body' | Select-String -Pattern "USERNAME"
```

**Extract:**
- **Discussion answers** — helping others, knowledge sharing
- **Release notes mentions** — features attributed to the user
- **Open-source contributions** — external repos where the user contributed (if work-related)

---

## Step 3: Consolidate and Map

### Deduplication

A single PR may appear in multiple passes (as a PR, in commits, linked to issues). Deduplicate by treating the **PR as the primary record** and linking related commits and issues to it.

### Priority Mapping

For each evidence item, map to:
- **Core priority** — which priority does this contribution support?
- **Role accountability** — which accountability area does it demonstrate?
- **Impact type** — what was the business/customer/team impact?

If a contribution doesn't map to any priority or accountability, include it under a "General Engineering" category but deprioritize it in the final output.

### Impact Framing

Transform raw GitHub data into impact statements:

| ❌ Raw Data | ✅ Impact Statement |
|------------|-------------------|
| Merged 12 PRs in repo X | Shipped the new authentication module, reducing login failures by 40% and unblocking 3 dependent teams |
| Reviewed 8 PRs | Provided architectural review on the data pipeline redesign, catching a critical race condition before production |
| Closed 5 issues | Resolved 5 customer-reported bugs in the analytics dashboard, improving CSAT for the Contoso deployment |
| 2,400 lines added | Delivered the partner integration API ahead of schedule, enabling Fabrikam to begin pilot testing in March |

---

## Step 4: Output

Return evidence as a structured list grouped by month, with each item containing:

- **Date** (month)
- **Type** (PR / Review / Issue / Commit / Discussion)
- **Title / Description** — what was done
- **Repo** — which repository
- **Impact** — framed as outcome, not activity
- **Maps to** — core priority and/or role accountability
- **Source link** — GitHub URL for the PR/issue/commit
- **Confidence:** Strong (merged PR with clear impact) | Partial (contribution clear, impact inferred) | Weak (minor or impact unclear)

---

## Self-Critique Gate

Before returning results, verify:

- [ ] Did I only search **user-confirmed work-related repos**?
- [ ] Did I search **month-by-month**, not the entire period in one query?
- [ ] Did I check for **code reviews given** (not just PRs authored)?
- [ ] Did I **map evidence to priorities**, not just list raw activity?
- [ ] Did I **frame as impact**, not raw counts?
- [ ] Did I **exclude bot commits, merge commits, and automated PRs**?
- [ ] Did I try the **fallback (gh CLI)** for any pass where MCP returned empty or incomplete results?

If any answer is "no" — rerun the relevant pass.

---

## Error Handling

| Failure | Fallback |
|---------|----------|
| GitHub MCP tools unavailable | Use `gh` CLI commands instead |
| `gh` CLI not installed or not authenticated | Note in output; ask user to provide GitHub evidence manually |
| Repo access denied (private repo, permissions) | Skip repo; note in output which repos were inaccessible |
| No activity found in a repo | Note the gap; do not fabricate activity |
| Rate limiting | Slow down queries; prioritize highest-signal repos first |

---

## Anti-Patterns to Avoid

- ❌ Counting lines of code or commits as a quality metric — focus on what shipped
- ❌ Including non-work repos the user didn't confirm
- ❌ Treating merge commits or bot activity as user contributions
- ❌ Listing PRs without mapping to priorities or framing impact
- ❌ Searching the entire date range in one query — always month-by-month
- ❌ Ignoring code reviews — they demonstrate leadership and quality ownership
- ❌ Giving up after MCP fails without trying `gh` CLI fallback
