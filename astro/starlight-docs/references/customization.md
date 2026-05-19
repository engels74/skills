# Customization (Component Overrides and Route Data)

When CSS, configuration, and frontmatter aren't enough, Starlight lets you swap out its built-in components for your own and modify the data they receive. This is the "advanced customization" layer of the escalation ladder; prefer simpler approaches when they suffice.

## Customization escalation ladder (recap)

For any tweak, try in this order before reaching for the surfaces in this file:

1. **`starlight()` config option** — sidebar shape, social links, custom CSS files, etc.
2. **Per-page frontmatter** — page-specific TOC, sidebar order, banner, hero, draft status.
3. **Custom CSS via `customCss`** — appearance changes that don't require new markup.
4. **Component overrides** *(this file)* — when you need different HTML/behavior in a specific UI slot.
5. **Route data middleware** *(this file)* — when you need to change *what data* a slot receives, not the slot itself.
6. **Plugins** *(see `references/plugins.md`)* — when you need a reusable, shareable extension or to bundle several of the above.

---

## Overriding components

Starlight has named UI slots — `SocialIcons`, `Footer`, `PageTitle`, `Search`, `Hero`, etc. — each backed by a default Astro component. Replace any of them by pointing `components: { … }` in `astro.config.mjs` at your own Astro file.

### The workflow

1. Choose the component to override. The full list is below. If you're not sure which slot owns a piece of UI, the interactive [Starlight Overrides Map](https://starlight-overrides-map.netlify.app/) is the fastest way to identify it visually.

2. Write your replacement Astro component. For example, an "email me" link to replace social icons:

   ```astro
   ---
   // src/components/EmailLink.astro
   const email = 'houston@example.com';
   ---
   <a href={`mailto:${email}`}>E-mail Me</a>
   ```

3. Wire it up:

   ```js
   // astro.config.mjs
   starlight({
     title: 'My Docs with Overrides',
     components: {
       SocialIcons: './src/components/EmailLink.astro',
     },
   })
   ```

### Reusing the built-in component inside your override

Replacement components don't have to throw away the original. Import the default from `@astrojs/starlight/components/<Name>.astro` and compose:

```astro
---
import Default from '@astrojs/starlight/components/SocialIcons.astro';
---

<a href="mailto:houston@example.com">E-mail Me</a>
<Default><slot /></Default>
```

A few details to get right when reusing defaults:

