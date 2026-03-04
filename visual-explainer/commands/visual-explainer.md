---
description: Generate a visual HTML explanation with full design pipeline — brainstorm the concept, apply professional design principles, then produce a stunning self-contained HTML page
---
Generate a visual explanation for: $@

This is the full visual-explainer pipeline. Follow these phases in order:

## Phase 1: Brainstorm (Design Before Building)

Before touching HTML, explore the concept. Invoke the `superpowers:brainstorming` skill to:

- Clarify what the user wants to visualize and who the audience is
- Identify the right diagram type (architecture, flowchart, data table, timeline, dashboard, slides, or hybrid)
- Consider 2-3 aesthetic directions that fit the content
- Map the information hierarchy: what dominates, what supports, what's collapsible
- Surface any data gathering needed (git history, codebase reading, file analysis)

Keep brainstorming focused — 2-3 minutes of alignment, not a full design doc. The goal is a clear direction before generating.

## Phase 2: Design (Apply Professional Aesthetics)

With direction established, invoke external design skills for additional polish. Use the Skill tool to load `ui-ux-pro-max:ui-ux-pro-max` and `frontend-design:frontend-design` before proceeding. Apply their guidance to these decisions:

- **Typography**: Select a distinctive font pairing that matches the content's voice. Never use Inter, Roboto, or system-ui as primary. Consider the pairing's personality — editorial serif for reviews, geometric sans for technical diagrams, monospace for terminal-aesthetic.
- **Color palette**: Build a cohesive palette with CSS custom properties. Use semantic naming. Support both light and dark themes. Avoid Tailwind default purples, cyan-magenta-pink neon, and gradient text.
- **Layout strategy**: Choose the spatial approach — CSS Grid cards, Mermaid diagrams, hybrid patterns, or split panels. Vary visual weight to create hierarchy (hero sections large and prominent, reference sections compact).
- **Animation**: Plan entrance animations that guide the eye. Staggered fade-ins, scale reveals for KPIs. No glowing shadows, no pulsing effects.

**Priority rule:** If guidance from `ui-ux-pro-max` or `frontend-design` conflicts with rules in the visual-explainer skill (forbidden fonts, forbidden colors, anti-slop constraints), the visual-explainer skill's rules take precedence.

## Phase 3: Generate (Visual Explainer Skill)

Load the visual-explainer skill and execute its full workflow:

1. **Think** — commit to the aesthetic direction chosen in Phase 2
2. **Structure** — read the appropriate reference templates and CSS patterns from the skill's resources
3. **Style** — apply the typography, color, layout, and animation decisions from Phase 2
4. **Deliver** — write to `~/Desktop/Visual Explainer Pages/` and open in browser

Follow all visual-explainer quality checks:
- The squint test (hierarchy perceivable when blurred)
- The swap test (not interchangeable with a generic dark theme)
- Both themes intentional
- No overflow at any viewport width
- Mermaid zoom controls on all diagram containers
- Information completeness (pretty but incomplete = failure)

## When to Use Each Sub-Command

If the user's request maps to a specific format, use the dedicated command instead of the full pipeline:

| Request | Command |
|---------|---------|
| "Review this diff / PR / branch" | `/diff-review` |
| "Review this implementation plan" | `/plan-review` |
| "Recap this project" | `/project-recap` |
| "Generate a slide deck" | `/generate-slides` |
| "Visualize this plan / feature" | `/generate-visual-plan` |
| "Fact-check this document" | `/fact-check` |
| Generic diagram / visualization | `/generate-web-diagram` |

This command (`/visual-explainer`) is the full-pipeline entry point — use it when the request benefits from the brainstorm-design-generate sequence, or when the user wants maximum design quality.
