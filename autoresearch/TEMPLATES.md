# Autoresearch Templates

Templates for every file the setup guide creates. Adapt to the user's specific domain — these are structures, not fill-in-the-blank forms.

---

## PROGRAM.MD

The program file is the human-owned specification that controls the entire autoresearch loop. The agent reads this before every experiment. It defines what to optimize, what constraints to respect, and what files the agent can touch.

```markdown
# Autoresearch Program: [Slice Name]

## Objective

[1-2 sentences: what AI behavior we're improving and why it matters to the product]

## Primary Metric

`[metric_name]` — [definition of what this metric measures]

- Current baseline: [score or "pending first run"]
- Target: [threshold]

### Shadow Diagnostics

Track these alongside the primary metric for deeper visibility. They don't gate keep/discard decisions, but they reveal problems the headline number hides.

- `[diagnostic_1]` — [what it measures]
- `[diagnostic_2]` — [what it measures]
- Per-segment breakdowns (e.g., per-vendor, per-category, per-document-type)
- Step logs: tool usage, step count, token consumption (if using agents)

## Guardrails

These must not regress. If any guardrail fails, the experiment is discarded regardless of primary metric improvement.

| Guardrail | Threshold | Why |
|-----------|-----------|-----|
| [name] | >= [value] | [what breaks if this regresses] |
| [name] | >= [value] | [what breaks if this regresses] |
| [name] | >= [value] | [what breaks if this regresses] |

## Design Principles

These guide how the agent should think about changes. They come from real failure modes — not theory.

1. **[Principle name]** — [explanation]. [Why this matters: what went wrong without it]
2. **[Principle name]** — [explanation]. [Why this matters]
3. **[Principle name]** — [explanation]. [Why this matters]

Examples of good design principles:
- "Model agency over rigid rules" — vendor PDFs vary wildly; prefer improving prompts and tool definitions over hard-coded parsing rules that break on the next format. Why: rigid extraction rules required constant maintenance and broke on every new vendor.
- "Deterministic code for math, model for judgment" — price conversions, unit normalization, and validation happen in code. The model handles identity, classification, and extraction. Why: the model occasionally hallucinated arithmetic; moving math to code eliminated an entire error class.
- "Respect architecture layers" — AI returns structured JSON → deterministic normalization → operations layer → persistence. The agent must not bypass layers. Why: early experiments that shortcut layers produced results that scored well but broke the product workflow.

## Mutable Surface

Files the agent may edit during experiments:

- `[path/to/file]` — [what this file controls]
- `[path/to/file]` — [what this file controls]
- New files under `[directory/]` if justified by the hypothesis

Keep this narrow. The tighter the mutable surface, the more interpretable each experiment is.

## Immutable Surface

Files the agent must NEVER touch. These are the rules of the game.

- `[path/to/gold-labels]` — benchmark truth, not up for debate
- `[path/to/scorer]` — scoring logic must stay deterministic
- `[path/to/business-logic]` — operations/persistence layer
- `[path/to/workflow]` — orchestration, not the agent's concern
- Architecture layer boundaries (AI returns structured output → deterministic code validates → persistence)

## Required Reading

Read these before making any changes. They explain WHY things are the way they are.

- `[path/to/spec]` — [what it covers]
- `[path/to/architecture-doc]` — [what it covers]
- `[path/to/README]` — [what it covers]

The agent needs to understand the design rationale before making changes. Without this, it will make changes that technically improve the score but violate the product's intent.

## Experiment Directions

Areas to explore, roughly ordered by expected impact:

1. [direction] — [why this might help, what evidence suggests it]
2. [direction] — [why this might help]
3. [direction] — [why this might help]

These are starting points, not a fixed plan. The agent should update this list based on what it learns from each experiment.

## Stop Conditions

- `[primary_metric]` >= [target] AND all guardrails pass
- [N] consecutive experiments with no improvement
- Any guardrail violation that can't be resolved without violating design principles
- Budget exhausted (if applicable)

## Current Baseline

[Leave blank until first benchmark run. After running, record:]

| Metric | Score |
|--------|-------|
| [primary_metric] | [value] |
| [diagnostic_1] | [value] |
| [diagnostic_2] | [value] |
| [guardrail_1] | [value] |
| [guardrail_2] | [value] |

Recorded: [date] | Artifact: `[path-to-baseline-artifact]`

<!-- AUTORESEARCH_CONFIG_START
{
  "version": 1,
  "domain": "[domain-name]",
  "task": "[specific-task-identifier]",
  "primaryMetric": "[metric_name]",
  "targetMetric": [target_number],
  "baseline": {
    "[metric_name]": [value],
    "[diagnostic_1]": [value],
    "[guardrail_metric_1]": [value]
  },
  "guardrails": {
    "max_[metric]_regression": [value],
    "min_[metric]": [value]
  },
  "focusArea": "[weakest segment — e.g., a specific vendor, category, or document type]",
  "mutablePaths": [
    "[path/to/file]"
  ],
  "immutablePaths": [
    "[path/to/file]"
  ],
  "requiredReading": [
    "[path/to/file]"
  ],
  "defaultProjects": ["[project-id]"]
}
AUTORESEARCH_CONFIG_END -->
```

