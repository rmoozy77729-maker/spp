# SPP Accountability

Single source of truth for the **ENS Service Provider Program (SPP)** — program governance, provider metadata, and quarterly report links.

## Interfaces

| Platform | URL | What it shows |
|---|---|---|
| Anticapture | [anticapture.com](https://anticapture.com) | Accountability table with report status, budgets, and governance links |

> Build an interface that reads this data? Open a PR to add it here.

---

## Data structure

```
spp-accountability/
├── programs.json             # Program definitions + governance proposals
├── programs.schema.json      # JSON Schema for programs
├── providers.json            # Provider metadata + quarterly report URLs
├── providers.schema.json     # JSON Schema for providers
├── validate.mjs              # Validation script (runs in CI)
└── avatars/
    └── {slug}.svg            # Provider logos
```

---

## `programs.json`

Defines each SPP term: quarters, budget, and the governance proposals that created it.

```json
{
  "programs": {
    "SPP2": {
      "name": "Service Provider Program Season 2",
      "year1Quarters": ["2025/Q3", "2025/Q4", "2026/Q1", "2026/Q2"],
      "year2Quarters": ["2026/Q3", "2026/Q4", "2027/Q1", "2027/Q2"],
      "budget": 4500000,
      "startDate": "2025-05-26",
      "discussionUrl": "https://discuss.ens.domains/t/...",
      "budgetProposal": {
        "id": "EP 6.3",
        "title": "Renew Service Provider Budget",
        "date": "2025-02-24",
        "forumUrl": "https://discuss.ens.domains/t/...",
        "snapshotUrl": "https://snapshot.box/#/s:ens.eth/proposal/...",
        "docsUrl": "https://docs.ens.domains/dao/proposals/6.3/"
      },
      "selectionProposal": {
        "id": "EP 6.10",
        "title": "Select Providers for SPP Season II",
        "date": "2025-05-12",
        "forumUrl": "https://discuss.ens.domains/t/...",
        "snapshotUrl": "https://snapshot.box/#/s:ens.eth/proposal/...",
        "docsUrl": "https://docs.ens.domains/dao/proposals/6.10/"
      }
    }
  }
}
```

### Program fields

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Full program name |
| `year1Quarters` | Yes | Quarters all providers report in (`YYYY/QN`) |
| `year2Quarters` | No | Second-year quarters (only `streamDuration: 2` providers) |
| `budget` | Yes | Total annual budget in USD |
| `startDate` | Yes | Date streams commenced (`YYYY-MM-DD`) |
| `discussionUrl` | Yes | Initial forum discussion for this program |
| `budgetProposal` | Yes | Snapshot vote that defined the budget |
| `selectionProposal` | Yes | Snapshot vote that selected providers |

### Proposal fields

| Field | Required | Description |
|---|---|---|
| `id` | Yes | ENS proposal ID (e.g., `EP 6.3`) |
| `title` | Yes | Proposal title |
| `description` | No | Brief summary |
| `date` | Yes | Vote conclusion date (`YYYY-MM-DD`) |
| `forumUrl` | Yes | Forum discussion |
| `snapshotUrl` | Yes | Snapshot vote |
| `docsUrl` | No | ENS docs page |

---

## `providers.json`

Provider metadata, program participation, and quarterly report URLs.

```json
{
  "providers": [
    {
      "name": "Your Organization",
      "slug": "your-slug",
      "website": "https://yoursite.com",
      "programs": {
        "SPP2": {
          "proposalUrl": "https://discuss.ens.domains/t/your-application",
          "budget": 400000,
          "streamDuration": 1
        }
      },
      "reports": {
        "2025/Q3": "https://discuss.ens.domains/t/your-q3-report"
      }
    }
  ]
}
```

### Provider fields

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Display name |
| `slug` | Yes | Lowercase kebab-case identifier (used for avatars) |
| `website` | No | Organization URL |
| `programs.{key}.proposalUrl` | No | Application/nomination for this program |
| `programs.{key}.budget` | Yes | Annual budget in USD |
| `programs.{key}.streamDuration` | Yes | `1` or `2` years |
| `reports.{YYYY/QN}` | — | URL to published forum report for that quarter |

---

## How to contribute

### Submit a quarterly report

1. Edit `providers.json`
2. Add your report URL to your `reports` object:
   ```json
   "reports": {
     "2025/Q3": "https://discuss.ens.domains/t/your-report-link"
   }
   ```
3. Open a pull request. CI validates automatically.

### Register as a new provider

1. Add your entry to `providers` in `providers.json`
2. Add your avatar to `avatars/{your-slug}.svg`
3. Open a pull request

### Add a new SPP program

1. Add the program definition to `programs.json`
2. Add provider entries referencing the new program key in `providers.json`
3. Open a pull request — consuming interfaces discover the new program automatically

### Update your avatar

Replace or add `avatars/{your-slug}.svg` (SVG preferred).

---

## Validation

CI runs automatically on PRs that touch data or schema files. To validate locally:

```bash
npm install ajv ajv-formats
node validate.mjs
```

The validator checks:
- JSON Schema conformance for both `providers.json` and `programs.json`
- Provider program keys reference defined programs
- Report quarter keys match program-defined quarters
- Report quarters belong to programs the provider participates in
- Providers are sorted alphabetically
- No duplicate provider slugs
- `streamDuration: 2` only used with programs that have `year2Quarters`
- Budget proposal date is before selection proposal date
