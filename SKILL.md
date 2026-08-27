---
name: theme-forge
description: "Use this skill whenever a user is starting, planning, auditing, developing, or reconciling a standalone eCommerce / Shopify storefront design prototype outside a Shopify theme directory. Run /theme-forge init to scaffold a new standalone prototype workspace (HTML/CSS/JS, tokens, documentation, AGENTS.md, CLAUDE.md), /theme-forge init-agents (or /theme-forge agents) to initialize or refresh agent guidance files in an existing project, or /theme-forge resolve to audit, reconcile, and resolve an existing theme prototype with updated skill rules, token architecture, OOP JS, and design integrations."
compatibility: "Plain Markdown workflow; requires filesystem access to the selected project root. Self-contained with zero runtime, build tool, framework, package manager, or external skill dependencies. No Shopify credentials required."
---

# Theme Forge

Theme Forge is a design-first workflow for building eCommerce storefront prototypes. It develops visual design, interaction architecture, token systems, and components completely standalone in a dedicated prototype workspace, keeping frontend craft independent of Shopify theme directories until the design contract is complete.

The skill is reusable across stores and themes. A theme name, theme slug, category, visual language, and design direction may change per project. The prototype rules, token architecture, and OOP JavaScript standards remain consistent.

## Portability and Invocation

This is a plain Markdown skill. It does not require a runtime, build tool, MCP server, package manager, or external design application.

When the host supports slash commands, use:

```text
/theme-forge init
/theme-forge init-agents [scope]
/theme-forge resolve [scope]
```

*(Note: `/theme-forge agents` is supported as an alias for `/theme-forge init-agents`.)*

Target scopes for `init-agents`:
- `all` (default): Refresh full agent guidance in `AGENTS.md`, ensure `CLAUDE.md` symlink, and update `.shopifyignore`.
- `guidelines`: Update only the design and coding rules section in `AGENTS.md`.
- `tokens`: Update the token and global settings contract reference in `AGENTS.md`.

Target scopes for `resolve`:
- `all` (default): Full audit and resolution across styles (tokens/colors/logical CSS), scripts (OOP classes), HTML templates (single shell/inline stripping/scaffolds), global settings contract (`docs/theme-settings.md`), design skill integration, and agent guidance.
- `styles`: Audit and reconcile `design/styles.css` (zero hardcoded colors, 9-category CSS custom properties, color scheme scoping, bidirectional logical CSS properties).
- `scripts`: Audit and refactor `design/script.js` into ES6 OOP classes with standardized lifecycles (`init`, `destroy`) and a central component registry.
- `templates`: Normalize HTML prototype templates (enforce single header/footer in `index.html`, strip duplicate header/footer from secondary pages, strip inline styles/scripts, add missing default page scaffolds).
- `docs`: Reconcile and synchronize `docs/theme-settings.md` global settings contract with active code tokens and features.

When the host does not support slash commands, interpret `init`, `init-agents`, `agents`, or `resolve` as the first user-provided argument to the skill. If no command is provided, explain the available commands and ask which one to run. Do not silently choose a command.

The skill must work from a project directory, not only from the directory where the skill file is installed. Resolve all project paths relative to the user's selected project root.

### Optional Design Skill Integration

- If the `frontend-design` skill (or an equivalent frontend design skill) is installed and available in the environment, leverage it to guide visual direction, layout hierarchy, typography, color schemes, motion, and UI refinement when prototyping in `design/` or resolving existing theme styles.
- During `/theme-forge resolve`, leverage available design skills to review visual craft, contrast, typography scaling, responsive behaviors, and micro-interactions while strictly enforcing Theme Forge token and architecture rules.
- If `frontend-design` is not installed, proceed normally using Theme Forge's built-in design principles. Do not fail, block, or attempt automatic package installation.
- Always preserve Theme Forge boundaries: all styles must live strictly in `design/styles.css`, client scripts in `design/script.js`, with no inline styles/scripts and no Liquid in prototypes.

### Command selection

1. Read the user's requested command and optional scope from the arguments or context.
2. If it is `init`, follow the standalone prototype initialization workflow.
3. If it is `init-agents` or `agents`, follow the existing project agent initialization workflow (scoped strictly to the standalone design project without modifying design files).
4. If it is `resolve`, follow the existing theme resolution and reconciliation workflow (auditing and updating styles, scripts, templates, settings contract, and integrations).
5. If no command is present, explain the valid commands (`init`, `init-agents`, `resolve`) and ask which one to run; do not inspect or modify project files yet.
6. If the command is unknown, show the valid commands and ask the user to choose.

At the start of a command, state the selected project root, command, and the files that will be inspected or created. At completion, report four separate lists: created, updated, skipped, and unresolved. Keep the report factual and distinguish observations from recommendations.

## Safety and Change Policy

- Inspect the project before creating or changing files.
- Never delete, reset, or overwrite user-authored files without explicit confirmation.
- Create missing directories and files only when they are part of the requested command output.
- If a target file already exists, preserve its content and make the smallest compatible update. Report what was preserved and what changed.
- **Standalone Boundary Rule**: Theme Forge operates strictly within the standalone design prototype workspace (`design/`, `docs/`, `fixtures/`, `assets/`, `AGENTS.md`, `CLAUDE.md`, `.shopifyignore`). It does NOT write into Shopify theme production directories (`sections/`, `templates/`, `layout/`, `config/`).
- The `init-agents` command modifies ONLY agent guidance files (`AGENTS.md`, `CLAUDE.md`, `.shopifyignore`); it NEVER touches, resets, or modifies existing HTML, CSS, JavaScript, fixtures, or assets.
- The `resolve` command audits and reconciles existing prototype files (`design/styles.css`, `design/script.js`, HTML templates, `docs/theme-settings.md`, `AGENTS.md`) against updated skill rules and design integrations, preserving existing design intent while eliminating hardcoded colors, refactoring scripts to OOP, stripping duplicate shells, and aligning tokens.
- Do not add dependencies, build tools, frameworks, design systems, or remote assets unless explicitly requested.
- At the end of any command, report created files, updated files, skipped files, and unresolved decisions.

## Commands

### `/theme-forge init`

Initialize a new standalone theme design workspace at the selected project root.

Before creating files, ask for:

1. **Theme name** - the merchant-facing name.
2. **Theme category** - the business or storefront category, such as fashion, beauty, food, furniture, electronics, jewelry and accessories, services, or general commerce.
3. **Theme slug** - derive a lowercase machine-readable slug from the theme name and confirm it if ambiguous.
4. **Supported languages and text directions** - default to English. RTL support is assumed unless explicitly excluded; the design must use logical CSS properties and bidirectional-safe layout so translation readiness is built in, not bolted on later.
5. **Design direction** - ask the user to describe the visual style or aesthetic direction for the theme, such as Neumorphism, Glassmorphism, Skeuomorphism, Neo-Brutalism, Swiss, Minimalism, Editorial, Y2K, Cyberpunk, Art Deco, or a custom direction. Accept free text input. This defines the visual language, not the technology.
6. **Existing project location** - use the current workspace unless the user specifies another directory.

Question formatting rules for hosts that use structured question tools:

- Never ask the project location question with only one selectable option.
- If using options, provide at least two: `Current workspace root` and `Custom path`.
- If only one practical path is available, ask as free text instead of a single-option chooser.
- If the user picks `Current workspace root`, run init there without another location prompt.
- The design direction question must always be free text input, never a choice question. The user types their answer or skips with an empty response for "none".

#### Initialization behavior

Create the smallest useful standalone design-first structure. Preserve existing user files and never overwrite an existing file without inspecting it first.

Initialization root rule: run `/theme-forge init` against the selected project root and create or update child paths from there. Do not treat `design/` as the project root.

Use this order:

1. Inspect the project root and identify existing design, docs, assets, and configuration directories.
2. Confirm the derived theme slug if it is ambiguous or conflicts with an existing project identifier.
3. Create only missing directories and files.
4. Write the settings contract using the requested identity.
5. Derive the global settings contract from the project itself: inspect existing brand assets, styles, fixtures, fonts, spacing, radii, shadows, and any pre-existing CSS or tokens before writing `docs/theme-settings.md`. The contract must cover every visual dimension the prototype will need, not a minimal starter set.
6. Create or update `.shopifyignore` in the project root to ensure Shopify CLI ignores prototype and documentation files.
7. Create or update `AGENTS.md` in the project root with agent guidelines and strict design prototype boundaries, then ensure a `CLAUDE.md` relative symbolic link points to `AGENTS.md`.
8. Summarize created, updated, skipped, and unresolved items.

