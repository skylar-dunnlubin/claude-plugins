---
name: feature-design
description: "Full-pipeline feature design orchestration. Use when starting a new feature from an idea — scans existing components, validates with specialized agents (brand, a11y, mobile, UX research), builds with feature-dev, reviews with code-review. Requires .claude/feature-design.yaml config in the project. Examples: 'design a patient list with filtering', 'I need a settings screen', 'add a login flow', 'build a dashboard widget'."
---

# Feature Design Pipeline

Orchestrate the full journey from idea to reviewed, implemented code.

## Prerequisites

This skill requires:
- `.claude/feature-design.yaml` in the project root (config with inventory paths)
- Installed agents: brand-guardian, accessibility-expert (required); mobile-ux-optimizer (for mobile); ux-researcher (opt-in)
- Installed skills: feature-dev, code-review or coderabbit

## Stage 0: Setup

1. Read `.claude/feature-design.yaml` from the project root using the Read tool.
   - If the file does not exist, tell the user: "No feature-design config found. I need to know where your components, specs, tokens, and source files live." Then ask for each path one at a time, detect platform from file extensions (`.kt`/`.swift` → mobile, `.tsx`/`.jsx`/`.vue` → web, both → both), and write the config file using the Write tool.
2. Parse the config to extract: `inventory.specs`, `inventory.screens`, `inventory.tokens`, `inventory.components`, `platform`, and any `pipeline` overrides.
3. Derive a feature slug from the user's description (kebab-case, concise — e.g., "patient list with filtering" → `patient-list-filtered`).
4. Present the slug to the user: "Feature: `{slug}`. Adjust or confirm?"
5. Wait for user confirmation before proceeding.

## Stage 1: Inventory Scan

1. Use the Glob tool to list all `.md` files in the `inventory.specs` path.
2. For each spec file, use the Read tool to read the YAML frontmatter (first ~20 lines). Extract: `id`, `name`, `category`, `status`, `token_namespace`, `platform_implementations`, `related`, `tags`.
3. Use the Glob tool to list all source files in `inventory.components` path (`.kt`, `.tsx`, `.swift`, etc. based on platform).
4. Use the Glob tool to list token files in `inventory.tokens` path. Read component-tier token file to extract `comp.*` namespaces.
5. Build a mental inventory table: component name → spec status, category, token namespace, implementation file, related components.
6. Match the inventory against the user's feature description. Present relevant matches:

```
── Inventory Scan ────────────────────────────────

EXISTING COMPONENTS (relevant to "{feature description}"):

| Component | Spec Status | Tokens | Implementation |
|-----------|------------|--------|----------------|
| Table     | stable     | comp.table.* | Table.kt |
| SearchInput | draft   | comp.search-input.* | SearchInput.kt |
| ...       | ...        | ...    | ...            |

Unrelated components omitted ({N} total in inventory).

──────────────────────────────────────────────────
Adjust, approve, or ask questions before continuing.
```

7. **Gate 1:** Wait for user response. Incorporate any adjustments (e.g., "also include AppTopBar") into the inventory context.

## Stage 2: Catalog Match

1. Read the component catalog from `${CLAUDE_PLUGIN_ROOT}/skills/feature-design/references/component-catalog.md` using the Read tool.
2. Extract intent keywords from the user's feature description. Match against component names AND aliases in the catalog. For example, "filtering" matches Combobox (alias: Autocomplete), Badge (alias: Chip, Tag), Select (alias: Dropdown).
3. Cross-reference catalog matches against the inventory from Stage 1:
   - **REUSE:** Component exists in inventory AND is relevant → use as-is
   - **EXTEND:** Component exists but needs new props/variants for this feature → propose extension
   - **NEW:** Component needed but not in inventory → reference catalog examples for design guidance
4. Compose a screen-level layout tree showing where each component sits:

```
── Catalog Match + Composition ───────────────────

REUSE/EXTEND/NEW DECISIONS:

| Component | Decision | Basis | Catalog Ref |
|-----------|----------|-------|-------------|
| AppTopBar | REUSE | Existing, fits as-is | Header (41 ex.) |
| Table | EXTEND | Add filter/sort props | Table (74 ex.) |
| FilterBar | NEW | No equivalent in inventory | Combobox (37 ex.) |
| Pagination | NEW | Not in inventory | Pagination (49 ex.) |

SCREEN COMPOSITION:
  AppTopBar [REUSE]
  SearchInput [REUSE]
  FilterBar [NEW]
  ├── Combobox x N [NEW]
  └── Badge (active filter count) [REUSE]
  FilterableTable [EXTEND from Table]
  ├── Rows → navigate to {related screen}
  └── Skeleton rows (loading) [REUSE]
  Pagination [NEW]
  EmptyState (filtered-empty variant) [REUSE]

  States: Loading | Populated | Filtered-Empty | Error

──────────────────────────────────────────────────
Adjust reuse/extend/new decisions, composition, or states before continuing.
```

5. **Gate 2:** Wait for user response. Incorporate adjustments into the composition context.

## Stage 3: UX Research (opt-in)

**Skip this stage unless:** the user passed `--research`, OR the config has `pipeline.include: [ux-researcher]`, OR the user explicitly asks for research.

If running:
1. Construct a prompt containing: the feature description, inventory matches from Stage 1, composition from Stage 2, and all user adjustments so far.
2. Dispatch the ux-researcher agent using the Agent tool:
   - subagent_type: `Explore`
   - Inject the ux-researcher system prompt by reading its agent file from the plugin cache
   - Prompt: "You are a UX researcher. Given this feature design: {accumulated context}. Research: What do users typically expect from this type of feature? What are common friction patterns? What interaction patterns work best? Return structured findings as markdown."
3. Present the agent's findings:

```
── UX Research Findings ──────────────────────────
{agent output}
──────────────────────────────────────────────────
Adjust, approve, or ask questions before continuing.
```

4. **Gate 3:** Wait for user response. Incorporate adjustments.

## Stage 4: Brand Alignment

**Skip this stage if:** config has `pipeline.skip: [brand-guardian]`.

1. Construct a prompt containing ALL accumulated context: feature description, inventory, composition, reuse decisions, user adjustments, and UX research (if it ran).
2. Dispatch the brand-guardian agent using the Agent tool:
   - subagent_type: `brand-guardian:brand-guardian`
   - Prompt: "You are reviewing a feature design for brand alignment. Feature: {feature description}. Platform: {platform}. The project uses these token namespaces: {token namespaces from Stage 1}. Proposed composition: {composition from Stage 2}. {UX findings if available}. User adjustments so far: {adjustments}. Review this composition against the project's brand system. Evaluate: visual consistency, token usage recommendations for new components, brand voice for labels/empty states/error messages. Flag any conflicts. Return structured findings as markdown."
3. Present the agent's findings:

```
── Brand Guardian Feedback ───────────────────────
{agent output}
──────────────────────────────────────────────────
Adjust, approve, or ask questions before continuing.
```

4. **Gate 4:** Wait for user response. Incorporate adjustments.

## Stage 5: Mobile Optimization

**Skip this stage if:** `platform` is `web`, OR config has `pipeline.skip: [mobile-ux-optimizer]`.

1. Construct a prompt containing ALL accumulated context including brand feedback.
2. Dispatch the mobile-ux-optimizer agent using the Agent tool:
   - subagent_type: `Explore`
   - Inject the mobile-ux-optimizer system prompt by reading its agent file from the plugin cache
   - Prompt: "You are a mobile UX optimization specialist. Feature: {feature description}. Platform: {platform}. Proposed composition: {composition}. Brand constraints: {brand feedback}. {UX findings if available}. Review this design for mobile-first optimization. Evaluate: touch target sizing (minimum 48dp), thumb zone placement, responsive layout, scroll performance, animation smoothness. Return structured recommendations as markdown."
3. Present findings with gate.
4. **Gate 5:** Wait for user response. Incorporate adjustments.

## Stage 6: Accessibility

**Skip this stage if:** config has `pipeline.skip: [accessibility-expert]` (discouraged).

1. Construct a prompt containing ALL accumulated context including brand and mobile feedback.
2. Dispatch the accessibility-expert agent using the Agent tool:
   - subagent_type: `Explore`
   - Inject the accessibility-expert system prompt by reading its agent file from the plugin cache
   - Prompt: "You are an accessibility expert. Feature: {feature description}. Proposed composition: {composition}. Brand constraints: {brand feedback}. Mobile constraints: {mobile feedback}. Review this design for accessibility compliance. Evaluate: WCAG 2.1 AA requirements per component, color contrast ratios (4.5:1 normal text, 3:1 large text), keyboard navigation flow, screen reader announcements, focus management, touch target compliance. Return structured requirements as markdown."
