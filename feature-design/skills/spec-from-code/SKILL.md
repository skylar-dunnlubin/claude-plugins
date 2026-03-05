---
name: spec-from-code
description: "Generate component or screen specs from implemented code. Reads feature-design manifests for richer context, or works directly from code paths. Use when you want to create or update specs after implementation. Examples: '/spec-from-code patient-list-filtered', '/spec-from-code --path src/FilterBar.kt --depth standard', '/spec-from-code --list'."
---

# Spec From Code

Generate formal component or screen specifications from implemented code, optionally enriched by feature-design manifests.

## Argument Parsing

Parse `$ARGUMENTS` for:
- `{feature-slug}` — look for manifest at `.feature-design/{slug}/manifest.json`
- `--path {file-path}` — work directly from a code file, no manifest
- `--list` — list all available manifests and exit
- `--depth skeleton|standard|full` — controls output depth (default: `standard`)

## Mode: --list

1. Use the Glob tool to find all `manifest.json` files in `.feature-design/*/`.
2. For each, read the `feature`, `status`, and `created` fields.
3. Present:

```
Available feature manifests:
  patient-list-filtered  (reviewed, 2026-03-05)
  login-error-handling   (reviewed, 2026-03-04)
```

4. Exit — do not run the spec generation pipeline.

## Mode: Manifest-Informed (feature slug provided)

1. Read `.feature-design/{slug}/manifest.json` using the Read tool.
2. Read `.feature-design/{slug}/design-brief.md` for design context.
3. Read `.feature-design/{slug}/agent-feedback.md` for agent constraints.
4. Read `.claude/feature-design.yaml` for spec template paths and output directories.
5. Read the appropriate spec template(s) from `spec_templates.component` and/or `spec_templates.screen`.
6. For each entry in the manifest:

### New Components (`spec_target.new_components`)

For each new component spec path:
1. Read the implementation file from `implemented_files` that corresponds to this component.
2. Read the spec template from `spec_templates.component`.
3. Generate a spec following the template format, filling:
   - **Frontmatter:** id, name, category, status (draft), token_namespace, platform_implementations
   - **At chosen depth:**
     - `skeleton`: Frontmatter + Summary + Anatomy + API
     - `standard`: + Token Mapping + States + Accessibility + Usage Guidelines
     - `full`: + Acceptance Criteria (provisional IDs) + State Transitions + Traceability + Changelog
   - Use agent feedback for: token recommendations (brand-guardian), a11y requirements (accessibility-expert), mobile constraints (mobile-ux-optimizer)
4. Write the spec file using the Write tool to the path specified in `spec_target.new_components`.

### Extended Components (`spec_target.update_components`)

For each existing component spec to update:
1. Read the EXISTING spec file using the Read tool. This is critical — never overwrite.
2. Read the implementation file to understand the new variant/extension.
3. Identify what's new: additional variant, new props, expanded states.
4. Append to the existing spec:
   - Add new variant section (if applicable)
   - Append new props to the API table
   - Add new states if introduced
   - Append to Changelog: `| {version} | Added {variant} for {feature-slug} |`
   - Preserve ALL existing content, IDs, and traceability
5. Write the updated spec using the Edit tool (not Write — to show the diff).

### Screen Spec (`spec_target.screen`)

If scope is `screen`:
1. Read the screen spec template from `spec_templates.screen`.
2. Generate screen spec in S1-S18 format:
   - S1: Frontmatter (screen_id with provisional ID, module, related_screens)
   - S2: Purpose and user context (from design brief)
   - S3: Screenshots — write `**Pending:** Capture via Paparazzi or manual screenshot.`
   - S4: Intended Functionality (derive from design brief + implementation)
   - S5-S6: User Needs + System Requirements (provisional IDs: UN-{SLUG}-*, SYS-{SLUG}-*)
   - S7: State orchestration (from design brief states)
   - S8-S10: Visual states, edge cases, acceptance criteria
   - At `full` depth: S11-S18 (assumptions, traceability matrix, etc.)
3. Write the screen spec to `spec_target.screen` path.

## Mode: Code-Only (--path provided)

1. Read `.claude/feature-design.yaml` for template paths and output directories.
2. Read the code file at the provided path using the Read tool.
3. Detect scope:
   - Single `@Composable` function, React component, or Swift View → component spec
   - File with navigation, state management, multiple composed components → screen spec
4. Read the appropriate spec template.
5. Generate spec from code analysis only (no manifest context).
6. Ask the user where to write the spec: suggest `{inventory.specs}/{component-name}.md` or `{inventory.screens}/{screen-name}.md`.
7. If target file exists, read it first and merge (same as Extended Components above).
8. Write the spec.

## Depth Reference

| Depth | Component Sections | Screen Sections |
|-------|-------------------|-----------------|
| skeleton | Frontmatter, Summary, Anatomy, API | S1, S2, S4 |
| standard | + Token Mapping, States, Accessibility, Usage Guidelines | + S3 (pending), S5, S6, S7, S8, S9 |
| full | + Acceptance Criteria, State Transitions, Traceability, Changelog | + S10, S11, S15, S18 |

## Completion

After generating all specs, present:

```
── Specs Generated ───────────────────────────────

Created:
  specs/components/filter-bar.md (standard depth)
  specs/components/pagination.md (standard depth)
  specs/screens/patient-list.md (standard depth, screenshots pending)

Updated:
  specs/components/table.md (added FilterableTable variant)

Provisional IDs used — swap with real QMS IDs when available.

──────────────────────────────────────────────────
```