```text
design/
  index.html
  404.html
  article.html
  blog.html
  cart.html
  cart-drawer.html
  collection.html
  collections-list.html
  page.html
  page.contact.html
  password.html
  product.html
  search.html
  gift_card.html
  styles.css
  script.js
  fixtures/
  assets/

docs/
  theme-settings.md

.shopifyignore
AGENTS.md
CLAUDE.md -> AGENTS.md
```

If the project already has an equivalent structure, reuse it and add only missing files. Do not create a framework, build step, component library, or design system.

Add or update `docs/theme-settings.md` with:

- Theme name, slug, category, supported locales, and direction.
- Design direction: the chosen visual style and key characteristics (e.g., soft shadows for Neumorphism, glass blur for Glassmorphism).
- The rule that global settings are designed before homepage sections and blocks.
- The selected CSS token naming convention: `--[theme-slug]-*` or an existing project convention.
- The approved page list.
- A setting inventory table with the canonical categories defined below.
- A clear marker for unresolved decisions.

---

### `/theme-forge init-agents [scope]` (Alias: `/theme-forge agents`)

Initialize or refresh agent guidance files (`AGENTS.md`, `CLAUDE.md`, `.shopifyignore`) in an existing project, scoped strictly to the standalone design prototype without altering existing design files.

This command is ideal for:
- Existing design repositories that need standardized AI agent rules and boundaries.
- Projects where `AGENTS.md` was removed, outdated, or needs realignment with Theme Forge's strict token and OOP standards.
- Monorepos or existing repositories adding a standalone `design/` prototype workspace.

#### `init-agents` Execution Steps:

1. **Project Inspection**:
   - Inspect the workspace root to detect existing directories (`design/`, `docs/`, `fixtures/`, `assets/`).
   - Detect existing theme slug or brand identifiers from `docs/theme-settings.md`, `design/styles.css`, or `package.json`.
   - Identify existing CSS custom property naming conventions (e.g. `--[theme-slug]-*`).
   - Identify existing interactive components and JS class structures in `design/script.js`.
2. **Generate / Update `AGENTS.md`**:
   - Write or update `AGENTS.md` in the project root with complete, scoped agent instructions (see `AGENTS.md` Specification below).
   - If `AGENTS.md` already exists, preserve project-specific custom sections while updating/ensuring Theme Forge design boundaries and coding standards.
3. **Establish `CLAUDE.md` Symbolic Link**:
   - Create or verify the relative symbolic link `CLAUDE.md -> AGENTS.md` in the project root (`ln -s AGENTS.md CLAUDE.md`).
4. **Configure `.shopifyignore`**:
   - Create or update `.shopifyignore` to guarantee prototype directories and agent files are ignored:
     ```text
     design/
     docs/
     AGENTS.md
     CLAUDE.md
     ```
5. **Report Summary**:
   - Report created, updated, skipped, and unresolved items. Do not modify any HTML, CSS, JS, or fixture files.

---

### `/theme-forge resolve [scope]`

Audit, reconcile, and resolve an existing theme design prototype against the latest Theme Forge standards, token architecture, zero hardcoded color rules, OOP JavaScript architecture, single shell principles, and design skill integrations.

This command is ideal for:
- Existing theme prototypes built with older conventions or incomplete token sets.
- Prototypes containing hardcoded colors (hex `#...`, `rgb()`, `rgba()`, `hsl()`, named colors) in `design/styles.css` that need migration to semantic `--[theme-slug]-color-*` tokens and color scheme groups.
- Prototypes with procedural JavaScript or loose global functions that need refactoring into ES6 OOP classes and a component registry.
- Prototypes where secondary HTML pages duplicate `<header>` and `<footer>` markup, contain inline styles/scripts, or are missing default page coverage.
- Synchronizing `docs/theme-settings.md` with current CSS custom properties and prototype features.
- Upgrading existing prototypes with visual refinement and modern web best practices when `frontend-design` is available.

#### `resolve` Execution Steps:

1. **Workspace Inspection & Audit**:
   - Inspect the workspace root and verify the prototype layout (`design/`, `docs/`, `fixtures/`, `assets/`).
   - Detect existing theme identity (theme name, slug, category, design direction) from `docs/theme-settings.md`, `design/styles.css`, or HTML titles. If ambiguous, confirm with the user.
   - Run an audit across all files:
     - **CSS Audit**: Detect hardcoded color literals (`#...`, `rgb()`, `hsl()`, named colors), missing 9-category tokens, missing color scheme scopes (`[data-color-scheme="..."]`), and physical directional properties (`left`, `right`, `margin-left`, etc.).
     - **JS Audit**: Detect loose procedural code, global functions, missing class encapsulation, unmanaged lifecycles, or missing component registry.
     - **HTML Audit**: Detect duplicate `<header>`/`<footer>` markup in secondary pages, inline styles (`style="..."`), inline scripts/handlers (`onclick="..."`), and missing default Shopify page prototypes.
     - **Docs Audit**: Check `docs/theme-settings.md` completeness across the 9 canonical categories and token alignment.

2. **Resolve Styles (`design/styles.css`)** (Scope: `all` or `styles`):
   - **Zero Hardcoded Colors**: Extract and replace all hardcoded color literals with semantic CSS custom properties (`--[theme-slug]-color-*`).
   - **Color Schemes**: Structure tokens under root and scoped selectors (`:root`, `[data-color-scheme="primary"]`, `[data-color-scheme="secondary"]`, `[data-color-scheme="contrast"]`) for background, text, buttons, borders, links, icons, and badges.
   - **9 Canonical Categories**: Ensure full token definitions for Brand, Colors, Typography (Display, Heading, Body, Accent), Buttons & Radii, Product Cards, Layout & Spacing, Cart, Search & Discovery, and Accessibility/Prose.
   - **Logical CSS & RTL Readiness**: Convert directional physical properties to logical properties (`margin-inline-start`/`end`, `padding-inline-start`/`end`, `inset-inline-start`/`end`, `text-align: start`/`end`, border-radius logical properties).

3. **Resolve JavaScript Architecture (`design/script.js`)** (Scope: `all` or `scripts`):
   - Refactor all interactive functionality into modular ES6 classes (e.g. `HeaderController`, `CartDrawer`, `ProductGallery`, `VariantSelector`, `PredictiveSearch`, `FilterDrawer`, `ModalManager`, `Accordion`).
   - Implement standardized lifecycle methods on each class:
     - `constructor(element, options)`: Store DOM references and initial state.
     - `init()`: Set up ARIA states, observers, and call `bindEvents()`.
     - `bindEvents()`: Attach event listeners with proper `this` binding and delegation.
     - `destroy()`: Clean up listeners and mutations.
   - Implement a central `ThemeApp` / `ComponentRegistry` coordinator instantiated on `DOMContentLoaded`.
   - Eliminate all floating global functions, top-level event listeners, and procedural scripting.

4. **Resolve HTML Prototype Templates** (Scope: `all` or `templates`):
   - **Single Header & Footer Rule**: Ensure the full global `<header>` and `<footer>` live strictly in `design/index.html`. Strip duplicated header and footer markup from secondary pages (`product.html`, `collection.html`, `cart.html`, `404.html`, etc.), retaining only page-specific `<main>` content.
   - **Strip Inline Code**: Remove all inline `style="..."` attributes and inline event handlers (`onclick`, `onchange`, etc.). Relocate styles to `design/styles.css` and event handlers to `design/script.js` classes.
   - **Default Page Completeness**: Verify all 14 default Shopify prototype pages are present (`index.html`, `404.html`, `article.html`, `blog.html`, `cart.html`, `cart-drawer.html`, `collection.html`, `collections-list.html`, `page.html`, `page.contact.html`, `password.html`, `product.html`, `search.html`, `gift_card.html`). Create minimal accessible scaffolds (`data-prototype-status="scaffold"`) for any missing pages.
   - **Accessibility & Semantics**: Ensure semantic HTML5 elements (`<main>`, `<nav>`, `<article>`, `<section>`, `<dialog>`, `<details>`), form labels, ARIA roles, and keyboard focus states.

5. **Reconcile Global Settings Contract (`docs/theme-settings.md`)** (Scope: `all` or `docs`):
   - Synchronize `docs/theme-settings.md` with resolved CSS tokens and prototype capabilities across all 9 canonical categories.
   - Ensure setting tables specify Shopify-safe `snake_case` IDs, control types, defaults, ranges, token mappings, scopes, and wiring statuses.
   - Document the theme's visual direction and key characteristics.

