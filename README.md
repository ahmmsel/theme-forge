# Theme Forge

Theme Forge is a design-first workflow skill for eCommerce and Shopify storefronts, embedded directly within Shopify theme directories.

It develops visual prototypes, design tokens, component architecture, and settings contracts inside dot-prefixed workspace folders alongside production theme code:

- `/theme-forge init` scaffolds an embedded design prototype workspace (`.design/`), 9-category global settings contract (`.docs/theme-settings.md`), theme agent instructions (`AGENTS.md`), symlink `CLAUDE.md`, and `.shopifyignore`.
- `/theme-forge init-agents [scope]` (alias `/theme-forge agents`) inspects an existing Shopify theme project and initializes or refreshes `AGENTS.md`, `CLAUDE.md`, and `.shopifyignore`—informing AI agents and developers of the complete design contract in `.design/` and `.docs/` and how to implement it across Liquid theme files without modifying existing design or theme files.
- `/theme-forge resolve [scope]` audits, reconciles, and resolves an existing theme prototype or migrates legacy standalone projects (`design/`, `docs/`) into the theme-embedded dot-prefixed structure (`.design/`, `.docs/`), extracting hardcoded colors into CSS tokens, refactoring scripts to OOP classes, stripping duplicate shells in secondary pages, normalizing templates, and aligning settings contracts.

The workflow uses plain Markdown instructions and dot-prefixed folders (`.design/` and `.docs/`) so Shopify CLI commands (`shopify theme dev`, `shopify theme push`, `shopify theme check`) automatically ignore prototype and documentation files during theme deployment.

## Prerequisites

Theme Forge operates strictly inside an existing **Shopify theme** directory. It cannot be initialized in an empty directory or a non-Shopify workspace.

If starting a new theme, scaffold the theme directory first using the Shopify CLI:
```bash
shopify theme init <theme-name>
```
Then run `/theme-forge init` inside the scaffolded theme directory.

## Installation

Install or copy this directory into the skills directory supported by your AI coding environment. The required skill file is `SKILL.md`; the `evals/` directory contains the evaluation prompts.

## Design Principles

- **Embedded Dot-Directory Workspace**: All visual prototypes and client interactions are organized in `.design/` and `.docs/` within the Shopify theme root, completely ignored by Shopify CLI.
- **Zero Hardcoded Colors**: `.design/styles.css` (and theme CSS) strictly resolves all colors via CSS custom properties (`--[theme-slug]-color-*`) and semantic color scheme scopes (`primary`, `secondary`, `contrast`).
- **Object-Oriented JavaScript (OOP)**: All client logic in `.design/script.js` is structured into ES6 classes with clean encapsulation, standardized lifecycles (`init`, `destroy`), and a central component registry (`ThemeApp`).
- **Single Stylesheet & Single Script Rule**: Single shared `.design/styles.css` and single `.design/script.js` with zero inline CSS (`style` attributes, `<style>` blocks) or inline JS (`onclick`, inline `<script>` blocks).
- **Single Header & Footer Source**: Global `<header>` and `<footer>` are developed strictly in `.design/index.html` (the shared shell); secondary pages contain only their page `<main>` content.
- **Logical CSS & RTL-Ready**: Bidirectional-safe layout using CSS logical properties (`inline-start`/`inline-end`).
- **Semantic HTML & Accessibility**: Native HTML elements, progressive enhancement, WCAG AA compliance, focus-visible states, and responsive styling.
- **Living Theme Contract**: `AGENTS.md` bridges `.design/` and `.docs/theme-settings.md` with Shopify theme development (`layout/`, `sections/`, `snippets/`, `templates/`, `config/settings_schema.json`).

## License

Released under the [MIT License](LICENSE).