3. Present findings with gate.
4. **Gate 6:** Wait for user response. Incorporate adjustments.

## Stage 7: Design Brief Consolidation

1. Merge ALL stage outputs and user adjustments into a single consolidated design brief.
2. Present the brief:

```
── Consolidated Design Brief ─────────────────────

FEATURE: {feature slug}
SCOPE: {screen | component}
PLATFORM: {mobile | web | both}

COMPONENTS:

| Component | Decision | Tokens | Brand Notes | Mobile | A11y |
|-----------|----------|--------|-------------|--------|------|
| {name} | REUSE/EXTEND/NEW | {namespace} | {notes} | {constraints} | {requirements} |
| ... | ... | ... | ... | ... | ... |

SCREEN COMPOSITION:
  {layout tree from Stage 2, updated with all agent feedback}

STATES:
  {state list with transitions}

CONSTRAINTS SUMMARY:
  Brand: {consolidated brand requirements}
  Accessibility: {consolidated a11y requirements}
  Mobile: {consolidated mobile requirements}

──────────────────────────────────────────────────
Approve this brief to proceed to implementation, or adjust.
```

3. **Gate 7:** This is the "go build" decision. Wait for explicit approval before proceeding.

## Stage 8: Build

1. Invoke the `feature-dev:feature-dev` skill with the approved design brief as context.
2. The skill prompt should include: the full design brief, the specific files to create/modify, the reuse/extend/new decisions, and all agent constraints.
3. feature-dev implements the feature in the codebase.
4. **Gate 8:** Present the implementation to the user. Wait for review.

## Stage 9: Review

1. Determine review tool from config: `pipeline.review` field. Default: `code-review`.
2. If `code-review` or not specified: invoke the `superpowers:code-reviewer` agent or `superpowers:requesting-code-review` skill.
3. If `coderabbit`: invoke the `coderabbit:code-review` skill.
4. Present review findings.
5. **Gate 9:** If issues found, fix them. Wait for user confirmation that review is resolved.

## Stage 10: Manifest Write

1. Create the manifest directory: `.feature-design/{feature-slug}/`
2. Write `manifest.json` using the Write tool:

```json
{
  "feature": "{feature-slug}",
  "scope": "{screen | component}",
  "created": "{YYYY-MM-DD}",
  "status": "reviewed",
  "design_brief": "./design-brief.md",
  "agent_feedback": "./agent-feedback.md",
  "implemented_files": ["{list of files created/modified by feature-dev}"],
  "reuse_decisions": {
    "reuse": ["{component names}"],
    "extend": ["{component → variant}"],
    "new": ["{component names}"]
  },
  "review_summary": "{one-line review outcome}",
  "spec_target": {
    "screen": "{path if scope is screen}",
    "new_components": ["{spec paths for new components}"],
    "update_components": ["{spec paths for extended components}"]
  }
}
```

3. Write `design-brief.md` — the consolidated brief from Stage 7.
4. Write `agent-feedback.md` — concatenated agent outputs from Stages 3-6, with section headers per agent.
5. Check if `.feature-design/` is in the project's `.gitignore`. If not, append it:

```bash
echo ".feature-design/" >> .gitignore
```

6. Present completion message:

```
── Feature Complete ──────────────────────────────

Feature "{feature-slug}" is built and reviewed.

Manifest saved to .feature-design/{feature-slug}/

To generate specs:
  /spec-from-code {feature-slug}
  /spec-from-code {feature-slug} --depth standard
  /spec-from-code {feature-slug} --depth full

To clean up:
  /feature-design --clean {feature-slug}

──────────────────────────────────────────────────
```

## Flags

The skill should check `$ARGUMENTS` for these flags:
- `--research` → include Stage 3 (UX Research) even if not in config
- `--clean` → skip pipeline, delete `.feature-design/{slug}/` or all manifests if no slug
- `--clean {slug}` → delete specific manifest directory
- `--list` → skip pipeline, list all existing manifests in `.feature-design/`