6. **Design Skill & Frontend Polish Integration**:
   - If `frontend-design` or modern web guidance is present, apply refined visual aesthetics, harmonious typography scales, balanced spacing, cohesive micro-animations, and interactive hover/active states consistent with the theme's design direction while strictly obeying the zero hardcoded colors and single stylesheet rules.

7. **Agent Guidance & Boundary Synchronization**:
   - Update `AGENTS.md` in the project root with the resolved theme tokens, OOP architecture, and strict prototype boundaries.
   - Ensure the relative symbolic link `CLAUDE.md -> AGENTS.md` is valid.
   - Ensure `.shopifyignore` contains entries for `design/`, `docs/`, `AGENTS.md`, and `CLAUDE.md`.

8. **Completion Report**:
   - Provide a factual summary separating created, updated, skipped, and unresolved items, highlighting rule violations resolved (e.g. number of hardcoded colors tokenized, classes created, secondary page headers stripped).

#### `.shopifyignore` Configuration

Create `.shopifyignore` in the project root if it does not exist, or append missing entries to an existing `.shopifyignore` so that Shopify CLI commands (`shopify theme dev`, `shopify theme push`, `shopify theme check`) strictly ignore prototype, documentation, and agent files:

```text
# Theme Forge Prototype & Documentation
design/
docs/
AGENTS.md
CLAUDE.md
```

#### `AGENTS.md` and `CLAUDE.md` Symbolic Link Specification

Create `AGENTS.md` in the project root if it does not exist, or update an existing `AGENTS.md` by preserving existing instructions and appending/merging the Theme Forge conventions. Create a symbolic link `CLAUDE.md` pointing to `AGENTS.md` (`ln -s AGENTS.md CLAUDE.md`) if missing.

`AGENTS.md` serves as persistent guidance for coding agents working on the standalone theme prototype. It must contain:

1. **Theme Identity & Overview**: Theme name, slug, category, supported languages, RTL direction requirements, and chosen design direction.
2. **Design-First Architecture & Boundaries**:
   - Visual prototyping is strictly confined to `design/`.
   - Standalone execution: no Liquid, Shopify API endpoints, build systems, CSS frameworks, or JavaScript libraries in `design/`.
   - Clean, semantic HTML5, vanilla CSS with CSS custom properties (`--[theme-slug]-*`), and vanilla Object-Oriented JavaScript (OOP) for client interactions.
   - Single stylesheet and script rule: all styles across the prototype must live strictly in the single shared CSS file (`design/styles.css`) and all JavaScript in the single shared JS file (`design/script.js`). Never use inline CSS (`style` attributes or `<style>` blocks) or inline JavaScript (event handlers like `onclick` or inline `<script>` blocks) inside HTML pages.
   - **Strict Zero Hardcoded Colors Rule**: `design/styles.css` must NEVER contain hardcoded color literals (no hex `#...`, `rgb()`, `rgba()`, `hsl()`, `hsla()`, or named colors like `white`, `black`, `red`). Every single color value across all components, utilities, and states must strictly reference semantic CSS variables / tokens (`var(--[theme-slug]-color-*)`).
   - **Strict Object-Oriented JavaScript (OOP) Rule**: `design/script.js` must strictly follow an Object-Oriented Programming architecture using ES6 classes. Every interactive component/feature must be encapsulated in a class (e.g., `HeaderController`, `CartDrawer`, `ProductGallery`, `VariantSelector`) with standardized lifecycle methods (`constructor`, `init`, `bindEvents`, `destroy`) and coordinated via a central component registry/app bootstrap. No loose procedural scripts or global floating functions.
   - **Single Header & Footer Rule**: The global `<header>` (announcement bar, navigation, logo, cart trigger, etc.) and `<footer>` (links, newsletter, copyright, disclosures) must **strictly only be developed in `design/index.html`** (the shared shell). Never duplicate full header and footer markup in secondary prototype pages (`product.html`, `collection.html`, `cart.html`, etc.); secondary pages contain only their page-specific `<main>` template content to prevent duplication and maintenance divergence.
   - Bidirectional-safe layout with logical CSS properties (`inline-start`/`inline-end`).
3. **Workspace Directory Map**:
   - `design/`: Isolated pure HTML/CSS/JS prototypes, styles, scripts, fixtures.
   - `docs/`: `theme-settings.md` (global settings contract).
4. **Theme Forge Commands**:
   - `/theme-forge init`: Initialize or update prototype workspace and global settings contract.
   - `/theme-forge init-agents` (alias `/theme-forge agents`): Initialize or refresh agent instructions and `.shopifyignore` for existing projects scoped strictly to the design prototype.
   - `/theme-forge resolve [scope]`: Audit, reconcile, and resolve existing prototype styles, scripts, templates, settings contracts, and design integrations against Theme Forge standards.
5. **Modification & Safety Rules**:
   - Never overwrite user files without confirmation.
   - Do not create or modify files outside the standalone prototype workspace (`design/`, `docs/`, `fixtures/`, `assets/`, `AGENTS.md`, `CLAUDE.md`, `.shopifyignore`).
   - Keep settings documented in `docs/theme-settings.md` synchronized with CSS tokens.
   - Route all colors strictly through documented color scheme roles and `--[theme-slug]-color-*` tokens; never introduce hard-coded colors or per-component color values in `design/styles.css`.

For the `CLAUDE.md` symbolic link:
- Create it in the project root pointing to `AGENTS.md` via `ln -s AGENTS.md CLAUDE.md` or equivalent filesystem operation.
- If a symlink or file named `CLAUDE.md` already exists, preserve it if it points to `AGENTS.md` or report it under `skipped` / `updated`.
- If symbolic link creation fails (e.g., due to filesystem limitations), report it under `unresolved` with manual creation instructions.

Do not create empty HTML files that imply a finished design. If a page prototype does not exist, create a minimal accessible scaffold with a clear `data-prototype-status="scaffold"` marker, or record the missing page in `docs/theme-settings.md` when the user asks for documentation only.

Do not build homepage sections or blocks during initialization. `design/index.html` should establish the shared shell only. Homepage composition is a separate design task after the global settings and default page layouts are credible.

#### Init design rules

The prototype must use:

- Plain HTML files, exactly one shared CSS file (`design/styles.css`), and exactly one shared JavaScript file (`design/script.js`).
- Single stylesheet rule: every style across the prototype must reside in the single shared CSS file (`design/styles.css`). Never write or inject inline CSS (using `style` attributes or `<style>` blocks) or inline JavaScript (using event attributes like `onclick` or inline `<script>` blocks) inside any HTML page.
- **Strict Zero Hardcoded Colors Rule**: `design/styles.css` must NEVER contain any hardcoded color literals (hex, rgb, rgba, hsl, or named colors). Every color property (`color`, `background-color`, `border-color`, `box-shadow`, `fill`, `stroke`, etc.) must strictly reference semantic CSS custom properties (`var(--[theme-slug]-color-*)`).
- Single header and footer source of truth: Develop the global `<header>` and `<footer>` **strictly in `design/index.html`** (the shared shell). Never duplicate header or footer markup in secondary prototype pages (`product.html`, `collection.html`, `cart.html`, `404.html`, etc.). All other prototype pages focus purely on their page-specific `<main>` content to eliminate duplication and maintenance overhead.
- Plain CSS only — no CSS frameworks, preprocessors, or utility libraries (no Tailwind, Bootstrap, etc.).
- **Vanilla JavaScript only (Object-Oriented / OOP)** — structured strictly using Object-Oriented Programming with ES6 classes, encapsulated component controllers, explicit lifecycles, and a clean component registry; no loose procedural scripts, no global floating functions, and no JS frameworks, libraries, or build tools (no React, Alpine, GSAP, etc.).
- Static fixtures that resemble future Shopify data but contain no Liquid or Shopify API calls.
- Stable semantic classes and `data-*` hooks that can survive Liquid conversion.
- CSS custom properties for global visual roles.
- Native HTML behavior first: forms, links, buttons, `details`, and `dialog` where appropriate.
- Progressive enhancement: important content and forms remain understandable without JavaScript.
- Responsive, keyboard, focus-visible, reduced-motion, empty, error, loading, unavailable, and missing-media states.
- Labels as well as clear placeholders in every prototype form.
- Logical CSS properties and bidirectional-safe layout by default. The design must be structurally ready for RTL via `inline-start`/`inline-end` properties, mirrored icons where needed, and no hard-coded left/right assumptions. This is a design-time concern, not a translation concern — do not add translated content or alternate layouts.

