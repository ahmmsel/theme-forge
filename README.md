# Theme Forge

Theme Forge is a design-first workflow skill for Shopify themes designed outside Shopify.

It separates visual prototyping from Shopify implementation by providing two commands:

- `/theme-forge init` creates a theme-neutral prototype workspace and an initial merchant-facing settings contract.
- `/theme-forge wire` audits an existing prototype and documents how it should map to Shopify templates, Liquid, schemas, AJAX, apps, and metafields.

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
