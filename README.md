# MS Connect Assistant

A GitHub Copilot agent that helps Microsoft employees prepare their **Connect performance review** by exhaustively gathering evidence and positive feedback from Microsoft 365 and GitHub data, mapping it to core priorities and role accountabilities, and producing paste-ready Connect form text.

## What It Does

The agent walks you through a structured, multi-phase workflow:

1. **Setup** — Looks up your role, confirms your core priorities, and sets the review period
2. **Evidence Gathering** — Searches M365 (emails, Teams, meetings, documents) and GitHub for work evidence, positive feedback, recognition, and developmental signals
3. **Compile** — Consolidates, deduplicates, and maps evidence to your priorities and role accountabilities, then presents a curated evidence pack for your review
4. **Draft** — Produces paste-ready text for Connect form fields (results, setbacks, goals, actions) with enforced character limits

## Prerequisites

| Requirement | Purpose |
|-------------|---------|
| [GitHub Copilot](https://github.com/features/copilot) with **Copilot Chat** | The agent runs inside Copilot Chat (VS Code or CLI) |
| [Node.js](https://nodejs.org/) ≥ 18 | Required by the WorkIQ MCP server (`npx`) |
| [WorkIQ](https://github.com/microsoft/work-iq) | MCP server that connects Copilot to your Microsoft 365 data |
| Microsoft 365 account (work) | The agent queries your M365 data via WorkIQ |
| GitHub account | Optional — for gathering GitHub contribution evidence |

### Installing WorkIQ

WorkIQ is the MCP server that gives the agent access to your emails, meetings, Teams messages, and documents. This repo's `.mcp.json` runs it automatically via `npx`, but you can also install it explicitly:

```bash
# Option A: Let npx handle it (no install needed — this is what .mcp.json does)
npx -y @microsoft/workiq mcp

# Option B: Install globally
npm install -g @microsoft/workiq
```

On first use, you'll need to accept the EULA:

```bash
workiq accept-eula
```

> ⚠️ **Tenant admin consent required:** WorkIQ needs permissions to access Microsoft 365 data. On first access, a consent dialog appears. If you're not a tenant admin, ask your admin to grant consent — see the [Tenant Administrator Enablement Guide](https://github.com/microsoft/work-iq/blob/main/ADMIN-INSTRUCTIONS.md) for details.

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/EirikHaughom/msconnect-agent.git
cd msconnect-agent
```

### 2. Open Copilot Chat

Open the project in **VS Code** and start **Copilot Chat**, or use the **GitHub Copilot CLI** from the project directory. MCP servers (WorkIQ and GitHub) start and authenticate automatically — on first run you'll be prompted to accept the [WorkIQ EULA](https://github.com/microsoft/work-iq).

### 3. Select the agent

- **VS Code:** Pick **MS Connect Assistant** from the agent dropdown in the Chat panel
- **CLI:** Type `/agent MS Connect Assistant`

### 4. Ask your question

Start simple:

> _"Help me prepare my Connect for FY26H2"_

Or provide more context up front:

> _"Prepare my Connect for FY26H2. My core priorities are in .temp/corepriorities.md. Include GitHub contributions from the msconnect-agent repo."_

### What the Agent Will Ask You

The agent uses structured interactive prompts at key decision points:

| Step | What It Asks |
|------|-------------|
| **Role** | Confirms your job title, level, and role description |
| **Core Priorities / Previous Connect** | How you want to provide them (full previous Connect or core priorities only) |
| **Lookback Period** | Which Connect period you're preparing for (e.g., FY26H2) |
| **Additional Context** | Key projects, manager guidance, or emphasis areas |
| **External Sources** | Links, metrics, or evidence from systems the agent can't search |
| **GitHub Repos** | Whether to include GitHub contributions and which repos |
| **Evidence Curation** | Review and approve the selected evidence before drafting |

### Output

The agent produces files in an output folder (e.g., `FY26H2/output/`):

| File | Description |
|------|-------------|
| `evidence-pack.md` | Full evidence pack with source trails, confidence scores, and mappings |
| `connect-draft.md` | Working draft with character counts and coverage notes |
| `connect-final.md` | **The deliverable** — clean, paste-ready text for each Connect form field |

### Tips

- **Prepare your core priorities first.** Save them to a file (e.g., `.temp/corepriorities.md`) so you can point the agent to them.
- **Have your previous Connect handy.** Save the HTML from the Connect website (Ctrl+S). The agent uses it to show progression and avoid repetition.
- **Provide metrics from tools the agent can't access** (MSX, Seismic, Power BI, CXObserve) — the agent will prompt you for these.
- **Review the evidence pack before drafting.** The agent presents a curation gate where you can swap, add, or adjust evidence.

## Project Structure

```
.github/
├── agents/
│   └── msconnect.agent.md        # Main agent definition
└── skills/
    ├── connect-writer/SKILL.md   # Drafts paste-ready Connect form text
    ├── find-feedback/SKILL.md    # Finds positive feedback across M365
    ├── github-evidence-search/SKILL.md  # Gathers GitHub contributions
    ├── humanizer/SKILL.md        # Tone and voice enforcement
    ├── m365-evidence-search/SKILL.md    # Multi-pass M365 evidence search
    └── role-lookup/SKILL.md      # Looks up role info from WorkIQ
.mcp.json                        # MCP server configuration
.gitignore                       # Excludes .temp/ (personal data)
```

## Privacy and Security

- The agent only searches **your own** M365 data (emails you sent/received, meetings you attended, etc.)
- All evidence stays local — nothing is sent to external services beyond the MCP integrations
- The `.temp/` directory is gitignored and should be used for personal files (core priorities, previous Connects, etc.)
- **Never commit personal performance data** to this repository
- Feedback quotes are preserved verbatim; work evidence is summarized to avoid exposing private messages

## Customization

The agent supports multiple Microsoft role families out of the box:

- **MCAPS / Sales** — Revenue, pipeline, partner, field enablement
- **Engineering / Product** — Features shipped, reliability, developer productivity
- **Customer Success** — Customer health, adoption, escalation resolution
- **Marketing** — Demand generation, content, campaigns
- **Finance & Operations** — Cost optimization, compliance, efficiency
- **HR / People** — Talent, engagement, culture
- **Research / Applied Science** — Publications, tech transfer, model quality

It automatically detects your role family from your job title and adjusts search keywords, impact framing, and metric priorities accordingly.

### Locale Support

The agent detects your country from your role profile and includes locale-specific praise keywords (Norwegian, German, French, Spanish, Japanese, etc.) to ensure feedback in local languages isn't missed.

## Troubleshooting

### WorkIQ not responding or returning errors

The agent relies on WorkIQ to query your M365 data. If it's not working, test it directly in Copilot Chat **without** selecting any agent:

> _"What's on my calendar tomorrow?"_

If this returns your calendar, WorkIQ is working. If not:

- **EULA not accepted** — WorkIQ requires a one-time EULA acceptance. Ask Copilot: _"Accept the WorkIQ EULA"_ and follow the prompts.
- **Node.js not installed** — WorkIQ runs via `npx`. Verify Node.js is installed: `node --version` (requires ≥ 18).
- **MCP server not started** — In VS Code, open the Command Palette and run `MCP: List Servers` to check if `workiq` is listed and running. If not, try reloading the window (`Developer: Reload Window`).
- **Authentication expired** — WorkIQ authenticates through your Microsoft 365 account. If your session expired, close and reopen VS Code or the CLI to re-authenticate.

### GitHub MCP not connecting

Test the GitHub MCP server by asking Copilot Chat (without an agent selected):

> _"List my recent GitHub pull requests"_

If this fails:

- **Not signed in to Copilot** — Ensure you're signed in to GitHub Copilot in VS Code or the CLI.
- **MCP server not listed** — Check `MCP: List Servers` in VS Code. The `github` server should appear and show as connected.

### Agent not appearing in the dropdown

- Make sure you opened the **msconnect-agent** project folder (not a parent or different folder) — the agent is discovered from `.github/agents/`.
- In VS Code, try reloading the window (`Developer: Reload Window`).
- In the CLI, ensure you're running from the repo root directory.

### Evidence searches returning empty results

- **Lookback period too narrow** — Try a wider date range. Some M365 data has retention limits.
- **Search terms too specific** — The agent searches month-by-month with targeted queries. If a topic yields nothing, provide alternative keywords or project names when prompted.
- **Data not in M365** — Some evidence lives in systems WorkIQ can't access (MSX, Seismic, Power BI, etc.). The agent will prompt you to provide this data manually.

## Contributing

Contributions are welcome! Areas where help is appreciated:

- Additional role family keyword sets
- Locale-specific praise keywords for more languages
- Improved evidence deduplication logic
- Connect form structure updates for new fiscal years

## License

This project is provided as-is for Microsoft employees to use in preparing their Connect performance reviews.