Do not hard-code a particular aesthetic. The category can influence sample content and fixture shape, but it must not force colors, fonts, layout, or design beyond what the user specified.

The chosen design direction shapes the visual language only:

- Neumorphism: soft extruded shadows, subtle depth, muted color palettes.
- Glassmorphism: frosted glass blur, translucency, layered depth.
- Skeuomorphism: realistic textures, lighting, dimensional detail.
- Neo-Brutalism: bold borders, raw typography, high contrast, visible structure.
- Swiss: grid-driven, clean typography, restrained hierarchy.
- Minimalism: whitespace-forward, minimal chrome, restrained decoration.
- Editorial: strong typographic hierarchy, magazine-inspired layout.
- Y2K: glossy surfaces, bold gradients, playful maximalism.
- Cyberpunk: neon accents, dark backgrounds, glitch/tech aesthetics.
- Art Deco: geometric patterns, metallic accents, ornamental detail.
- Custom: the user defines the direction; capture it in `docs/theme-settings.md`.

Record the chosen direction in `docs/theme-settings.md` under a Design Direction section with key visual characteristics. Do not install or reference any framework, library, or build tool.

## Global Settings Contract

`docs/theme-settings.md` is the primary output of `/theme-forge init`. It must describe settings by their design effect, not by HTML structure.

The contract must be robust and project-derived:

- Derive every setting from what the project actually contains — its design direction, category, fixtures, and visual language — not from a generic template.
- The contract must be complete enough that a merchant could restyle the entire storefront (brand, colors, typography, layout, components, commerce surfaces) without touching code. If a prototype behavior or visual role exists but has no corresponding setting or documented decision to keep it code-level, the contract is incomplete.
- Cover all recommended global groups below; when a group does not apply to this project's category, record it as `not-applicable` with a reason instead of silently omitting it.
- Every default page in the coverage table must be traceable to at least one global group that affects it.
- Keep the contract forward-compatible with Shopify's `config/settings_schema.json`: stable IDs, control types, defaults, and ranges must be expressible as a real theme editor schema.

Every proposed setting needs:

- Stable Shopify-safe ID using `snake_case`.
- Category or group.
- Merchant-facing label.
- Shopify control type candidate.
- Allowed values or range.
- Prototype default.
- CSS token or visible effect.
- Scope: global, template, section, or block.
- Fallback when unset or unsupported.
- Wiring status: pending, native, Liquid, schema, AJAX, app-dependent, metafield-dependent, or implementation-only.

### The Nine Canonical Global Settings Categories

The global theme settings contract consolidates all global storefront controls into **nine non-overlapping categories**. Typography drops redundant per-element font pickers in favor of **four reusable type roles** (`display`, `heading`, `body`, `accent`), and colors route entirely through Shopify's native `color_scheme_group` system.

Every setting must have a corresponding CSS custom property using the `--[theme-slug]-*` namespace so that static prototype styles map 1:1 to theme settings and Liquid schema tokens.

#### 1. Brand (`--[theme-slug]-brand-*`)
- **Logo** (image picker)
- **Logo — Dark Mode variant** (image picker)
- **Logo width** (range in px) -> `--[theme-slug]-brand-logo-width`
- **Favicon** (image picker)
- **Social accounts** (text inputs for Instagram, Facebook, TikTok, Pinterest, YouTube, X, LinkedIn, Snapchat, Tumblr, Vimeo)

#### 2. Color System (`--[theme-slug]-color-*`)
- **Color Scheme Group** — one native Shopify `color_scheme_group` defined once in `settings_schema.json`. Every scheme carries definition inputs for: background (solid or gradient), text, primary button, secondary button, primary button border, secondary button border, on-primary button text, on-secondary button text, links, and icons. Default schemes: **Primary, Secondary, Contrast**.
  - Scheme CSS Variables (scoped to `[data-color-scheme="..."]` or `:root`):
    - `--[theme-slug]-color-background`
    - `--[theme-slug]-color-background-gradient`
    - `--[theme-slug]-color-text`
    - `--[theme-slug]-color-primary-button`
    - `--[theme-slug]-color-primary-button-border`
    - `--[theme-slug]-color-on-primary-button`
    - `--[theme-slug]-color-secondary-button`
    - `--[theme-slug]-color-secondary-button-border`
    - `--[theme-slug]-color-on-secondary-button`
    - `--[theme-slug]-color-links`
    - `--[theme-slug]-color-icons`
- **Shared Color Scheme control**: Any section, block, card, drawer, or modal selects a scheme via a `color_scheme` type setting rather than storing individual color pickers.
- **Badge Colors**:
  - Sale Badge Colors (background / text) -> `--[theme-slug]-color-badge-sale-bg`, `--[theme-slug]-color-badge-sale-text`
  - Sold Out Badge Colors (background / text) -> `--[theme-slug]-color-badge-soldout-bg`, `--[theme-slug]-color-badge-soldout-text`
  - Custom Badge Colors (background / text; paired with `badge.custom` metafield) -> `--[theme-slug]-color-badge-custom-bg`, `--[theme-slug]-color-badge-custom-text`

#### 3. Typography System (`--[theme-slug]-type-[role]-*`)
Four **type roles** replace fragmented font pickers. Every heading, label, button, and body text is assigned one role by default:
- **Display**: Hero banner, slideshow, large marketing headlines
- **Heading**: Section headings, page titles, H1–H6
- **Body**: Paragraphs, descriptions, long-form content
- **Accent**: Eyebrows, buttons, badges, nav labels, product card meta (short, uppercase-leaning UI text)

Each role exposes and maps to CSS custom properties:
- Font family -> `--[theme-slug]-type-[role]-font-family`
- Size Scale (`XS` – `XL`, five steps) -> `--[theme-slug]-type-[role]-font-size`
- Letter Spacing -> `--[theme-slug]-type-[role]-letter-spacing`
- Line Height -> `--[theme-slug]-type-[role]-line-height`
- Text Case (`none`, `uppercase`, `lowercase`, `capitalize`) -> `--[theme-slug]-type-[role]-text-transform`

#### 4. Buttons & Corners (`--[theme-slug]-radius-*` & `--[theme-slug]-button-*`)
- **Default Button Style**: Solid, Outline, Text
- **Show arrow icon by default**: boolean
- **Corner Radius Preset**: Sharp (`0px`), Soft (`4px`–`8px`), Round (`12px`–`16px`), Pill (`9999px`) applied to buttons, cards, media, inputs, and popups:
  - `--[theme-slug]-radius-base`
  - `--[theme-slug]-radius-button`
  - `--[theme-slug]-radius-card`
  - `--[theme-slug]-radius-media`
  - `--[theme-slug]-radius-input`
  - `--[theme-slug]-radius-popup`
- **Advanced overrides**: toggle to independently adjust button and media radii.

#### 5. Product Card (`--[theme-slug]-product-card-*`)
Single source of truth for product card presentation everywhere (grids, search, cart upsell, recommendations):
- Aspect Ratio (Portrait, Square, Landscape) -> `--[theme-slug]-product-card-aspect-ratio`
- Image Fit (Cover, Contain) -> `--[theme-slug]-product-card-image-fit`
- Show secondary image on hover
- Enable hover effect
- Show vendor
- Show SKU
- Show collection label
- Show color swatches (from variants) + swatch size -> `--[theme-slug]-product-card-swatch-size`
- Show product rating (reads from reviews app or `reviews.rating` metafield)
- Sale Badge (show, format: Text only / Text + percentage)
- Sold Out Badge (show)
- Custom Badge (show, requires `badge.custom` single-line-text metafield)

#### 6. Layout & Spacing (`--[theme-slug]-layout-*` & `--[theme-slug]-spacing-*`)
- **Page Width**: theme's max content width -> `--[theme-slug]-page-width`
- **Section Spacing Scale**: defines pixel values for `Tight`, `Default`, `Loose`, `Extra Loose` -> `--[theme-slug]-spacing-section` (consumed by all section Space Above / Space Below controls)
- **Media Corner Radius override**: overrides `--[theme-slug]-radius-media` when enabled

#### 7. Cart (`--[theme-slug]-cart-*`)
- **Cart Behavior**: Drawer or Page
- **Open drawer automatically on add to cart**: boolean
- **Drawer: show order note field**: boolean
- **Drawer: show discount code field**: boolean
- **Free Shipping Progress Bar**: show, and threshold amount -> `--[theme-slug]-cart-free-shipping-threshold`
- **Cart Upsell**: heading, product source (rendered via shared Product Card component)

