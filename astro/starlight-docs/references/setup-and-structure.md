# Setup and Project Structure

This reference covers how to create a Starlight site (either fresh or by adding Starlight to an existing Astro project), what the resulting file layout looks like, and a few advanced setup scenarios (using Starlight at a subpath, enabling SSR).

## Creating a new project

Use the official template:

```bash
# npm
npm create astro@latest -- --template starlight

# pnpm
pnpm create astro --template starlight

# Yarn
yarn create astro --template starlight
```

This scaffolds a working site with the integration installed, a sample content collection config, and a few starter pages. Start the dev server with `npm run dev` (or `pnpm dev` / `yarn dev`) to preview.

You can also try Starlight in-browser via StackBlitz: `https://stackblitz.com/github/withastro/starlight/tree/main/examples/basics`.

There are pre-configured templates for two common variants too:

```bash
# Tailwind pre-wired
npm create astro@latest -- --template starlight/tailwind

# Markdoc pre-wired
npm create astro@latest -- --template starlight/markdoc
```

## Adding Starlight to an existing Astro project

Three steps:

1. Run the `astro add` command, which installs `@astrojs/starlight` and adds it to the `integrations` array in `astro.config.mjs`:

   ```bash
   npx astro add starlight        # or `pnpm astro add starlight` / `yarn astro add starlight`
   ```

2. Set at least a `title` in the integration options:

   ```js
   // astro.config.mjs
   import { defineConfig } from 'astro/config';
   import starlight from '@astrojs/starlight';

   export default defineConfig({
     integrations: [
       starlight({
         title: 'My delightful docs site',
       }),
     ],
   });
   ```

3. Configure the `docs` content collection (and optionally `i18n`) in `src/content.config.ts`:

   ```ts
   import { defineCollection } from 'astro:content';
   import { docsLoader } from '@astrojs/starlight/loaders';
   import { docsSchema } from '@astrojs/starlight/schema';

   export const collections = {
     docs: defineCollection({ loader: docsLoader(), schema: docsSchema() }),
   };
   ```

4. Add an `index.md` (or `.mdx` / `.mdoc`) to `src/content/docs/` with at minimum a `title` in frontmatter. That becomes the homepage.

If the project predates Astro v5 and you can't migrate collections immediately, Starlight is compatible with the [`legacy.collectionsBackwardsCompat` flag](https://docs.astro.build/en/reference/legacy-flags/#collectionsbackwardscompat).

## Project layout

Starlight projects follow normal Astro structure. The Starlight-relevant directories:

```
public/                        Static assets served as-is (favicon, robots.txt, OG images, PDFs).
src/
├── assets/                    Images and similar that should go through Astro's asset pipeline.
├── components/                Your own custom components (.astro, .jsx, .svelte, etc.).
├── content/
│   ├── docs/                  Your documentation pages (.md / .mdx / .mdoc).
│   └── i18n/                  Optional: JSON/YAML translation files for UI strings.
├── content.config.ts          Defines the `docs` (and optionally `i18n`) collection.
├── fonts/                     Optional: local font files + a font-face.css.
├── styles/                    Optional: custom CSS files referenced from `customCss`.
└── pages/                     Optional: custom Astro/HTML pages outside the docs collection.
astro.config.mjs               Astro config; this is where the `starlight()` integration lives.
package.json
tsconfig.json
```

Files in `src/content/docs/` are routed by filename. `src/content/docs/guides/intro.md` becomes `/guides/intro/`. Filenames starting with `_` are ignored. The default slug generation lowercases and strips special characters; you can override it via `docsLoader({ generateId })`.

Files in `src/pages/` use Astro's standard file-based routing and are independent of Starlight's docs layout. Use this when you need a page with a completely custom layout (or to wrap your own content in the `<StarlightPage>` component from `@astrojs/starlight/components/StarlightPage.astro`).

## Custom pages with Starlight's design

If you need a page outside `src/content/docs/` (say, a programmatically generated one) but still want it to look like the rest of the site, use the `<StarlightPage>` component:

```astro
---
// src/pages/custom-page/example.astro
import StarlightPage from '@astrojs/starlight/components/StarlightPage.astro';
---

<StarlightPage frontmatter={{ title: 'My custom page' }}>
  <p>Page content here.</p>
</StarlightPage>
```

`<StarlightPage>` accepts most of the same frontmatter fields as a regular page (`title` is required), plus a few extras:

- `sidebar` — array of sidebar items to use *only on this page* (overrides the global sidebar). Same shape as the `sidebar` config option.
- `hasSidebar` — boolean; defaults to `false` for splash templates, `true` otherwise.
- `headings` — array of `{ depth, slug, text }` if you want Starlight to build a table of contents from them.
- `dir` / `lang` — override writing direction and language.
- `isFallback` — mark the page as using fallback content from another locale.

A few frontmatter fields behave differently here vs. regular Markdown pages: `slug` is set automatically from the URL, `editUrl` needs a full URL, the sidebar frontmatter shortcuts don't apply (since the page isn't in a collection), and `draft: true` only shows a notice rather than excluding the page from production builds.

To add headings with the same anchor-link styling Markdown gets, use `<AnchorHeading>` from `@astrojs/starlight/components/AnchorHeading.astro`. It accepts a required `level` (1–6) and `id`, plus any standard HTML attributes.

## Using Starlight at a subpath of an existing Astro site

If Starlight pages should live under `/guides/` instead of the site root, put all your docs content inside a nested directory: `src/content/docs/guides/`. The directory mirrors the URL path you want. (This is a known rough edge — there's mention in the docs of improving this in the future.)

## Server-side rendering (SSR)

By default, Starlight pages are pre-rendered to static HTML regardless of the rest of your Astro project's output mode. To opt out of pre-rendering, set `prerender: false` in the `starlight()` integration options and follow Astro's [on-demand rendering guide](https://docs.astro.build/en/guides/on-demand-rendering/) to add a server adapter.

Note: Pagefind (the default search) cannot be used when `prerender: false` — it requires a built static site to index.

If you're using the Cloudflare adapter for SSR, add the `nodejs_compat` compatibility flag to your Wrangler configuration.

## Updating

```bash
npx @astrojs/upgrade       # or `pnpm dlx @astrojs/upgrade` / `yarn dlx @astrojs/upgrade`
```

This upgrades both Astro and Starlight (and any other `@astrojs/*` packages) to their latest compatible versions. The [Starlight changelog](https://github.com/withastro/starlight/blob/main/packages/starlight/CHANGELOG.md) is worth skimming before updates since Starlight is still in beta.

## Troubleshooting checklist

If something isn't working, walk this list before going deeper:

- `starlight()` is actually in `integrations:` of `astro.config.mjs`, not somewhere else.
- The `docs` collection is defined in `src/content.config.ts` and uses both `docsLoader()` and `docsSchema()` (not just one of them).
- Page files live in `src/content/docs/`, not `src/pages/` or `src/docs/`.
- Every page has at least a `title` frontmatter field.
- File names don't start with `_` (those are ignored by `docsLoader()`).
- For MDX / Markdoc files: the Astro integration for that format is installed (`@astrojs/mdx` is bundled in the Starlight template; `@astrojs/markdoc` plus `@astrojs/starlight-markdoc` must be added manually).
- For component imports in MDX: imported from `@astrojs/starlight/components`, not a local path.

If the answer truly isn't in the docs, the [Astro Discord](https://astro.build/chat) `#starlight` channel and the [Starlight GitHub issues](https://github.com/withastro/starlight/issues) are good next stops.
