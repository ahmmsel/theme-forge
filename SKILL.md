---
name: theme-forge
description: "Use this skill whenever a user is starting, planning, auditing, or wiring a Shopify theme whose visual design is being developed outside Shopify, including requests to create a theme prototype, define merchant-facing settings, map HTML/CSS/JavaScript to Liquid, or produce a Shopify wiring report. Run /theme-forge init to create a theme-neutral design workspace, or /theme-forge wire to inspect an existing prototype and generate docs/shopify-wiring.md. Use it even when the user does not name Theme Forge but asks for a design-first Shopify theme workflow."
compatibility: "Plain Markdown workflow; requires filesystem access to the selected project root. Depends on the frontend-design skill from https://github.com/anthropics/skills for visual direction and UI refinement. Package manager access (npx) is required only for automatic installation during init when the companion skill is missing. No runtime, framework, or Shopify credentials required."
---

# Theme Forge

Theme Forge is a design-first workflow for building Shopify themes. It keeps visual design independent from Shopify implementation until the design contract is stable, then investigates how the approved design should be wired to Shopify.

The skill is reusable across themes. A theme name, theme slug, category, visual language, and design system may change per project. The workflow and Shopify checks remain consistent.

## Portability and Invocation

This is a plain Markdown skill. It does not require a package manager, runtime, build tool, MCP server, or design application.

When the host supports slash commands, use:

```text
/theme-forge init
/theme-forge wire
```

When the host does not support slash commands, interpret `init` and `wire` as the first user-provided argument to the skill. If no command is provided, explain the two commands and ask which one to run. Do not silently choose `init` or `wire`.

The skill must work from a project directory, not only from the directory where the skill file is installed. Resolve all project paths relative to the user's selected project root.

Companion skill dependency:

- Treat `frontend-design` as a required companion skill for visual design and UX shaping tasks.
- Use `frontend-design` whenever the user asks for UI direction, layout systems, typography, color systems, motion, responsive behavior, accessibility polish, or interface critique.
- Run `frontend-design` before drafting major prototype aesthetics, then apply its guidance while preserving Theme Forge's design-first and Shopify-wiring boundaries.
- During `/theme-forge init`, if `frontend-design` is missing, install it automatically with `npx skills add https://github.com/anthropics/skills --skill frontend-design`.
- If auto-install fails, continue init and report the install failure under `unresolved` with the exact command output and a manual retry command.

### Command selection

1. Read the user's requested command from the first argument or from the request context.
2. If it is `init`, follow only the initialization workflow.
3. If it is `wire`, follow only the wiring investigation workflow.
4. If no command is present, explain both commands and ask which one to run; do not inspect or modify project files yet.
5. If the command is unknown, show the two valid commands and ask the user to choose.

At the start of a command, state the selected project root and the files that will be inspected or created. At completion, report four separate lists: created, updated, skipped, and unresolved. Keep the report factual and distinguish observations from recommendations.

## Safety and Change Policy

- Inspect the project before creating or changing files.
- Never delete, reset, or overwrite user-authored files without explicit confirmation.
- Create missing directories and files only when they are part of the requested command output.
- If a target file already exists, preserve its content and make the smallest compatible update. Report what was preserved and what changed.
- Do not modify Shopify production files during `wire`.
- Do not add dependencies, build tools, frameworks, design systems, or remote assets unless explicitly requested.
- Do not invent Shopify capabilities. Mark uncertain mappings as `unresolved` and explain what must be verified.
- At the end of either command, report created files, updated files, skipped files, and unresolved decisions.

## Commands

### `/theme-forge init`

Initialize a new theme design workspace at the selected project root.

Before creating files, ask for:

1. **Theme name** - the merchant-facing name.
2. **Theme category** - the business or storefront category, such as fashion, beauty, food, furniture, electronics, services, or general commerce.
3. **Theme slug** - derive a lowercase machine-readable slug from the theme name and confirm it if ambiguous.
4. **Supported languages and text directions** - default to English and RTL support only when requested or relevant.
5. **Existing project location** - use the current workspace unless the user specifies another directory.

Question formatting rule for hosts that use structured question tools:

- Never ask the project location question with only one selectable option.
- If using options, provide at least two: `Current workspace root` and `Custom path`.
- If only one practical path is available, ask as free text instead of a single-option chooser.
- If the user picks `Current workspace root`, run init there without another location prompt.

Do not ask the user to choose a design system unless the project needs one. The skill must remain compatible with Figma, Penpot, Storybook, hand-written HTML, or any other design workflow.

#### Initialization behavior

Create the smallest useful design-first structure. Preserve existing user files and never overwrite an existing file without inspecting it first.

Initialization root rule: run `/theme-forge init` against the selected project root and create or update child paths from there. Do not treat `design/` as the project root.

Use this order:

1. Inspect the project root and identify existing design, docs, assets, Shopify, and configuration directories.
2. Check whether `frontend-design` is installed and available.
3. If missing, run `npx skills add https://github.com/anthropics/skills --skill frontend-design` before continuing.
4. Confirm the derived theme slug if it is ambiguous or conflicts with an existing project identifier.
5. Create only missing directories and files.
6. Write the settings contract and wiring placeholder using the requested identity.
7. Summarize created, updated, skipped, and unresolved items.