#### 8. Search & Discovery (`--[theme-slug]-discovery-*` / `--[theme-slug]-search-*`)
- **Quick Actions**: action on card (`none`, `quick_view`, `quick_add`), button style, visibility (`always`, `hover`), enabled on mobile
- **Search Modal**: enable boolean
- **Predictive Results**: enable, show product / collection / article suggestions
- **Empty Search Suggestions**: heading, curated product list
- **Pagination Style**: Numbered Pages, Load More Button, Infinite Scroll
- **Load More Button Style**: button style variant
- **Recently Viewed**: tracking duration (days), track after Quick View

#### 9. Accessibility & Behavior (`--[theme-slug]-prose-*` & `--[theme-slug]-behavior-*`)
- **Show back-to-top button**: boolean
- **Show breadcrumbs**: boolean
- **Open external links in a new tab**: boolean
- **Prose Style**: Custom, Standard (governs long-form rich text typography) -> `--[theme-slug]-prose-style`

### CSS Variable & Token Architecture Rules
1. **1:1 Alignment**: Every visual setting in the 9 categories must have an exact matching CSS custom property in `design/styles.css`.
2. **Single Stylesheet**: All tokens, utility classes, and component rules reside strictly in `design/styles.css`.
3. **No Inline Styling**: HTML templates must reference semantic classes or CSS custom properties—never inline `style="..."` or `<style>` blocks.
4. **Color Scheme Scoping**: Color variables must be scoped to `[data-color-scheme="primary"]`, `[data-color-scheme="secondary"]`, `[data-color-scheme="contrast"]` so components simply use `var(--[theme-slug]-color-background)`, etc., and react automatically to scheme changes.
5. **Strict Zero Hardcoded Colors**: `design/styles.css` must **NEVER** contain hardcoded colors anywhere (no hex codes `#...`, `rgb()`, `rgba()`, `hsl()`, or named CSS colors like `black`, `white`, `red`). Every background, text, border, outline, shadow color, SVG fill/stroke, and badge color must strictly resolve through `var(--[theme-slug]-color-*)`.

### JavaScript Architecture Rules (Object-Oriented / OOP)

All interactive behaviors and client logic in `design/script.js` (and subsequent theme JavaScript) must strictly adhere to Object-Oriented Programming (OOP) principles:

1. **ES6 Class Encapsulation**:
   - Every interactive component or UI feature must be encapsulated in a dedicated ES6 class (e.g., `HeaderController`, `CartDrawer`, `ProductGallery`, `VariantSelector`, `PredictiveSearch`, `FilterDrawer`, `ModalManager`, `Accordion`, `ToastNotification`, `QuickView`).
   - Classes must manage their own DOM references, component state, event handlers, and ARIA accessibility attributes without leaking variables into the global scope.

2. **Standardized Lifecycle Methods**:
   - `constructor(element, options = {})`: Stores DOM element references, merges options/defaults, and initializes component state.
   - `init()`: Sets up initial DOM states, creates mutation observers or ARIA attributes, and invokes `this.bindEvents()`.
   - `bindEvents()`: Attaches event listeners with bound execution contexts (`this.handleEvent = this.handleEvent.bind(this)` or arrow class methods) and leverages event delegation where appropriate.
   - `destroy()`: Cleans up all attached event listeners, observers, and DOM mutations when a component is dismantled or reloaded.

3. **Component Registry & Auto-Initialization Bootstrap**:
   - A central coordinator class or registry (e.g., `ThemeApp` or `ComponentRegistry`) discovers components via semantic DOM hooks or `data-component="..."` attributes on `DOMContentLoaded` and instantiates corresponding controller instances.
   - Example OOP pattern:
     ```javascript
     class CartDrawer {
       constructor(rootElement) {
         this.drawer = rootElement;
         this.openButtons = document.querySelectorAll('[data-cart-drawer-trigger]');
         this.closeButtons = this.drawer.querySelectorAll('[data-cart-drawer-close]');
         this.isOpen = false;
         this.init();
       }

       init() {
         this.bindEvents();
       }

       bindEvents() {
         this.openButtons.forEach(btn => btn.addEventListener('click', () => this.open()));
         this.closeButtons.forEach(btn => btn.addEventListener('click', () => this.close()));
         this.drawer.addEventListener('keydown', (e) => {
           if (e.key === 'Escape' && this.isOpen) this.close();
         });
       }

       open() {
         this.isOpen = true;
         this.drawer.setAttribute('aria-hidden', 'false');
         this.drawer.classList.add('is-open');
         document.body.classList.add('overflow-hidden');
       }

       close() {
         this.isOpen = false;
         this.drawer.setAttribute('aria-hidden', 'true');
         this.drawer.classList.remove('is-open');
         document.body.classList.remove('overflow-hidden');
       }

       destroy() {
         // Cleanup listeners
       }
     }

     class ThemeApp {
       constructor() {
         this.components = new Map();
       }

       init() {
         const drawerEl = document.querySelector('[data-component="cart-drawer"]');
         if (drawerEl) this.components.set('cartDrawer', new CartDrawer(drawerEl));
         // Register and instantiate additional OOP controllers...
       }
     }

     document.addEventListener('DOMContentLoaded', () => {
       window.themeApp = new ThemeApp();
       window.themeApp.init();
     });
     ```

4. **Zero Loose Procedural Scripts**:
   - Never write loose global scripts, top-level event listeners, or inline scripts. All logic must belong to a class and be instantiated cleanly.


## Default Page Coverage

The initialization must account for these Shopify page experiences:

| Prototype file          | Shopify target          | Required design concern                                                     |
| ----------------------- | ----------------------- | --------------------------------------------------------------------------- |
| `404.html`              | `404.json`              | Recovery, search/discovery, and not-found state                             |
| `article.html`          | `article.json`          | Article header, metadata, media, prose, sharing, related content            |
| `blog.html`             | `blog.json`             | Blog listing, pagination/load-more, empty state                             |
| `cart.html`             | `cart.json`             | Lines, quantities, removal, notes, discounts, totals, checkout, empty state |
| `cart-drawer.html`      | Section via AJAX        | Slide-out drawer, same cart data, progressive enhancement, overlay behavior |
| `collection.html`       | `collection.json`       | Collection header, product listing, filtering, sorting, empty state         |
| `collections-list.html` | `collections-list.json` | Collection directory, card defaults, pagination/empty state                 |
| `index.html`            | `index.json`            | Shared shell only until homepage sections are designed                      |
| `page.html`             | `page.json`             | Standard page title and rich content                                        |
| `page.contact.html`     | `page.contact.json`     | Contact fields, consent, validation, and success state                      |
| `password.html`         | `password.json`         | Password message, access form, errors, and brand treatment                  |
| `product.html`          | `product.json`          | Media, title, price, options, quantity, add-to-cart, unavailable state      |
| `search.html`           | `search.json`           | Search input, result types, filtering/sorting, no-results state             |
| `gift_card.html`        | `gift_card.liquid`      | Gift-card value/code, barcode or QR, print, and wallet actions              |

These are page contracts, not a section and block catalog. Do not invent homepage sections during `/theme-forge init`.

## Shopify Liquid Reference Objects for Default Pages

When designing prototypes and fixtures, include field coverage for each page's primary Liquid objects so the visual contract can accommodate real Shopify data shapes.

Rules:

- Treat this section as a fixture and UI coverage checklist, not a requirement to display every field in the final interface.
- Include all listed fields in fixture data models for the matching page.
- If a field is not surfaced in UI, still account for it in behavior, states, or documentation.
- Do not invent unsupported fields; when Shopify docs and existing theme code disagree, mark the mismatch as `unresolved` in `docs/shopify-wiring.md`.

### Product page (`product.html` -> `product.json`)

Primary object: `product`

Use this baseline field checklist in fixtures and design mapping:

```json
{
  "available": true,
  "category": {},
  "collections": [],
  "compare_at_price": "25.00",
  "compare_at_price_max": "25.00",
  "compare_at_price_min": "25.00",
  "compare_at_price_varies": false,
  "content": "<h3>...</h3>",
  "created_at": "2022-04-13 14:46:16 -0400",
  "description": "<h3>...</h3>",
  "featured_image": {},
  "featured_media": {},
  "first_available_variant": {},
  "gift_card?": false,
  "handle": "health-potion",
  "has_only_default_variant": false,
  "id": 6786188247105,
  "images": [],
  "media": [],
  "metafields": {},
  "options": ["Size", "Strength"],
  "options_by_name": {},
  "options_with_values": [],
  "price": "10.00",
  "price_max": "22.00",
  "price_min": "10.00",
  "price_varies": true,
  "published_at": "2022-04-13 14:53:34 -0400",
  "quantity_price_breaks_configured?": false,
  "requires_selling_plan": false,
  "selected_or_first_available_selling_plan_allocation": {},
  "selected_or_first_available_variant": {},
  "selected_selling_plan": null,
  "selected_selling_plan_allocation": null,
  "selected_variant": null,
  "selling_plan_groups": [],
  "tags": ["healing"],
  "template_suffix": "",
  "title": "Health potion",
  "type": {},
  "url": {},
  "variants": [],
  "variants_count": 9,
  "vendor": "Polina's Potent Potions"
}
```

### Collection page (`collection.html` -> `collection.json`)

Primary objects: `collection`, `collection.products`, `paginate`, `current_tags`, `sort_by`

Use this baseline field checklist in fixtures and design mapping:

```json
{
  "all_products_count": 10,
  "all_tags": [
    "Burning",
    "dried",
    "extracts",
    "fresh",
    "ingredients",
    "plant",
    "supplies"
  ],
  "all_types": [
    "Animals & Pet Supplies",
    "Baking Flavors & Extracts",
    "Cooking & Baking Ingredients",
    "Dried Flowers",
    "Fruits & Vegetables",
    "Seasonings & Spices",
    "Water"
  ],
  "all_vendors": [
    "Clover's Apothecary",
    "Polina's Potent Potions",
    "Ted's Apothecary Supply"
  ],
  "current_type": null,
  "current_vendor": null,
  "default_sort_by": "created-ascending",
  "description": "Brew your own potions at home using our fresh, ethically-sourced ingredients.",
  "featured_image": {},
  "filters": {},
  "handle": "ingredients",
  "id": 266168401985,
  "image": {},
  "metafields": {},
  "next_product": null,
  "previous_product": null,
  "products": {},
  "products_count": 1,
  "published_at": "2022-04-19 09:52:18 -0400",
  "sort_by": "",
  "sort_options": [],
  "tags": ["Burning"],
  "template_suffix": "eight-products-per-page",
  "title": "Ingredients",
  "url": {}
}
```

Field groups to cover:

- Collection identity: title, description, handle, url, image, featured_image, template_suffix.
- Merchandising metadata: products_count, all_tags, current_tags, default_sort_by, sort_options.
- Product listing behavior: product card data requirements, empty collection state, unavailable products.
- Filtering and sorting state: applied filters, clear-all behavior, zero-result behavior.
- Pagination: current page, total pages, next/previous behavior and fallback.

### Collections directory (`collections-list.html` -> `collections-list.json`)

Primary objects: `collections`, `collection` items, optional `paginate`

Use this baseline field checklist in fixtures and design mapping:

```json
[
  {
    "id": 266168401985,
    "handle": "potions",
    "title": "Potions & Elixirs",
    "description": "Handcrafted magical brews for every affliction and adventure.",
    "products_count": 12,
    "url": "/collections/potions",
    "image": {
      "src": "https://cdn.shopify.com/s/files/1/0000/0001/collections/potions.jpg",
      "alt": "Row of glass potion vials",
      "width": 1200,
      "height": 800
    },
    "featured_image": {
      "src": "https://cdn.shopify.com/s/files/1/0000/0001/collections/potions.jpg",
      "alt": "Row of glass potion vials",
      "width": 1200,
      "height": 800
    }
  },
  {
    "id": 266168434753,
    "handle": "ingredients",
    "title": "Raw Ingredients",
    "description": "Ethically sourced herbs, minerals, and dried botanicals.",
    "products_count": 8,
    "url": "/collections/ingredients",
    "image": {
      "src": "https://cdn.shopify.com/s/files/1/0000/0001/collections/ingredients.jpg",
      "alt": "Dried herbs and crystals in wooden bowls",
      "width": 1200,
      "height": 800
    },
    "featured_image": {
      "src": "https://cdn.shopify.com/s/files/1/0000/0001/collections/ingredients.jpg",
      "alt": "Dried herbs and crystals in wooden bowls",
      "width": 1200,
      "height": 800
    }
  }
]
```

Field groups to cover:

- Directory card basics: title, description excerpt, image, products_count, url, handle.
- Ordering and visibility: sort strategy, hidden/empty collections, fallback media.
- Empty and sparse states: no collections, missing image, long titles.

### Cart page (`cart.html` -> `cart.json`)

Primary objects: `cart`, `cart.items`, `line_item`, `routes`

Use this baseline field checklist in fixtures and design mapping:

```json
{
  "attributes": {},
  "cart_level_discount_applications": [],
  "checkout_charge_amount": "380.25",
  "currency": {},
  "discount_applications": [],
  "discounts": [],
  "duties_included": false,
  "empty?": false,
  "item_count": 2,
  "items": [],
  "items_subtotal_price": "422.49",
  "note": "Hello this is a note",
  "original_total_price": "380.25",
  "requires_shipping": true,
  "taxes_included": false,
  "total_discount": "44.74",
  "total_price": "380.25",
  "total_weight": 0
}
```

Line item object (`cart.items`):

```json
{
  "discount_allocations": [],
  "discounts": [],
  "error_message": "",
  "final_line_price": "74.97",
  "final_price": "24.99",
  "fulfillment": {},
  "fulfillment_service": "manual",
  "gift_card": false,
  "grams": 0,
  "id": 10974183882817,
  "image": {},
  "instructions": null,
  "item_components": null,
  "key": 10974183882817,
  "line_level_discount_allocations": [],
  "line_level_total_discount": "0.00",
  "line_price": "74.97",
  "message": "",
  "options_with_values": [
    {
      "name": "Title",
      "value": "Default Title"
    }
  ],
  "original_line_price": "74.97",
  "original_price": "24.99",
  "parent_relationship": null,
  "price": "24.99",
  "product": {},
  "product_id": 6792596455489,
  "properties": {},
  "quantity": 3,
  "requires_shipping": true,
  "selling_plan_allocation": null,
  "sku": "",
  "successfully_fulfilled_quantity": 2,
  "tax_lines": [],
  "taxable": true,
  "title": "Bloodroot (whole)",
  "total_discount": "0.00",
  "unit_price": "49.98",
  "unit_price_measurement": {
    "measured_type": "weight",
    "quantity_value": "500.0",
    "quantity_unit": "g",
    "reference_value": 1,
    "reference_unit": "kg"
  },
  "url": {},
  "url_to_remove": null,
  "variant": {},
  "variant_id": 39888235757633,
  "vendor": "Clover's Apothecary"
}
```

Field groups to cover:

- Cart summary: item_count, total_price, original_total_price, total_discount, checkout_charge_amount, items_subtotal_price, note, currency, empty?, requires_shipping, taxes_included, duties_included, total_weight.
- Cart discounts: cart_level_discount_applications, discount_applications, discounts.
- Line items: key, id, product_id, variant_id, title, url, image, variant, quantity, properties, options_with_values, price, original_price, final_price, line_price, original_line_price, final_line_price, total_discount, line_level_total_discount, discounts, discount_allocations, line_level_discount_allocations.
- Line item product context: product, vendor, sku, gift_card, taxable, requires_shipping, unit_price, unit_price_measurement.
- Fulfillment: fulfillment, fulfillment_service, successfully_fulfilled_quantity, instructions, item_components.
- Line item messaging: error_message, message.
- Commerce actions: quantity updates, removal, notes, checkout path, continue shopping path, url_to_remove.
- Edge states: empty cart, sold out, invalid quantity, discount conflicts, parent_relationship, selling_plan_allocation.

### Cart drawer (section, not a template)

Primary objects: same as cart page (`cart`, `cart.items`, `line_item`), plus `routes`

The cart drawer is a slide-out panel rendered as a Shopify section, not a standalone template. It shares the same Liquid data objects as the cart page but is loaded via AJAX into a global container.

Use this baseline field checklist in fixtures and design mapping (same cart/line_item objects as above):

```json
{
  "attributes": {},
  "cart_level_discount_applications": [],
  "checkout_charge_amount": "380.25",
  "currency": {},
  "discount_applications": [],
  "discounts": [],
  "duties_included": false,
  "empty?": false,
  "item_count": 2,
  "items": [],
  "items_subtotal_price": "422.49",
  "note": "Hello this is a note",
  "original_total_price": "380.25",
  "requires_shipping": true,
  "taxes_included": false,
  "total_discount": "44.74",
  "total_price": "380.25",
  "total_weight": 0
}
```

