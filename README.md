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
| Microsoft 365 account (work) | The agent queries your M365 data via WorkIQ |
| GitHub account | Optional — for gathering GitHub contribution evidence |

## Setup

### 1. Clone the Repository

```bash
git clone https://github.com/EirikHaughom/msconnect-agent.git
cd msconnect-agent
```

### 2. Configure MCP Servers

The repository includes a `.mcp.json` that configures two MCP servers:

- **WorkIQ** — Queries Microsoft 365 Copilot for emails, meetings, Teams messages, documents, and people data
- **GitHub** — Accesses GitHub repositories, PRs, commits, issues, and code reviews

The `.mcp.json` is pre-configured — no manual edits needed:

```json
{
  "inputs": [],
  "servers": {
    "workiq": {
      "command": "npx",
      "args": ["-y", "@microsoft/workiq@latest", "mcp"],
      "tools": ["*"]
    },
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    }
  }
}
```

### 3. Start the MCP Servers

When you open this project in VS Code with Copilot Chat, the MCP servers start automatically. On first run:

1. **WorkIQ** will install via `npx` and prompt you to accept the [EULA](https://github.com/microsoft/work-iq-mcp). You must accept before it can query your M365 data.
2. **GitHub MCP** connects automatically through your Copilot authentication.

> **Tip:** You can verify MCP server status in VS Code by opening the **MCP Servers** panel (click the `{}` icon in the Copilot Chat toolbar, or run `MCP: List Servers` from the Command Palette).

### 4. Accept the WorkIQ EULA

The first time WorkIQ is used, you'll be prompted to accept the EULA. The agent will handle this automatically, but you can also accept it manually:

> In Copilot Chat, type: _"Accept the WorkIQ EULA"_

## Usage

### Starting the Agent

Open this project in VS Code, then in **Copilot Chat**, invoke the agent:

> `@MS Connect Assistant` — _"Help me prepare my Connect for FY26H2"_

Or provide more context up front:

> `@MS Connect Assistant` — _"Prepare my Connect for FY26H2. My core priorities are in .temp/corepriorities.md. Include GitHub contributions from the msconnect-agent repo."_

### What the Agent Will Ask You

The agent uses structured interactive prompts at key decision points:

| Step | What It Asks |
|------|-------------|
| **Role** | Confirms your job title, level, and role description |
| **Core Priorities** | How you want to provide them (paste, file, manual entry) |
| **Previous Connect** | Whether you have a prior Connect to import for context |
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

## Contributing

Contributions are welcome! Areas where help is appreciated:

- Additional role family keyword sets
- Locale-specific praise keywords for more languages
- Improved evidence deduplication logic
- Connect form structure updates for new fiscal years

## License

This project is provided as-is for Microsoft employees to use in preparing their Connect performance reviews.