```text
design/
  index.html
  404.html
  article.html
  blog.html
  cart.html
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
  shopify-wiring.md
```

If the project already has an equivalent structure, reuse it and add only missing files. Do not create a framework, build step, component library, or design system.

Add or update `docs/theme-settings.md` with:

- Theme name, slug, category, supported locales, and direction.
- The rule that global settings are designed before homepage sections and blocks.
- The selected CSS token naming convention: `--[theme-slug]-*` or an existing project convention.
- The approved page list.
- A setting inventory table with the columns defined below.
- A clear marker for unresolved decisions.

Add `docs/shopify-wiring.md` as a placeholder only when it does not exist. It must contain the theme identity, date, source directories, inspection status (`not started`), and a note that `/theme-forge wire` must be run after the prototype is sufficiently complete. If the file already contains a wiring report, do not replace it during `init`.

Do not create empty HTML files that imply a finished design. If a page prototype does not exist, create a minimal accessible scaffold with a clear `data-prototype-status="scaffold"` marker, or record the missing page in `docs/theme-settings.md` when the user asks for documentation only.

Do not build homepage sections or blocks during initialization. `design/index.html` should establish the shared shell only. Homepage composition is a separate design task after the global settings and default page layouts are credible.

#### Init design rules

The prototype must use:

- Plain HTML files, one shared CSS file, and one shared JavaScript file.
- Static fixtures that resemble future Shopify data but contain no Liquid or Shopify API calls.
- Stable semantic classes and `data-*` hooks that can survive Liquid conversion.
- CSS custom properties for global visual roles.
- Native HTML behavior first: forms, links, buttons, `details`, and `dialog` where appropriate.
- Progressive enhancement: important content and forms remain understandable without JavaScript.
- Responsive, keyboard, focus-visible, reduced-motion, empty, error, loading, unavailable, and missing-media states.
- Labels as well as clear placeholders in every prototype form.
- Logical CSS properties and RTL-safe layout when RTL is in scope.

Do not hard-code a particular aesthetic. The category can influence sample content and fixture shape, but it must not force colors, fonts, layout, or a design system.

## Global Settings Contract

`docs/theme-settings.md` is the primary output of `/theme-forge init`. It must describe settings by their design effect, not by HTML structure.

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

### Recommended global groups

These are recommendations, not mandatory categories. Add, merge, or rename groups when the theme needs it.

- **Brand:** logo, alternate logo, logo width, favicon, social links.
- **Color:** reusable color schemes and semantic roles such as background, text, muted text, borders, links, icons, buttons, badges, success, warning, and error.
- **Typography:** display, heading, body, and accent roles, each with font, scale, line height, letter spacing, and case where Shopify supports the control.
- **Layout:** page width, content gutters, spacing scale, media ratios, container variants, and corner-radius preset.
- **Buttons and forms:** button style, arrow behavior, field style, focus-ring style, and density where the merchant needs control.
- **Cards and media:** product, collection, and article card defaults; image ratio, fit, metadata, badges, swatches, and hover behavior.
- **Cart and discovery:** cart mode, drawer behavior, cart fields, shipping progress, search modal, predictive search, quick actions, pagination, and recently viewed behavior.
- **Navigation and behavior:** sticky and overlay header behavior, menus, selectors, account access, breadcrumbs, back-to-top, external links, reduced motion, and prose treatment.

Do not expose implementation details as settings merely because they are technically configurable. A setting belongs in the global contract only when a merchant can understand its outcome and reasonably needs to change it across the storefront.

## Default Page Coverage

The initialization must account for these Shopify page experiences:

| Prototype file          | Shopify target          | Required design concern                                                     |
| ----------------------- | ----------------------- | --------------------------------------------------------------------------- |
| `404.html`              | `404.json`              | Recovery, search/discovery, and not-found state                             |
| `article.html`          | `article.json`          | Article header, metadata, media, prose, sharing, related content            |
| `blog.html`             | `blog.json`             | Blog listing, pagination/load-more, empty state                             |
| `cart.html`             | `cart.json`             | Lines, quantities, removal, notes, discounts, totals, checkout, empty state |
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

## Source Precedence for `wire`

When sources disagree, use this precedence and record the conflict:

1. Existing Shopify schema and Liquid implementation.
2. Existing `docs/theme-settings.md` decisions.
3. Existing HTML, CSS, and JavaScript behavior.
4. Fixtures and comments.
5. Generic Shopify assumptions.

The report must distinguish observed facts from recommendations. Use `observed`, `inferred`, or `proposed` in the confidence field rather than presenting an inference as an existing implementation.

## `/theme-forge wire`

Generate or update `docs/shopify-wiring.md` from the current codebase. This command is an investigation, not a blind conversion and not a request to implement Liquid files.

Before writing, inspect the project and identify:

