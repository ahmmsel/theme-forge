# Theme Forge

Theme Forge is a design-first workflow skill for eCommerce and Shopify storefronts designed outside Shopify theme directories.

It focuses purely on developing standalone frontend prototypes, design tokens, and component architecture:

- `/theme-forge init` scaffolds a standalone design prototype workspace (`design/`), 9-category global settings contract (`docs/theme-settings.md`), agent instructions (`AGENTS.md`), symlink `CLAUDE.md`, and `.shopifyignore`.
- `/theme-forge init-agents [scope]` (alias `/theme-forge agents`) inspects an existing project and initializes or refreshes `AGENTS.md`, `CLAUDE.md`, and `.shopifyignore` scoped strictly to the design prototype rules without modifying existing design files.

The workflow uses plain Markdown instructions and keeps prototypes entirely independent of Liquid, Shopify theme directories, frameworks, and design tools.

## Installation

Install or copy this directory into the skills directory supported by your AI coding environment. The required skill file is `SKILL.md`; the `evals/` directory contains the evaluation prompts.

## Design Principles

- **Standalone Prototype Workspace**: All visual design and client interactions are isolated in `design/`, `docs/`, `fixtures/`, and `assets/`.
- **Zero Hardcoded Colors**: `design/styles.css` strictly resolves all colors via CSS custom properties (`--[theme-slug]-color-*`) and semantic color scheme scopes.
- **Object-Oriented JavaScript (OOP)**: All client logic in `design/script.js` is structured into ES6 classes with clean encapsulation, standardized lifecycles (`init`, `destroy`), and a central component registry.
- **Single Stylesheet & Single Script Rule**: Single shared `styles.css` and single `script.js` with zero inline CSS (`style` attributes, `<style>` blocks) or inline JS (`onclick`, inline `<script>` blocks).
- **Single Header & Footer Source**: Global `<header>` and `<footer>` are developed strictly in `design/index.html` (the shared shell); secondary pages contain only their page `<main>` content.
- **Logical CSS & RTL-Ready**: Bidirectional-safe layout using CSS logical properties (`inline-start`/`inline-end`).
- **Semantic HTML & Accessibility**: Native HTML elements, progressive enhancement, WCAG AA compliance, focus-visible states, and responsive styling.

## License

Released under the [MIT License](LICENSE).