Field groups to cover:

- Data parity with cart page: same `cart` and `line_item` fields apply.
- Drawer behavior: open/close triggers, overlay/backdrop, focus trap, ESC to close, body scroll lock.
- AJAX loading: section-render-path (`cart` section ID), update targets, variant ID posting, quantity change endpoints.
- Empty state: drawer-specific empty messaging, continue shopping link.
- Progressive enhancement: works without JS via full cart page fallback.
- Accessibility: `role="dialog"`, `aria-modal`, `aria-label`, `aria-live` for count/total updates, focus management.
- Edge states: sold-out items, invalid quantity, pending state during AJAX, error rollback.
- Design concerns: overlay opacity, transition animation, z-index stacking, responsive width, mobile full-screen variant.

### Blog page (`blog.html` -> `blog.json`)

Primary objects: `blog`, `blog.articles`, `article`, `paginate`, `current_tags`

Use this baseline field checklist in fixtures and design mapping:

```json
{
  "all_tags": [],
  "articles": [],
  "articles_count": 3,
  "comments_enabled?": true,
  "handle": "potion-notions",
  "id": 78580613185,
  "metafields": {},
  "moderated?": true,
  "next_article": {},
  "previous_article": {},
  "tags": [],
  "template_suffix": "",
  "title": "Potion Notions",
  "url": "/blogs/potion-notions"
}
```

Field groups to cover:

- Blog identity: title, url, handle, id, template_suffix.
- Listing metadata: articles_count, all_tags, tags, comments_enabled?, moderated?.
- Article list: articles collection, article card fields (title, excerpt/content preview, image, author, published_at, url, comments_count).
- Navigation: next_article, previous_article, pagination, current_tags filter.
- Empty and filtered states: no posts, tag with no matches.
- Metafields: blog-level metafields for custom data.

### Article page (`article.html` -> `article.json`)

Primary objects: `article`, `blog`, `comments`, `comment`, `form`

Use this baseline field checklist in fixtures and design mapping:

```json
{
  "author": "Polina Waters",
  "comment_post_url": "/blogs/potion-notions/how-to-tell-if-you-have-run-out-of-invisibility-potion/comments",
  "comments": [],
  "comments_count": 1,
  "comments_enabled?": true,
  "content": "<p>We've all had this problem before: we peek into the potions vault to determine which potions we are running low on, and the invisibility potion bottle looks completely empty.</p>\n<p>...</p>\n<p> </p>",
  "created_at": "2022-04-14 16:56:02 -0400",
  "excerpt": "And where to buy <strong>more</strong>!",
  "excerpt_or_content": "And where to buy <strong>more</strong>!",
  "handle": "potion-notions/how-to-tell-if-you-have-run-out-of-invisibility-potion",
  "id": 556510085185,
  "image": {},
  "metafields": {},
  "moderated?": true,
  "published_at": "2022-04-14 16:56:02 -0400",
  "tags": [],
  "template_suffix": "",
  "title": "How to tell if you're out of invisibility potion",
  "updated_at": "2022-06-04 19:27:33 -0400",
  "url": {},
  "user": {}
}
```

Field groups to cover:

- Article identity: title, id, handle, url, template_suffix.
- Content: content (rich HTML), excerpt, excerpt_or_content, image.
- Metadata: author, user, created_at, published_at, updated_at, tags.
- Comments: comments_enabled?, moderated?, comments_count, comments collection, comment_post_url, comment form.
- Navigation context: parent blog, previous/next article behavior if used.
- Social and engagement: share URL targets, comments_count.
- Comment lifecycle: list state, empty state, submission success, validation errors.
- Metafields: article-level metafields for custom data.

### Search page (`search.html` -> `search.json`)

Primary objects: `search`, `search.results`, `paginate`, `routes`

Use this baseline field checklist in fixtures and design mapping:

```json
{
  "default_sort_by": "relevance",
  "filters": {},
  "performed": true,
  "results": [],
  "results_count": 17,
  "sort_by": "relevance",
  "sort_options": [],
  "terms": "potion",
  "types": ["article", "page", "product"]
}
```

Result types: `search.results` can contain `article`, `page`, or `product` objects. Each result type carries its own fields from the corresponding object checklist above.

Field groups to cover:

- Query state: terms, performed, results_count.
- Sorting: default_sort_by, sort_by, sort_options.
- Filtering: filters, applied filters, clear-all behavior.
- Results: results collection, result type detection (article, page, product), type-specific card metadata.
- Pagination: paginate object, current page, total pages, next/previous behavior.
- No-results handling: query echo, recovery links, fallback suggestions, zero-result state.
- Discovery controls: type filtering, sort switching, predictive search when available.

### Standard page (`page.html` -> `page.json`)

Primary object: `page`

Use this baseline field checklist in fixtures and design mapping:

```json
{
  "author": "Polina Waters",
  "content": "<p>Founded in 1842, our apothecary brings time-tested botanical remedies and sustainably harvested ingredients directly to modern homes.</p><h2>Our Philosophy</h2><p>Every formula begins with respect for nature, small-batch brewing, and uncompromising purity standards.</p>",
  "handle": "about-us",
  "id": 89234857217,
  "published_at": "2023-01-15 10:00:00 -0400",
  "template_suffix": "",
  "title": "About Us",
  "url": "/pages/about-us"
}
```

Field groups to cover:

- Identity and content: title, content, handle, url, published_at, template_suffix.
- Rich text behavior: headings, lists, tables, media embeds, long-form content.

### Contact page (`page.contact.html` -> `page.contact.json`)

Primary objects: `page`, `form` (`contact`)

Use this baseline field checklist in fixtures and design mapping:

```json
{
  "page": {
    "id": 89234857218,
    "handle": "contact-us",
    "title": "Contact Us",
    "content": "<p>Have a question about our products or custom orders? Send us a message and we will reply within one business day.</p>",
    "url": "/pages/contact-us"
  },
  "form": {
    "name": "Jane Doe",
    "email": "jane@example.com",
    "phone": "+1 (555) 234-5678",
    "body": "I would like to inquire about bulk ordering.",
    "posted_successfully?": false,
    "errors": {
      "messages": {
        "email": ["Please enter a valid email address."]
      }
    }
  }
}
```

Field groups to cover:

- Static context: page title/content.
- Form fields: name, email, phone, order reference, message, consent.
- Form states: success confirmation, field errors, global errors, preserved input values.

### Password page (`password.html` -> `password.json`)

Primary objects: `shop`, `settings`, `form` (`storefront_password`), `form` (`customer`)

Use this baseline field checklist in fixtures and design mapping:

```json
{
  "shop": {
    "name": "Cedar & Loom",
    "password_message": "Opening soon! We are putting the final touches on our new seasonal collection."
  },
  "password_form": {
    "posted_successfully?": false,
    "errors": {
      "messages": {
        "password": ["Incorrect password. Please try again."]
      }
    }
  },
  "newsletter_form": {
    "email": "",
    "posted_successfully?": false,
    "errors": {}
  }
}
```

Field groups to cover:

- Brand presentation: logo, store name, message, social links.
- Access form behavior: password field, submit, validation errors, retry flow.
- Optional capture: email signup or announcement behaviors when configured.

### Gift card page (`gift_card.html` -> `gift_card.liquid`)

Primary object: `gift_card`

Use this baseline field checklist in fixtures and design mapping:

```json
{
  "balance": "100.00",
  "code": "ABCD 1234 EFGH 5678",
  "currency": "USD",
  "customer": {
    "name": "Alex Mercer",
    "email": "alex@example.com"
  },
  "disabled": false,
  "enabled": true,
  "expired": false,
  "expires_on": null,
  "id": 58392019482,
  "initial_value": "100.00",
  "masked_code": "•••• •••• •••• 5678",
  "qr_identifier": "58392019482",
  "recipient": {
    "name": "Sam Taylor",
    "email": "sam@example.com",
    "message": "Enjoy finding something special for your home!"
  }
}
```

Field groups to cover:

- Value and balance: initial_value, balance, currency formatting.
- Redemption data: code display strategy, masked/unmasked states.
- Expiry and status: enabled/expired/disabled presentation.
- Utility actions: print, Apple/Google wallet actions, QR/barcode presentation.

