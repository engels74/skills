# Styling

Custom CSS, Tailwind integration, theming, and fonts.

## Where styles live

The minimal pattern: put a CSS file under `src/styles/` (or wherever you like), and add its path to the `customCss` array in the `starlight()` integration.

```css
/* src/styles/custom.css */
:root {
  --sl-content-width: 50rem;
  --sl-text-5xl: 3.5rem;
}
```

```js
// astro.config.mjs
starlight({
  title: 'Docs With Custom CSS',
  customCss: ['./src/styles/custom.css'],
})
```

`customCss` accepts both local paths and npm modules — useful for Fontsource packages (`'@fontsource/roboto'`).

The full set of CSS custom properties Starlight exposes lives in `props.css`:
`https://github.com/withastro/starlight/blob/main/packages/starlight/style/props.css`

When users ask "how do I change X?", check that file first — Starlight likely has a variable for it (content width, max text size, accent color, gray ramp, fonts, spacing scale, etc.).

## Cascade layers

Starlight uses CSS cascade layers internally so its own styles can be overridden predictably. Any of your unlayered CSS will override the default Starlight styles automatically.

If you also use cascade layers, declare their relationship with Starlight's `starlight` layer:

```css
/* src/styles/custom.css */
@layer my-reset, starlight, my-overrides;
```

`my-reset` rules apply before Starlight's (Starlight can still override them); `my-overrides` rules apply after Starlight's (your overrides win).

## Theming (colors)

Starlight uses two color ramps and one accent ramp, all controllable via CSS custom properties:

- `--sl-color-accent-*` — used for links and "current item" highlighting in navigation.
- `--sl-color-gray-*` — used for backgrounds and borders.
- `--sl-color-text-accent` (and related text variants) — used for accent-colored text.

There's an interactive theme editor on the Starlight docs page (`https://starlight.astro.build/guides/css-and-tailwind/#color-theme-editor`) that generates the exact CSS or Tailwind config to paste in. When users ask for a custom color palette, point them there — manually computing accessible color ramps by hand is error-prone.

Site fonts:

- `--sl-font` — primary UI/content font
- `--sl-font-mono` — code font

## Tailwind CSS

Tailwind v4 in Astro is wired up via the Tailwind Vite plugin. Starlight ships a complementary CSS package (`@astrojs/starlight-tailwind`) that:

- Configures Tailwind's `dark:` variants to match Starlight's dark mode.
- Maps Tailwind theme colors and fonts into Starlight's CSS variables.
- Restores essential parts of Tailwind's Preflight reset (Starlight overrides Preflight by default because docs content needs typographic defaults).

### Scaffolding a new project with Tailwind

```bash
npm create astro@latest -- --template starlight/tailwind
```

### Adding Tailwind to an existing Starlight project

1. Add the Tailwind Vite plugin:

   ```bash
   npx astro add tailwind     # or pnpm / yarn equivalent
   ```

2. Install Starlight's Tailwind compatibility package:

   ```bash
   npm install @astrojs/starlight-tailwind
   ```

3. Replace the contents of `src/styles/global.css` (scaffolded by `astro add tailwind`) with:

   ```css
   @layer base, starlight, theme, components, utilities;

   @import '@astrojs/starlight-tailwind';
   @import 'tailwindcss/theme.css' layer(theme);
   @import 'tailwindcss/utilities.css' layer(utilities);
   ```

   This defines the layer order, pulls in the Starlight-Tailwind bridge, and imports Tailwind's theme and utility styles.

4. Add the global CSS as the *first* entry in `customCss`:

   ```js
   import { defineConfig } from 'astro/config';
   import starlight from '@astrojs/starlight';
   import tailwindcss from '@tailwindcss/vite';

   export default defineConfig({
     integrations: [
       starlight({
         title: 'Docs with Tailwind',
         customCss: ['./src/styles/global.css'],
       }),
     ],
     vite: { plugins: [tailwindcss()] },
   });
   ```

