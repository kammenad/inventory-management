---
name: saas-sidebar-redesign
description: Redesign a Vue 3 app's UI into a modern SaaS-style interface with a fixed vertical sidebar on the left in place of a top nav bar, consistent spacing, and a polished professional look. Use when asked to restyle, modernize, or re-layout a Vue 3 app around a sidebar, or to replace a top navigation bar.
---

# SaaS Sidebar Redesign

Converts a Vue 3 app from a top navigation bar to a fixed left sidebar, and normalizes spacing and styling through shared design tokens.

## What this skill bundles

- [tokens.css](tokens.css): spacing scale, slate/blue palette, status colors, radii, shadows, type scale, sidebar sizes, z-index layers.
- [templates/AppSidebar.vue](templates/AppSidebar.vue): reference sidebar component (brand block, route-driven nav, footer slot).
- [templates/app-shell.md](templates/app-shell.md): target shell markup, global CSS, and the checklist for sticky elements and modals.

The templates are starting points. Adapt them to the app in front of you; do not paste them in unchanged.

## Scope

In scope: a fixed, full-height, left sidebar; shared tokens; consistent spacing, radii, borders, and hover/active/focus states in the files you touch.

Out of scope unless the user asks: collapsible or icon-rail sidebar, mobile drawer, grouped nav sections, dark mode, rewriting every view's hardcoded colors, and any backend or API change.

## Rules that always apply

- Follow the project's own CLAUDE.md design rules (palette, charts, "no emojis in the UI"). Project rules win over this skill.
- Icons are inline SVG. Do not add an icon library or any new dependency.
- Nav labels come from the app's i18n keys, never hardcoded strings. If a label is currently hardcoded, add keys to every locale file.
- Do not touch the backend, API contracts, or data files.
- Comment non-obvious layout logic (why an offset exists, why a z-index is set).
- If the repo is public, never commit credentials, internal hostnames, private registry URLs, or lockfiles the project ignores. Leave `.npmrc` and similar config alone.

## Delegation

If the project defines a `vue-expert` agent (`.claude/agents/vue-expert.md`), hand every `.vue` file you create or significantly modify to that agent. Give it the token file path, the sidebar template path, and the exact nav/i18n details you discovered. If no such agent exists, edit directly.

## Workflow

### 1. Discover (read-only)

Do not assume the layout of any particular repo. Find:

- The router file and its routes (for example `src/main.js` or `src/router/index.js`).
- The shell component (usually `src/App.vue`), its nav markup, and its global (unscoped) style block.
- Every sticky or fixed element: `grep -rn "position: *\(sticky\|fixed\)" src`, and note each `top`/`z-index`.
- The i18n setup and the existing nav label keys.
- Components that live in the nav today (language switcher, profile menu, user avatar) so they can be relocated rather than rewritten.
- Hardcoded color frequency, to confirm the tokens match the existing palette: `grep -rhoE "#[0-9a-fA-F]{6}" src --include=*.vue | sort | uniq -c | sort -rn | head`.

If there is no router or no single shell component, stop and ask the user how the app is structured before editing.

### 2. Plan

List the files that will change and what changes in each, then proceed. Keep the change set small: shell, sidebar, tokens, entry file, plus dependents found in step 1.

### 3. Install tokens

Copy [tokens.css](tokens.css) to `src/styles/tokens.css` and import it once from the entry file. Compare its palette with the color frequency from step 1; if the app uses a different brand color, change the token values, not the call sites. Replace hardcoded values with tokens only in files you are already editing.

### 4. Build the sidebar

Adapt [templates/AppSidebar.vue](templates/AppSidebar.vue) into the app's components folder:

- Drive nav items from the real routes and i18n keys.
- Match the project's component style (Composition API form, scoped styles).
- Move the existing language switcher, profile menu, and similar controls into the footer slot.

### 5. Restructure the shell

Follow [templates/app-shell.md](templates/app-shell.md): replace the top-nav markup and its CSS with the sidebar plus an offset main column. Delete the old top-nav rules completely, including pseudo-element underlines tied to the old bar.

### 6. Fix dependents

Work through the checklist in the shell template. The usual breakages are sticky bars still offset by the old top-nav height, modals stacked below the sidebar, and hardcoded `calc(100vh - ...)` heights.

### 7. Normalize spacing and polish

In the files you touched, use the `--space-*` scale for padding, margin, and gap, and use the shared radii, border color, and shadow. Page headers, cards, tables, and stat grids should share the same rhythm. Every interactive element needs hover, active, and a visible focus state.

### 8. Verify in a browser

Start the app, then use Playwright against the dev server:

1. Visit every route. Confirm the sidebar is on the left, there is no top nav, and content is not hidden under the sidebar.
2. Confirm the active nav item changes with the route, and `/` is not active on other routes.
3. Switch language (if the app has i18n) and confirm nav labels change.
4. Open a modal and confirm it overlays the sidebar.
5. Scroll a page with a sticky bar or sticky table header and confirm it sticks at the top of the content column without overlap or a gap.
6. Check the browser console for new errors, and run the production build (`npm run build`) to confirm it succeeds.
7. Take a screenshot of at least the home route and one data-heavy route.

### 9. Report

Summarize what changed, list any hardcoded colors or spacing you left alone, and state plainly what is not handled (for example narrow viewports, since the sidebar is not collapsible). If anything failed verification, say so with the output.