### Config block notes

The `AUTORESEARCH_CONFIG` JSON block is machine-readable. The loop runner extracts it to configure each experiment. Key fields:

- `version` — schema version for forward compatibility
- `domain` — what area this covers (e.g., "extraction", "classification", "retrieval")
- `task` — specific task identifier (e.g., "quote-extraction-quality")
- `primaryMetric` / `targetMetric` — what to optimize and the goal
- `baseline` — current numbers to beat (updated after each kept experiment)
- `guardrails` — constraints with thresholds (keys should match guardrail names)
- `focusArea` — the weakest segment that needs the most attention
- `mutablePaths` / `immutablePaths` — what the agent can and cannot edit
- `requiredReading` — files the agent must read before making changes
- `defaultProjects` — which datasets to run the benchmark against

---

## BENCHMARK README

```markdown
# [Slice Name] Benchmark

## What this measures

[1-2 sentences: what AI behavior this benchmark evaluates and why it matters]

## Scoring

- **Primary metric:** `[metric_name]` — [definition]
- **Segmented scoring:** results are broken down by [segments — e.g., review status (accepted/pending/all), vendor, category, document type]
- **Per-field metrics:** precision, recall, F1 for each scored field
- **Mismatch artifacts:** every disagreement between expected and actual output is recorded with full context for debugging

### Scored fields

[List every field that gets scored, e.g.:]
- `vendorProductName`, `manufacturer`, `partNumber`, `catalogNumber`
- `unitPrice`, `quotedUnitPrice`, `quotedPriceBasis`, `quantity`, `totalPrice`, `quotedTotalPrice`
- `leadTime`, `extractionConfidence`

### Fields NOT scored (context only)

These appear in the gold labels for human review but are not used in metric computation:
- [e.g., `reviewNotes` — why a label was set a certain way]
- [e.g., `sourceValues` — raw values from the source document]
- [e.g., `seedReferences` — cross-references from other data sources]
- [e.g., `aiReasoning` — model's self-reported reasoning (tracked but not scored)]

## Gold labels

Gold labels live in each project folder under `projects/[project-id]/`.

- Labels are reviewed by [who — e.g., "product owner", "domain expert", "stakeholder"]
- Each label has a `reviewStatus`: `accepted` (fully verified), `pending` (needs confirmation)
- The primary optimization target uses the `accepted` slice — this is the scientifically cleaner subset
- Use the `all` slice as a shadow diagnostic to catch regressions on pending items

## How to run

```bash
# Run benchmark for a specific project
[command] --project [project-id] --artifact-dir artifacts/[run-label]

# Run benchmark for all default projects
[command] --artifact-dir artifacts/[run-label]

# Examples for common stacks:
pnpm benchmark --project [project-id] --artifact-dir artifacts/[run-label]
python run_benchmark.py --project [project-id] --output artifacts/[run-label]
tsx benchmarks/[slice]/run-benchmark.ts --project [project-id] --artifact-dir artifacts/[run-label]
```

### Output

Each run produces a timestamped artifact directory containing:

| File | Purpose |
|------|---------|
| `run-metadata.json` | Run parameters, git hash, hypothesis, file paths |
| `summary.json` | Full metrics breakdown by project, vendor, slice, field |
| `scorecard.json` | Consolidated keep/discard recommendation with guardrail results |
| `mismatches.md` | Human-readable failure summary — start here for debugging |
| `mismatches.json` | Machine-readable mismatch details with field-level disagreements |
| `program.md` | Copy of the program file at run time (audit trail) |

## Projects

Each project in `projects/` represents a distinct dataset:

| Project | Description | Entries | Status |
|---------|-------------|---------|--------|
| [id] | [what this dataset covers] | [N] | [review status] |

## Autoresearch

The autoresearch scaffold lives in `autoresearch/`. See `autoresearch/program.md` for the optimization objective and constraints.

To run the optimization loop:
```
/autoresearch [path-to-program.md]
```
```

---

## AUTORESEARCH README