### 404 page (`404.html` -> `404.json`)

Primary objects: `routes`, optional discovery sources (`search`, featured links)

Use this baseline field checklist in fixtures and design mapping:

```json
{
  "routes": {
    "root_url": "/",
    "all_products_collection_url": "/collections/all",
    "search_url": "/search",
    "cart_url": "/cart",
    "account_url": "/account"
  },
  "recovery_links": [
    { "title": "Bestsellers", "url": "/collections/bestsellers" },
    { "title": "New Arrivals", "url": "/collections/new-arrivals" },
    { "title": "Contact Support", "url": "/pages/contact-us" }
  ]
}
```

Field groups to cover:

- Recovery actions: continue shopping, popular collections, search entry.
- Context clarity: not-found messaging, suggested next steps.
- Optional telemetry hooks: missing path context where available.

### Home shell (`index.html` -> `index.json`)

Primary objects: global `shop`, `routes`, `linklists`, `localization`

Use this baseline field checklist in fixtures and design mapping:

```json
{
  "shop": {
    "name": "Cedar & Loom",
    "description": "Modern handcrafted furniture and timeless home goods.",
    "money_format": "${{amount}}",
    "currency": "USD",
    "domain": "cedar-loom.myshopify.com",
    "email": "care@cedarandloom.com"
  },
  "routes": {
    "root_url": "/",
    "all_products_collection_url": "/collections/all",
    "search_url": "/search",
    "cart_url": "/cart",
    "cart_add_url": "/cart/add",
    "cart_change_url": "/cart/change",
    "cart_clear_url": "/cart/clear",
    "predictive_search_url": "/search/suggest",
    "account_url": "/account",
    "account_login_url": "/account/login",
    "account_logout_url": "/account/logout",
    "account_register_url": "/account/register"
  },
  "linklists": {
    "main-menu": {
      "title": "Main menu",
      "links": [
        { "title": "Home", "url": "/", "active": true, "links": [] },
        {
          "title": "Shop",
          "url": "/collections/all",
          "active": false,
          "links": [
            { "title": "Seating", "url": "/collections/seating", "links": [] },
            { "title": "Tables", "url": "/collections/tables", "links": [] },
            { "title": "Lighting", "url": "/collections/lighting", "links": [] }
          ]
        },
        { "title": "Stories", "url": "/blogs/journal", "active": false, "links": [] },
        { "title": "About", "url": "/pages/about-us", "active": false, "links": [] }
      ]
    },
    "footer": {
      "title": "Footer menu",
      "links": [
        { "title": "Privacy Policy", "url": "/policies/privacy-policy" },
        { "title": "Terms of Service", "url": "/policies/terms-of-service" },
        { "title": "Shipping & Returns", "url": "/pages/shipping-returns" },
        { "title": "Contact Us", "url": "/pages/contact-us" }
      ]
    }
  },
  "localization": {
    "available_countries": [
      { "name": "United States", "iso_code": "US", "currency": { "iso_code": "USD", "symbol": "$" } },
      { "name": "United Kingdom", "iso_code": "GB", "currency": { "iso_code": "GBP", "symbol": "£" } },
      { "name": "United Arab Emirates", "iso_code": "AE", "currency": { "iso_code": "AED", "symbol": "د.إ" } }
    ],
    "available_languages": [
      { "name": "English", "iso_code": "en", "endonym_name": "English" },
      { "name": "Arabic", "iso_code": "ar", "endonym_name": "العربية" }
    ],
    "country": { "name": "United States", "iso_code": "US", "currency": { "iso_code": "USD", "symbol": "$" } },
    "language": { "name": "English", "iso_code": "en", "endonym_name": "English" }
  }
}
```

Field groups to cover:

- Shared shell data contracts used by future sections.
- Navigation, cart count indicator, account links, locale/currency selectors.
- Global announcement and utility surfaces without inventing homepage sections.

## Completion Criteria

### `/theme-forge init` is complete when:

- The requested theme identity and category are recorded in `docs/theme-settings.md`.
- The design direction is documented with key visual characteristics.
- The standalone design workspace (`design/`) exists with all default page prototypes represented (`index.html`, `product.html`, `collection.html`, `collections-list.html`, `cart.html`, `cart-drawer.html`, `blog.html`, `article.html`, `search.html`, `page.html`, `page.contact.html`, `password.html`, `gift_card.html`, `404.html`).
- All styles are consolidated strictly into the single shared CSS file (`design/styles.css`) with **zero hardcoded colors** (all colors strictly mapped to `--[theme-slug]-color-*` tokens and color scheme scopes), and all client logic into `design/script.js` written **strictly using Object-Oriented Programming (OOP)** ES6 classes with clean encapsulation, standardized lifecycles, and a central component registry. Zero inline CSS (`style` attributes, `<style>` tags) or inline JS (`onclick`, inline `<script>` tags) in HTML pages.
- `docs/theme-settings.md` contains a robust, project-derived global settings contract covering the nine canonical global categories, with 1:1 matching CSS custom properties (`--[theme-slug]-*`) defined in `design/styles.css`, including the full color scheme system with at least three schemes and role-to-token mappings.
- Global header and footer are developed strictly in `design/index.html` without duplicating markup in secondary page prototypes.
- Realistic static fixture data is placed in `design/fixtures/` and assets in `design/assets/`.
- `.shopifyignore` is created or updated in the project root to ignore `design/`, `docs/`, `AGENTS.md`, and `CLAUDE.md`.
- `AGENTS.md` is created or updated in the project root with the theme conventions and strict standalone prototype rules.
- `CLAUDE.md` is created as a relative symbolic link pointing to `AGENTS.md`.
- The prototype remains completely independent of Liquid, Shopify APIs, and any external frameworks.
- No homepage sections or blocks were invented prematurely.
- Completion report lists created, updated, skipped, and unresolved items.

### `/theme-forge init-agents` (or `/theme-forge agents`) is complete when:

- The existing project structure, theme identifiers, CSS custom properties, and JavaScript components have been inspected.
- `AGENTS.md` is created or updated in the project root with comprehensive guidelines scoped strictly to the standalone design prototype (tokens, OOP JS, single stylesheet/script, single header/footer in `index.html`, RTL logical properties, boundary isolation).
- `CLAUDE.md` is created or verified as a relative symbolic link pointing to `AGENTS.md`.
- `.shopifyignore` is created or updated to ignore prototype and agent guidance files.
- No existing HTML, CSS, JavaScript, fixture, or asset files were modified, overwritten, or reset.
- Completion report lists created, updated, skipped, and unresolved items.

### `/theme-forge resolve` is complete when:

- An audit of the existing workspace (`design/styles.css`, `design/script.js`, HTML templates, and `docs/theme-settings.md`) has been performed against the latest Theme Forge rules and standards.
- All hardcoded color literals in `design/styles.css` have been extracted and replaced with semantic `--[theme-slug]-color-*` tokens and color scheme scopes (`:root`, `[data-color-scheme="primary"]`, `[data-color-scheme="secondary"]`, `[data-color-scheme="contrast"]`).
- All physical directional CSS properties in `design/styles.css` have been converted to logical CSS properties (`*-inline-start`, `*-inline-end`, etc.) ensuring full RTL support.
- All JavaScript in `design/script.js` is refactored into modular ES6 classes with standard lifecycle methods (`constructor`, `init`, `bindEvents`, `destroy`) and coordinated via a central `ThemeApp` / `ComponentRegistry`. All floating global functions and procedural scripts are eliminated.
- Global `<header>` and `<footer>` are consolidated strictly into `design/index.html`, and any duplicate headers/footers in secondary HTML pages have been stripped to leave clean `<main>` templates.
- All inline styles (`style="..."`) and inline event handlers (`onclick`, etc.) have been removed from HTML files and relocated to `design/styles.css` and `design/script.js`.
- All 14 default Shopify page prototypes exist in `design/`, with minimal accessible scaffolds (`data-prototype-status="scaffold"`) added for any missing pages.
- `docs/theme-settings.md` is synchronized with the 9 canonical categories and matches active CSS custom properties in `design/styles.css`.
- If design skills (e.g., `frontend-design`) are available, visual refinements, hierarchy, typography scaling, and micro-interactions are integrated without violating token or boundary rules.
- `AGENTS.md` is updated with full standalone prototype guidelines, `CLAUDE.md` is verified as a relative symlink to `AGENTS.md`, and `.shopifyignore` is updated.
- A completion report is presented detailing created, updated, skipped, and unresolved items, including specific rule violations reconciled.