### Styling Starlight from Tailwind theme tokens

When the Tailwind compatibility package is installed, these custom properties (set inside a `@theme { … }` block in your global CSS) will override Starlight's defaults:

- `--color-accent-*` — links and current-item highlights.
- `--color-gray-*` — backgrounds and borders.
- `--font-sans` — UI and content text.
- `--font-mono` — code text.

```css
/* src/styles/global.css */
@layer base, starlight, theme, components, utilities;

@import '@astrojs/starlight-tailwind';
@import 'tailwindcss/theme.css' layer(theme);
@import 'tailwindcss/utilities.css' layer(utilities);

@theme {
  --font-sans: 'Atkinson Hyperlegible';
  --font-mono: 'IBM Plex Mono';

  --color-accent-50:  var(--color-indigo-50);
  --color-accent-100: var(--color-indigo-100);
  /* … all the way through 950 */
  --color-accent-950: var(--color-indigo-950);

  --color-gray-50:  var(--color-zinc-50);
  /* … all the way through 950 */
  --color-gray-950: var(--color-zinc-950);
}
```

The Tailwind color presets to use as bases are spelled out in the docs example: Indigo for accent (closest to Starlight default), Zinc for gray (closest to default).

### Multiple Tailwind configurations

When using Starlight at a subpath of a larger site, or when adding custom pages outside the docs collection, you may want a different Tailwind config for non-Starlight pages — typically because you want Tailwind's full Preflight reset there but Starlight's typographic defaults inside the docs.

Starting point for non-Starlight pages:

```css
/* src/styles/custom-pages-tailwind.css */
@import 'tailwindcss';   /* no Starlight bridge, no extra layers */
```

Apply it by importing it from layouts used by those pages:

```astro
---
// src/layouts/CustomPageLayout.astro
import '../styles/custom-pages-tailwind.css';
---
```

Starlight pages continue to use the global CSS from step 3 above.

## Fonts

Starlight defaults to a system font stack (no extra bandwidth to download font files). To switch to a custom font, you have two practical options.

### Local font files

1. Drop the font files in `src/fonts/` and create a `font-face.css` next to them.

2. Declare `@font-face` rules using relative paths:

   ```css
   /* src/fonts/font-face.css */
   @font-face {
     font-family: 'Custom Font';
     src: url('./CustomFont.woff2') format('woff2');
     font-weight: normal;
     font-style: normal;
     font-display: swap;
   }
   ```

3. Add the CSS file to `customCss`:

   ```js
   starlight({
     customCss: ['./src/fonts/font-face.css'],
   })
   ```

### Fontsource (Google Fonts / open-source fonts via npm)

[Fontsource](https://fontsource.org/) packages popular open-source fonts as npm modules with ready-made CSS.

1. Find your font at `https://fontsource.org/` and install the package:

   ```bash
   npm install @fontsource/ibm-plex-serif
   ```

2. Add the specific weight/style CSS files you want to use to `customCss`:

   ```js
   starlight({
     customCss: [
       '@fontsource/ibm-plex-serif/400.css',
       '@fontsource/ibm-plex-serif/600.css',
     ],
   })
   ```

   See `https://fontsource.org/docs/getting-started/install#4-weights-and-styles` for which files exist for each font.

### Applying the font

Once the font is loaded, point Starlight at it from your custom CSS:

```css
/* src/styles/custom.css */
:root {
  --sl-font: 'IBM Plex Serif', serif;
}
```

For more targeted application (e.g. only main content, not the sidebars):

```css
main {
  font-family: 'IBM Plex Serif', serif;
}
```

Remember to add the styles file to `customCss` too.

## When to step outside CSS

For changes that CSS alone can't accomplish — restructuring layout, conditional rendering, swapping out interactive widgets — `customCss` isn't enough. Go to `references/customization.md` to override components themselves.