- HTML pages and their shared shell.
- CSS custom properties, token groups, media queries, and component selectors.
- JavaScript behavior, state attributes, event hooks, forms, dialogs, and network calls.
- Product, collection, article, cart, account, search, contact, password, and gift-card fixtures.
- Existing Liquid files, Shopify schemas, templates, snippets, locales, metafield references, app integrations, or theme configuration if present.
- Sections and blocks if they already exist, without assuming they are the source of truth for the design.
- Accessibility and localization behavior visible in the codebase.

### Wiring document requirements

Write `docs/shopify-wiring.md` with these sections:

1. **Project identity and inspection scope**
   - Theme name, slug, category, inspected paths, and date.
   - What was found and what was absent.

2. **Global settings map**
   - Every documented setting and every discovered CSS token.
   - Proposed `config/settings_schema.json` ID and control type.
   - Whether it belongs in global settings or a section/template schema.
   - Default, fallback, CSS output, and affected pages.
   - Confidence and unresolved questions.

3. **Token-to-Shopify map**
   - CSS custom property.
   - Source setting or Liquid value.
   - Where it should be emitted.
   - Whether it needs a color scheme, font picker, range, select, checkbox, image, URL, or text setting.
   - Tokens that must remain code-level.

4. **Page-to-template map**
   - Each default prototype page.
   - Shopify template target.
   - Required Liquid objects, filters, forms, routes, and fallback content.
   - Template limitations or special handling.

5. **Section and block map**
   - Existing sections and blocks only.
   - Their settings, data sources, shared snippets, and template availability.
   - Items intentionally deferred for later homepage design.

6. **Behavior and JavaScript map**
   - Prototype behavior.
   - Shopify implementation target such as native HTML, Liquid, theme JavaScript, AJAX API, or app.
   - Progressive-enhancement fallback.
   - Loading, error, and failure behavior.

7. **Data and integration dependencies**
   - Native Shopify data.
   - Metafields.
   - Shopify apps.
   - Customer/account requirements.
   - External services.
   - Missing data and fallback rules.

8. **What not to wire**
   - Internal class names, DOM mechanics, breakpoints, animation internals, test hooks, fixture data, and other code-level details.
   - Explain any exception where a merchant-facing setting is justified.

9. **Risks and decisions**
   - Shopify schema limitations.
   - Unsupported or app-dependent behavior.
   - Localization and RTL risks.
   - Accessibility and performance risks.
   - Conflicts between the prototype and Shopify's data model.

10. **Recommended implementation order**
    - Settings and token foundation first.
    - Shared shell and reusable snippets next.
    - Commerce-critical templates next.
    - Homepage sections and blocks only after their separate design work.

11. **Inspection summary**

- Files inspected.
- Files intentionally ignored.
- Files not found but expected.
- Commands or external documentation still needed for verification.

### Classification vocabulary

Use one or more of these labels for every mapped item:

- `native`: supported by Shopify's platform behavior.
- `liquid`: rendered from Shopify objects, filters, or tags.
- `theme-schema`: configured through theme settings, section settings, or block settings.
- `ajax`: requires a Shopify endpoint and theme JavaScript.
- `app-dependent`: requires an installed app or app embed.
- `metafield-dependent`: requires merchant-created custom data.
- `implementation-only`: should remain in CSS, JavaScript, or markup internals.
- `not-applicable`: prototype-only or irrelevant to Shopify output.
- `unresolved`: needs a decision or validation before implementation.

### Rules for accurate wiring

- Do not claim a feature is native without checking the current Shopify theme and Liquid capabilities.
- Do not convert every CSS token into a merchant setting. Preserve code-level tokens when exposing them would add noise or weaken consistency.
- Do not put global concerns into every section schema. Use global settings for genuinely site-wide behavior and section settings for local composition.
- Do not assume a prototype fixture has a direct Shopify equivalent. Record the actual object, route, form, endpoint, app, or metafield required.
- Do not silently remove prototype behavior. Mark it unsupported, deferred, app-dependent, or unresolved.
- Do not modify the design prototype during `/theme-forge wire` unless the user explicitly asks for implementation changes.
- Do not claim that a wiring report is a Shopify-valid implementation. It is an implementation map that must be validated during theme development.
- Preserve an existing report's decisions and history when updating it; revise stale findings in place and add an `Updated` date.

## Completion Criteria

`/theme-forge init` is complete when:

- The requested theme identity and category are recorded.
- The design workspace exists or an existing equivalent is documented.
- All default page prototypes are represented.
- `docs/theme-settings.md` contains a stable initial global settings contract.
- The prototype remains independent of Liquid, Shopify APIs, and any specific design system tool.
- No homepage sections or blocks were invented prematurely.

`/theme-forge wire` is complete when:

- `docs/shopify-wiring.md` reflects the inspected codebase rather than generic assumptions.
- Every global setting and CSS token has a wiring decision or an explicit unresolved status.
- Every default page has a Shopify target and data-path summary.
- Existing sections and blocks are mapped without becoming the design authority.
- Shopify, app, metafield, AJAX, localization, accessibility, and performance dependencies are visible.
- Implementation-only details are clearly excluded from merchant settings.