```markdown
# Autoresearch: [Slice Name]

Pre-loop harness for bounded, scored experiments against the [slice name] benchmark.

## What this optimizes

`[primary_metric]` — [definition]

Current target: `>= [target]`

## How it works

1. Read the program file (`program.md`) for objective, constraints, and surfaces
2. Read required context to understand architecture and design rationale
3. Read gold labels to understand what correct output looks like
4. Read latest mismatch artifacts to understand current failure patterns
5. Form hypothesis → make bounded change → run benchmark → evaluate → keep or discard
6. Log every run to `results.tsv` (append-only)

## Key files

| File | Purpose |
|------|---------|
| `program.md` | Human-owned objective, metrics, surfaces, principles |
| `results.tsv` | Append-only experiment ledger |
| `artifacts/` | Per-run output (metrics, mismatches, metadata) |
| `baselines/` | Baseline snapshots for comparison |
| `candidates/` | Candidate changes under evaluation (optional) |

## Standard workflow

```
Edit bounded surface
  → run benchmark with descriptive label
  → inspect artifacts (start with mismatches.md)
  → keep or discard based on scorecard
  → if keep: record in results.tsv, update baseline in program.md
  → if discard: revert, try different approach
```

## Current bottleneck

[What the main source of errors is right now — update this after each significant run]

## Running

```
/autoresearch [path-to-this-directory]/program.md
```
```

---

## RESULTS.TSV

Create the ledger with these tab-separated headers:

```
timestamp	run_label	hypothesis	files_changed	projects	segments	[primary_metric]	[diagnostic_1]	[diagnostic_2]	[focus_segment_metric]	latency_seconds	decision	guardrails	artifact_directory	git_head	notes
```

### Column definitions

| Column | Description | Example |
|--------|-------------|---------|
| `timestamp` | ISO 8601 run timestamp | `2026-03-24T14-39-36.702Z` |
| `run_label` | Short human-readable label | `baseline`, `improve-tool-defs` |
| `hypothesis` | What you thought was wrong and what you changed | `Graybar product names truncated because...` |
| `files_changed` | Comma-separated modified files | `ai/extraction/prompts.ts,ai/extraction/schemas.ts` |
| `projects` | Comma-separated project IDs tested | `62n33` |
| `segments` | Comma-separated segment filters (empty = all) | `Graybar` or empty |
| `[primary_metric]` | **Rename to your actual metric** | `0.9649` |
| `[diagnostic_N]` | **Rename to your diagnostics; add/remove as needed** | `0.9194` |
| `[focus_segment_metric]` | **Rename to your focus segment metric** | `0.963` |
| `latency_seconds` | How long the benchmark run took | `193` |
| `decision` | `keep`, `discard`, or `baseline` | `keep` |
| `guardrails` | `pass` or comma-separated list of failures | `pass` |
| `artifact_directory` | Relative path to run's artifact folder | `artifacts/2026-03-24T14-39__baseline` |
| `git_head` | Commit hash at time of run | `f464ddb` |
| `notes` | Optional free-text | `First run after architecture change` |

### Rules

- **Append-only.** Never edit or delete rows. The ledger is an audit trail.
- **Every run gets a row.** Even discarded experiments. You learn from what didn't work.
- **Baseline runs use `decision: baseline`.** They establish the starting point, not a keep/discard judgment.

---

## PROJECT README

Create one per dataset/segment in `projects/[project-id]/`:

```markdown
# Project: [project-id]

## Review Status

- **Reviewer:** [who reviewed the gold labels]
- **Review date:** [when]
- **Status:** [e.g., "owner-reviewed", "partially-reviewed", "pending-review"]
- **Accepted entries:** [N] / [total]
- **Pending entries:** [N] (awaiting [what — e.g., stakeholder confirmation, domain expert review])

## Dataset

- [N] total entries across [N] segments/vendors/categories
- [Breakdown by segment, e.g.:]
  - Segment A: [N] entries
  - Segment B: [N] entries
  - Segment C: [N] entries

## Source artifacts

[List the real input files this project benchmarks against, e.g.:]
- `path/to/vendor-quote-1.pdf`
- `path/to/vendor-quote-2.pdf`
- `path/to/comparison-sheet.xlsx`

## Gold label file

`[filename — e.g., extraction-map.json, labels.json, ground-truth.jsonl]`

### Schema

Each entry contains:

**Scored fields** (used in metric computation):
- [list each field and what it represents]

**Source values** (human reference, not scored):
- [list each field — these help reviewers but don't affect the score]

**Review metadata:**
- `reviewStatus` — `accepted` or `pending`
- `reviewNotes` — array of notes explaining label decisions

### Review notes

[Domain-specific notes about label decisions, ambiguities resolved, stakeholder clarifications. This section preserves the domain knowledge that informed the labels.]

[e.g., "Resolved 2026-03-24: stakeholder confirmed that catalog numbers and part numbers are the same for this vendor" or "When a quote prints pricing per 1000 units, unitPrice is normalized to single-unit basis"]
```