- Add a `<slot />` inside the default component if the original accepts children. Without it, content passed to your override silently disappears.
- For components with **named slots** (notably `PageFrame` with `header` + `sidebar`, and `TwoColumnContent` with `right-sidebar`), you must [transfer the slots](https://docs.astro.build/en/basics/astro-components/#transferring-slots) so they pass through to the default implementation:

  ```astro
  ---
  // src/components/CustomContent.astro
  import Default from '@astrojs/starlight/components/TwoColumnContent.astro';
  ---
  <Default>
    <slot />
    <slot name="right-sidebar" slot="right-sidebar" />
  </Default>
  ```

### Using page data inside an override

Overrides have access to `Astro.locals.starlightRoute`, which contains everything Starlight knows about the current page. Use it to customize behavior per page.

```astro
---
// src/components/Title.astro
const { title } = Astro.locals.starlightRoute.entry.data;
---

<h1 id="_top">{title}</h1>
<style>
  h1 { font-family: 'Comic Sans'; }
</style>
```

Important: `PageTitle` overrides should keep `id="_top"` on the `<h1>` element — many other parts of Starlight (skip links, anchor navigation) depend on it.

### Conditional overrides (default behavior on most pages, custom on others)

Component overrides apply to every page. To behave like an override only on some pages, render conditionally based on `starlightRoute`:

```astro
---
// src/components/ConditionalFooter.astro
import Default from '@astrojs/starlight/components/Footer.astro';
const isHomepage = Astro.locals.starlightRoute.id === '';
---

{
  isHomepage ? (
    <footer>Built with Starlight 🌟</footer>
  ) : (
    <Default>
      <slot />
    </Default>
  )
}
```

### The full list of overridable components

Each accepts a path string in `components: { Name: '...' }`. The "Default component" links go to the source on GitHub — useful for matching the prop interface and named slots when writing replacements.

Override conservatively. The Head, layout, and accessibility components are particularly intertwined; touch them only when you've exhausted simpler options.

#### Head (rendered inside `<head>`)

- **`Head`** — main `<head>` content. Prefer the `head` config option, `head` frontmatter, or route middleware over overriding this. Default: `Head.astro` in `withastro/starlight`.
- **`ThemeProvider`** — sets up dark/light theme support; includes an inline script and `<template>` consumed by `ThemeSelect`. Default: `ThemeProvider.astro`.

#### Accessibility

- **`SkipLink`** — first element inside `<body>`; jumps to main content for keyboard users. Hidden until focused by default. Default: `SkipLink.astro`.

#### Layout (high complexity — override sparingly)

- **`PageFrame`** — wraps most of the page content. Named slots: `header`, `sidebar`. Also renders `MobileMenuToggle`. Default: `PageFrame.astro`.
- **`MobileMenuToggle`** — handles toggling the sidebar on mobile viewports. Default: `MobileMenuToggle.astro`.
- **`TwoColumnContent`** — wraps the main content column and the right sidebar (TOC). Named slot: `right-sidebar`. Handles the single-column-to-two-column responsive switch. Default: `TwoColumnContent.astro`.

#### Header

- **`Header`** — the top navigation bar. By default renders `SiteTitle`, `Search`, `SocialIcons`, `ThemeSelect`, `LanguageSelect`. Default: `Header.astro`.
- **`SiteTitle`** — site title and/or logo at the start of the header. Default: `SiteTitle.astro`.
- **`Search`** — the search UI. Even when `pagefind: false`, your override always renders — handy for wiring an alternative search provider. Default: `Search.astro`.
- **`SocialIcons`** — social icon links from the `social` config. Default: `SocialIcons.astro`.
- **`ThemeSelect`** — color-scheme selector. Default: `ThemeSelect.astro`.
- **`LanguageSelect`** — language switcher (multilingual sites only). Default: `LanguageSelect.astro`.

#### Global sidebar

- **`Sidebar`** — main site navigation. Renders as a sidebar on wide viewports and a drop-down menu on mobile. Also renders `MobileMenuFooter`. Default: `Sidebar.astro`.
- **`MobileMenuFooter`** — bottom of the mobile drop-down. By default renders `ThemeSelect` and `LanguageSelect`. Default: `MobileMenuFooter.astro`.

#### Page sidebar (table of contents)

- **`PageSidebar`** — wraps the TOC. By default renders `TableOfContents` + `MobileTableOfContents`. Default: `PageSidebar.astro`.
- **`TableOfContents`** — the TOC for wider viewports. Default: `TableOfContents.astro`.
- **`MobileTableOfContents`** — the sticky drop-down TOC for mobile. Default: `MobileTableOfContents.astro`.

#### Content

- **`Banner`** — top-of-page announcement banner. Renders based on the `banner` frontmatter field. Default: `Banner.astro`.
- **`ContentPanel`** — layout wrapper around sections of the main content. Default: `ContentPanel.astro`.
- **`PageTitle`** — the page `<h1>`. Replacements should keep `id="_top"`. Default: `PageTitle.astro`.
- **`DraftContentNotice`** — dev-only notice on draft pages. Default: `DraftContentNotice.astro`.
- **`FallbackContentNotice`** — notice on pages using fallback content from another locale (multilingual sites only). Default: `FallbackContentNotice.astro`.
- **`Hero`** — the splash-style hero at the top of pages with a `hero:` frontmatter block. Default: `Hero.astro`.
- **`MarkdownContent`** — wraps the rendered Markdown body. By default applies Starlight's content typographic styles. Those styles are also available standalone at `@astrojs/starlight/style/markdown.css`, scoped to `.sl-markdown-content`. Default: `MarkdownContent.astro`.

#### Footer

- **`Footer`** — bottom of each page. By default renders `LastUpdated`, `Pagination`, `EditLink`. Default: `Footer.astro`.
- **`LastUpdated`** — the last-updated timestamp. Default: `LastUpdated.astro`.
- **`EditLink`** — "Edit page" link. Default: `EditLink.astro`.
- **`Pagination`** — prev/next page navigation. Default: `Pagination.astro`.

---

## Customizing route data (middleware)

Sometimes the right answer isn't replacing a component — it's changing the data the default component receives. Route middleware is a function called on every render that can read and modify the route data object.

This is conceptually parallel to component overrides:

- Component override → "I want different markup in this slot."
- Route middleware → "I want different data fed into the default markup."

### What route data is

Every Starlight page render produces a `starlightRoute` object accessible via `Astro.locals.starlightRoute`. It includes the current page entry, the sidebar tree, the TOC, the site title, locale info, the head tags, and more. See `references/configuration.md`'s and `https://starlight.astro.build/reference/route-data/` for the full shape.

The key field shapes:

- `dir: 'ltr' | 'rtl'`
- `lang: string` (BCP-47)
- `locale: string | undefined` (URL prefix; `undefined` for root locale)
- `siteTitle: string`
- `siteTitleHref: string` (e.g. `/` or `/en/`)
- `id: string` (page slug; empty string for the homepage)
- `isFallback: boolean | undefined`
- `entryMeta: { dir, lang }` (the page's own locale metadata; may differ from top-level when fallback content is in use)
- `entry` — Astro content-collection entry, with `entry.data.title`, `entry.data.description`, and the rest of the frontmatter
- `sidebar: SidebarEntry[]`
- `hasSidebar: boolean`
- `pagination: { prev?: Link; next?: Link }`
- `toc: { minHeadingLevel, maxHeadingLevel, items: TocItem[] } | undefined`
- `headings: { depth, slug, text }[]` — raw, before TOC filtering
- `lastUpdated: Date | undefined`
- `editUrl: URL | undefined`
- `head: HeadConfig[]`

### Defining middleware

```ts
// src/routeData.ts
import { defineRouteMiddleware } from '@astrojs/starlight/route-data';

export const onRequest = defineRouteMiddleware((context) => {
  // Read or modify context.locals.starlightRoute here.
});
```

Wire it in `astro.config.mjs`:

```js
starlight({
  title: 'My delightful docs site',
  routeMiddleware: './src/routeData.ts',
})
```

Example: add an exclamation mark to every page title.

```ts
export const onRequest = defineRouteMiddleware((context) => {
  const { entry } = context.locals.starlightRoute;
  entry.data.title = entry.data.title + '!';
});
```

### Multiple middleware

Pass an array — each middleware runs in order:

```js
starlight({
  routeMiddleware: ['./src/middleware-one.ts', './src/middleware-two.ts'],
})
```

### Awaiting later middleware

The second argument is `next()`. Awaiting it runs all middleware further down the stack before your code continues — useful for letting plugin middleware finish before applying your changes.

```ts
export const onRequest = defineRouteMiddleware(async (context, next) => {
  await next();
  const { entry } = context.locals.starlightRoute;
  entry.data.title = entry.data.title + '!';
});
```

### `StarlightRouteData` type

For helper functions that operate on route data outside the middleware file, the type is exported:

```ts
// src/route-utils.ts
import type { StarlightRouteData } from '@astrojs/starlight/route-data';

export function usePageTitleInTOC(starlightRoute: StarlightRouteData) {
  const overviewLink = starlightRoute.toc?.items[0];
  if (overviewLink) {
    overviewLink.text = starlightRoute.entry.data.title;
  }
}
```

```ts
// src/routeData.ts
import { defineRouteMiddleware } from '@astrojs/starlight/route-data';
import { usePageTitleInTOC } from './route-utils';

export const onRequest = defineRouteMiddleware((context) => {
  usePageTitleInTOC(context.locals.starlightRoute);
});
```

### When to choose middleware vs. a component override

- If you want the same default Starlight UI but driven by different data → middleware.
- If you want different markup → override.
- If both → do both (middleware shapes the data, the override consumes the shaped data).
- If this customization needs to ship as a reusable package → wrap it all in a plugin (see `references/plugins.md`), which can register both component overrides and middleware in one place.
