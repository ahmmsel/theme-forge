---
name: theme-forge
description: "Use this skill whenever a user is starting, planning, auditing, or wiring a Shopify theme whose visual design is being developed outside Shopify, including requests to create a theme prototype, define merchant-facing settings, map HTML/CSS/JavaScript to Liquid, or produce a Shopify wiring report. Run /theme-forge init to create a theme-neutral design workspace, or /theme-forge wire to inspect an existing prototype and generate docs/shopify-wiring.md. Use it even when the user does not name Theme Forge but asks for a design-first Shopify theme workflow."
compatibility: "Plain Markdown workflow; requires filesystem access to the selected project root. Depends on the frontend-design skill from https://github.com/anthropics/skills for visual direction and UI refinement. Package manager access (npx) is required only for automatic installation during init when the companion skill is missing. No runtime, framework, or Shopify credentials required."
---

# Theme Forge

Theme Forge is a design-first workflow for building Shopify themes. It keeps visual design independent from Shopify implementation until the design contract is stable, then investigates how the approved design should be wired to Shopify.

The skill is reusable across themes. A theme name, theme slug, category, visual language, and design direction may change per project. The workflow and Shopify checks remain consistent.

## Portability and Invocation

This is a plain Markdown skill. It does not require a runtime, build tool, MCP server, or design application. A package manager is only needed when `/theme-forge init` performs companion-skill auto-install.

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
- If auto-install fails (missing `npx`, network/auth issues, or permission errors), continue init and report the install failure under `unresolved` with the exact command output and a manual retry command.
- If the user explicitly says not to install dependencies in the current run, skip auto-install, continue init, and record `frontend-design` as `unresolved`.

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
- Exception: installing `frontend-design` during `/theme-forge init` is allowed as part of this skill's declared dependency workflow.
- Do not invent Shopify capabilities. Mark uncertain mappings as `unresolved` and explain what must be verified.
- At the end of either command, report created files, updated files, skipped files, and unresolved decisions.

## Commands

### `/theme-forge init`

Initialize a new theme design workspace at the selected project root.

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

Create the smallest useful design-first structure. Preserve existing user files and never overwrite an existing file without inspecting it first.

Initialization root rule: run `/theme-forge init` against the selected project root and create or update child paths from there. Do not treat `design/` as the project root.

Use this order:

1. Inspect the project root and identify existing design, docs, assets, Shopify, and configuration directories.
2. Check whether `frontend-design` is installed and available.
3. If missing, run `npx skills add https://github.com/anthropics/skills --skill frontend-design` before continuing, unless the user explicitly requested no install for this run.
4. Re-check companion-skill availability and record success or failure.
5. Confirm the derived theme slug if it is ambiguous or conflicts with an existing project identifier.
6. Create only missing directories and files.
7. Write the settings contract and wiring placeholder using the requested identity.
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
  shopify-wiring.md
```

If the project already has an equivalent structure, reuse it and add only missing files. Do not create a framework, build step, component library, or design system.

Add or update `docs/theme-settings.md` with:

- Theme name, slug, category, supported locales, and direction.
- Design direction: the chosen visual style and key characteristics (e.g., soft shadows for Neumorphism, glass blur for Glassmorphism).
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
- Plain CSS only — no CSS frameworks, preprocessors, or utility libraries (no Tailwind, Bootstrap, etc.).
- Vanilla JavaScript only — no JS frameworks, libraries, or build tools (no React, Alpine, GSAP, etc.).
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
- **Layout:** page width, content gutters, spacing scale, media ratios, container variants, corner-radius preset, and bidirectional-safe spacing/margins.
- **Buttons and forms:** button style, arrow behavior, field style, focus-ring style, and density where the merchant needs control.
- **Cards and media:** product, collection, and article card defaults; image ratio, fit, metadata, badges, swatches, and hover behavior.
- **Cart and discovery:** cart page, cart drawer, cart mode, drawer behavior, cart fields, shipping progress, search modal, predictive search, quick actions, pagination, and recently viewed behavior.
- **Navigation and behavior:** sticky and overlay header behavior, menus, selectors, account access, breadcrumbs, back-to-top, external links, reduced motion, bidirectional layout, and prose treatment.

Do not expose implementation details as settings merely because they are technically configurable. A setting belongs in the global contract only when a merchant can understand its outcome and reasonably needs to change it across the storefront.

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

Field groups to cover:

- Identity and content: title, content, handle, url, published_at, template_suffix.
- Rich text behavior: headings, lists, tables, media embeds, long-form content.

### Contact page (`page.contact.html` -> `page.contact.json`)

Primary objects: `page`, `form` (`contact`)

Field groups to cover:

- Static context: page title/content.
- Form fields: name, email, phone, order reference, message, consent.
- Form states: success confirmation, field errors, global errors, preserved input values.

### Password page (`password.html` -> `password.json`)

Primary objects: `shop`, `settings`, `form` (`storefront_password`)

Field groups to cover:

- Brand presentation: logo, store name, message, social links.
- Access form behavior: password field, submit, validation errors, retry flow.
- Optional capture: email signup or announcement behaviors when configured.

### Gift card page (`gift_card.html` -> `gift_card.liquid`)

Primary object: `gift_card`

Field groups to cover:

- Value and balance: initial_value, balance, currency formatting.
- Redemption data: code display strategy, masked/unmasked states.
- Expiry and status: enabled/expired/disabled presentation.
- Utility actions: print, Apple/Google wallet actions, QR/barcode presentation.

### 404 page (`404.html` -> `404.json`)

Primary objects: `routes`, optional discovery sources (`search`, featured links)

Field groups to cover:

- Recovery actions: continue shopping, popular collections, search entry.
- Context clarity: not-found messaging, suggested next steps.
- Optional telemetry hooks: missing path context where available.

### Home shell (`index.html` -> `index.json`)

Primary objects: global `shop`, `routes`, localization context, shared header/footer needs

Field groups to cover:

- Shared shell data contracts used by future sections.
- Navigation, cart count indicator, account links, locale/currency selectors.
- Global announcement and utility surfaces without inventing homepage sections.

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
   - Localization and bidirectional layout risks (hard-coded left/right, directional icons, asymmetric spacing).
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
- The design direction is documented with key visual characteristics.
- The design workspace exists or an existing equivalent is documented.
- All default page prototypes are represented.
- `docs/theme-settings.md` contains a stable initial global settings contract.
- The prototype remains independent of Liquid, Shopify APIs, and any external frameworks.
- No homepage sections or blocks were invented prematurely.

`/theme-forge wire` is complete when:

- `docs/shopify-wiring.md` reflects the inspected codebase rather than generic assumptions.
- Every global setting and CSS token has a wiring decision or an explicit unresolved status.
- Every default page has a Shopify target and data-path summary.
- Existing sections and blocks are mapped without becoming the design authority.
- Shopify, app, metafield, AJAX, localization, accessibility, and performance dependencies are visible.
- Implementation-only details are clearly excluded from merchant settings.