---

## GOLD LABEL STRUCTURE

Gold label files should follow these principles:

### 1. Separate scorer inputs from reviewer context

The fields the benchmark scores against (e.g., `expectedOutput`) must be distinct from fields that exist for human review (e.g., `sourceValues`, `reviewNotes`). This prevents the scorer from accidentally using human-only context and keeps the benchmark honest.

### 2. Include `reviewStatus` per entry

Use `accepted` for fully verified labels, `pending` for labels awaiting confirmation. This enables segmented scoring — optimize against the settled slice first, track the full set as a shadow diagnostic.

Why this matters: if you mix settled and unsettled labels, you can't tell if a score drop is because the model got worse or because the label was wrong. Segment first, optimize on the clean slice.

### 3. Anchor to source artifacts

Each label should reference what it was derived from:
- Source file (which PDF, document, query)
- Location within the source (page number, line reference, section)
- Human-readable context excerpt (`sourceContext`)

This makes labels auditable. When a mismatch shows up, you can trace it back to the source and determine whether the label or the model is wrong.

### 4. Preserve reviewer reasoning

`reviewNotes` capture WHY a label is what it is — especially for ambiguous cases. This is domain knowledge that helps future reviewers and the agent understand edge cases.

Good review notes:
- "Vendor SKU 744110647 is also the catalog number for this vendor — confirmed with stakeholder"
- "Pricing listed as $66.35/M — normalized to $0.06635 per unit based on quantity 60,000"
- "Two acceptable part numbers exist: the vendor's SKU and the industry standard code"

### Example structure

Adapt field names to your domain:

```json
{
  "schemaVersion": 1,
  "projectId": "[project-id]",
  "status": "[review-status]",
  "createdAt": "[date]",
  "reviewedAt": "[date]",
  "sourceArtifacts": {
    "[artifact-type]": "[path-or-reference]",
    "[artifact-type]": "[path-or-reference]"
  },
  "[policy-notes]": "[Any domain-specific scoring policies, e.g., 'When a quote uses per-1000 pricing, expectedOutput.unitPrice is normalized to single-unit basis']",
  "entries": [
    {
      "entryId": "[unique-id]",
      "reviewStatus": "accepted",
      "sourceRef": "[where in the source artifact — e.g., page 1, item 100]",
      "sourceContext": "[human-readable excerpt from the source — e.g., 'Item 100 · Qty 60000 · 744110647 · 14 SOL HF-CCS PE30 · $66.35 per 1000']",
      "sourceValues": {
        "[raw-field-1]": "[value as it appears in the source]",
        "[raw-field-2]": "[value as it appears in the source]",
        "[acceptable-alternatives]": ["[alt-1]", "[alt-2]"]
      },
      "expectedOutput": {
        "[scored-field-1]": "[correct value for scoring]",
        "[scored-field-2]": "[correct value for scoring]"
      },
      "seedReferences": [
        {
          "[cross-ref-field-1]": "[value from another data source for comparison]"
        }
      ],
      "reviewNotes": [
        "[why this label is correct]",
        "[any ambiguities and how they were resolved]",
        "[stakeholder confirmation with date if applicable]"
      ]
    }
  ]
}
```

### What makes a good gold label file

- Every entry is traceable to a source artifact
- Scored fields are cleanly separated from context fields
- Review status lets you score on settled vs. unsettled subsets
- Reviewer notes preserve domain knowledge for future reference
- Policy notes explain any normalization or transformation rules
- Alternative acceptable values are documented (e.g., multiple valid part numbers)

```
┌─────────────────────────────────────────────────┐
│ ⚠ LESSON FROM THE FIELD                        │
│                                                 │
│ The gold label file is the most important file  │
│ in the entire setup. If it's wrong, the loop    │
│ optimizes toward wrong. Spend real time on it.  │
│                                                 │
│ Separate what gets scored from what's there     │
│ for human context. Include reviewStatus so you  │
│ can score against the settled subset. Anchor    │
│ every label to its source. And preserve the     │
│ reasoning — future you will forget why that     │
│ edge case was labeled that way.                 │
│                                                 │
│ — autoresearch in production                    │
└─────────────────────────────────────────────────┘
```
