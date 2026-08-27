# Theme Forge

Theme Forge is a design-first workflow skill for Shopify themes designed outside Shopify.

It separates visual prototyping from Shopify implementation by providing three core commands:

- `/theme-forge init` scaffolds a theme-neutral prototype workspace, settings contract, and `.shopifyignore`.
- `/theme-forge wire [target]` audits an existing prototype and produces a wiring blueprint in `docs/shopify-wiring.md` for global settings, sections, blocks, or specific pages (`product`, `collection`, `cart`, etc.).
- `/theme-forge forge [target]` (or `/theme-forge implement [target]`) implements the wiring map into production Shopify theme files (`config/`, `sections/`, `templates/`, `layout/`).

The workflow uses plain Markdown instructions and keeps prototypes independent of Liquid, Shopify APIs, frameworks, and design tools.

## Installation

Install or copy this directory into the skills directory supported by your AI coding environment. The required skill file is `SKILL.md`; the `evals/` directory contains the initial evaluation prompts.

## Design Principles

- Preserve existing project files and avoid destructive changes.
- Keep design prototypes independent from Shopify implementation.
- Distinguish observed facts, inferences, recommendations, and unresolved decisions.
- Prefer native HTML, progressive enhancement, accessibility, localization, and RTL-safe patterns.
- Do not invent Shopify capabilities or modify production Shopify files during wiring analysis.

## License

Released under the [MIT License](LICENSE).
