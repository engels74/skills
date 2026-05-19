# Built-in Components

Starlight ships a small library of components designed for documentation. All are available via `@astrojs/starlight/components` in MDX, and as lowercased `{% tag %}` syntax in Markdoc (with the Starlight Markdoc preset configured).

This reference is organized by component. Each section shows the import, MDX usage, Markdoc equivalent, and the available props.

## Plain Markdown limitation

Built-in components don't work in plain `.md` files — they need MDX (`.mdx`) or Markdoc (`.mdoc`). For `.md` content, use the Markdown directive equivalents where available:

- `<Aside>` → `:::note` / `:::tip` / `:::caution` / `:::danger`
- Code with file titles, line highlights, frames → fence options like ` ```js title="…" {2-3} `

If you need `<Card>`, `<Tabs>`, `<Steps>`, etc. in a page that is `.md`, convert it to `.mdx`.

## Compatibility with Starlight's content styles

Starlight applies default styling to Markdown content (margin between elements, etc.). If those styles conflict with your custom component's appearance, set `class="not-content"` on the component's root element to disable them.

## Component props typing

Use Astro's `ComponentProps<typeof Badge>` when wrapping or extending built-in components — props are typed even if not exported by name:

```astro
---
import type { ComponentProps } from 'astro/types';
import { Badge } from '@astrojs/starlight/components';
type BadgeProps = ComponentProps<typeof Badge>;
---
```

---

## `<Card>` — bordered content block

```js
import { Card } from '@astrojs/starlight/components';
```

```mdx
<Card title="Stars" icon="star">
  Sirius, Vega, Betelgeuse
</Card>
```

```markdoc
{% card title="Stars" icon="star" %}
Sirius, Vega, Betelgeuse
{% /card %}
```

Props:

- `title` *(required)* — heading text shown at the top of the card.
- `icon` — name of a [built-in icon](icons.md). Optional.

Use `<CardGrid>` (below) to display several cards side-by-side.

---

## `<LinkCard>` — prominent link

```js
import { LinkCard } from '@astrojs/starlight/components';
```

```mdx
<LinkCard title="Authoring Markdown" href="/guides/authoring-content/" />

<LinkCard
  title="Internationalization"
  href="/guides/i18n/"
  description="Configure Starlight to support multiple languages."
/>
```

```markdoc
{% linkcard title="Authoring Markdown" href="/guides/authoring-content/" /%}
```

Props (plus any other `<a>` HTML attributes):

- `title` *(required)* — the visible link label.
- `href` *(required)* — the URL.
- `description` — short text shown below the title.

Like `<Card>`, group multiple with `<CardGrid>`.

---

## `<CardGrid>` — responsive grid wrapper

```js
import { Card, CardGrid } from '@astrojs/starlight/components';
```

```mdx
<CardGrid>
  <Card title="Check this out" icon="open-book">Interesting content.</Card>
  <Card title="Other feature" icon="information">More information.</Card>
</CardGrid>

<CardGrid stagger>
  <Card title="Feature A">…</Card>
  <Card title="Feature B">…</Card>
</CardGrid>
```

```markdoc
{% cardgrid stagger=true %}
{% card title="…" %}…{% /card %}
{% /cardgrid %}
```

Props:

- `stagger` — boolean. Vertically offsets the second column for visual interest. Useful on homepages.

Works with both `<Card>` and `<LinkCard>` children (or a mix).

---

## `<Aside>` — admonitions / callouts

```js
import { Aside } from '@astrojs/starlight/components';
```

```mdx
<Aside>Some content in an aside.</Aside>
<Aside type="caution">Some cautionary content.</Aside>
<Aside type="tip">
  Other content is also supported in asides.

  ```js
  // Including code snippets.
  ```
</Aside>
<Aside type="danger">Do not give your password to anyone.</Aside>

<Aside type="caution" title="Watch out!">
  A warning aside *with* a custom title.
</Aside>

<Aside type="tip" icon="starlight">
  A tip aside *with* a custom icon.
</Aside>
```

```markdoc
{% aside type="caution" title="Watch out!" %}
A warning aside *with* a custom title.
{% /aside %}
```

Props:

- `type` — `'note' | 'tip' | 'caution' | 'danger'`. Default `'note'`. Controls color, default icon, default title.
- `title` — overrides the default title for the type.
- `icon` — overrides the default icon. Must be a name from the icon catalog.

In `.md` files, the `:::note` / `:::tip` / `:::caution` / `:::danger` Markdown directive syntax is the equivalent and works the same way (including `:::tip[Custom title]{icon="heart"}`).

---

## `<Badge>` — small inline label

```js
import { Badge } from '@astrojs/starlight/components';
```

```mdx
<Badge text="Note" variant="note" />
<Badge text="Success" variant="success" />
<Badge text="Tip" variant="tip" />
<Badge text="Caution" variant="caution" />
<Badge text="Danger" variant="danger" />

<Badge text="New" size="small" />
<Badge text="New and improved" size="medium" />
<Badge text="New, improved, and bigger" size="large" />

<Badge text="Custom" variant="success" style={{ fontStyle: 'italic' }} />
```

Props (plus any `<span>` HTML attributes):

- `text` *(required)* — the badge label.
- `variant` — `'note' | 'danger' | 'success' | 'caution' | 'tip' | 'default'`. Default `'default'` (uses the site accent color).
- `size` — `'small' | 'medium' | 'large'`.

---

## `<Code>` — render code from a string

Use this when you can't use a Markdown fenced code block — typically when the code comes from a file import, a CMS, or another dynamic source.

```js
import { Code } from '@astrojs/starlight/components';
```

```mdx
import importedCode from '/tsconfig.json?raw';

<Code code={importedCode} lang="json" title="tsconfig.json" />

<Code
  code={`console.log('Inline string source')`}
  lang="js"
  title="example.js"
  mark={['file', 'CMS']}
/>
```

Vite's `?raw` import suffix imports a file as a string — handy for displaying real config files alongside docs.

The full prop list is documented at `https://expressive-code.com/key-features/code-component/`. Common ones: `code`, `lang`, `title`, `mark`, `ins`, `del`, `frame`, `meta`.

---

## `<FileTree>` — directory structure

```js
import { FileTree } from '@astrojs/starlight/components';
```

```mdx
<FileTree>
- astro.config.mjs
- package.json
- src
  - components
    - **Header.astro** an **important** file
    - Title.astro
  - pages/
- …
</FileTree>
```

Inside `<FileTree>`, write a Markdown unordered list. Items ending in `/` (e.g. `pages/`) are directories without specific listed contents; items with nested lists are directories whose contents are shown. Bold (`**name**`) emphasizes a file. Text after the filename becomes a comment (supports inline Markdown). `...` or `…` becomes a placeholder. Wrap names in backticks to escape special characters (underscores, spaces).

`<FileTree>` accepts no props.

---

## `<Icon>` — standalone icon

```js
import { Icon } from '@astrojs/starlight/components';
```

```mdx
<Icon name="star" />
<Icon name="starlight" label="The Starlight logo" />
<Icon name="star" color="goldenrod" size="2rem" />
<Icon name="rocket" color="var(--sl-color-text-accent)" />
```

Props:

- `name` *(required)* — must be a name from the icon catalog. TypeScript: import the `StarlightIcon` type from `@astrojs/starlight/types`.
- `label` — accessible label for screen readers. When omitted, the icon is hidden from assistive tech entirely. If an icon is the *only* content of an interactive element (e.g. a link), set `label`.
- `size` — CSS size value (`'1rem'`, `'24px'`, etc.).
- `color` — CSS color value (named color, hex, `var(--…)`, etc.).
- `class` — extra CSS classes.

---

## `<LinkButton>` — visually prominent call-to-action

```js
import { LinkButton } from '@astrojs/starlight/components';
```

```mdx
<LinkButton href="/getting-started/">Get started</LinkButton>

<LinkButton href="/reference/configuration/" variant="secondary">
  Configuration Reference
</LinkButton>

<LinkButton
  href="https://docs.astro.build"
  variant="secondary"
  icon="external"
  iconPlacement="start"
>
  Related: Astro
</LinkButton>
```

Props (plus any `<a>` attributes):

- `href` *(required)* — destination URL.
- `variant` — `'primary' | 'secondary' | 'minimal'`. Default `'primary'`.
- `icon` — name of a built-in icon to include in the button.
- `iconPlacement` — `'start' | 'end'`. Default `'end'`.

---

## `<Steps>` — styled numbered list

Wraps a standard Markdown ordered list and renders it as a numbered, step-by-step procedure. Especially useful when each step is long or contains rich content (code blocks, asides, sub-lists).

```js
import { Steps } from '@astrojs/starlight/components';
```

```mdx
<Steps>
1. Import the component into your MDX file:

   ```js
   import { Steps } from '@astrojs/starlight/components';
   ```

2. Wrap `<Steps>` around your ordered list items.
</Steps>
```

`<Steps>` accepts no props. The content must be a single ordered list (`1.`, `2.`, …).

---

## `<Tabs>` and `<TabItem>` — tabbed content

```js
import { Tabs, TabItem } from '@astrojs/starlight/components';
```

```mdx
<Tabs>
  <TabItem label="Stars">Sirius, Vega, Betelgeuse</TabItem>
  <TabItem label="Moons">Io, Europa, Ganymede</TabItem>
</Tabs>

<Tabs>
  <TabItem label="Stars" icon="star">Sirius, Vega, Betelgeuse</TabItem>
  <TabItem label="Moons" icon="moon">Io, Europa, Ganymede</TabItem>
</Tabs>
```

`<Tabs>` props:

- `syncKey` — string. When multiple `<Tabs>` on the same page share a `syncKey` AND share the same `<TabItem>` labels, the active tab stays in sync across them. Common pattern: package managers, operating systems. The selection persists across page navigations.

`<TabItem>` props:

- `label` *(required)* — the tab label.
- `icon` — name of a built-in icon to display next to the label.

Example of synced tab groups (note that the labels match across the groups):

```mdx
<Tabs syncKey="constellations">
  <TabItem label="Orion">Bellatrix, Rigel, Betelgeuse</TabItem>
  <TabItem label="Gemini">Pollux, Castor A, Castor B</TabItem>
</Tabs>

<Tabs syncKey="constellations">
  <TabItem label="Orion">HD 34445 b, Gliese 179 b, Wasp-82 b</TabItem>
  <TabItem label="Gemini">Pollux b, HAT-P-24b, HD 50554 b</TabItem>
</Tabs>
```

---

## Quick decision guide

When users describe what they want but don't know the component name:

- "Box with a heading and content" → `<Card>` (decorative) or `<LinkCard>` (acts as a link)
- "Side-by-side cards / feature grid" → `<CardGrid>` wrapping `<Card>` or `<LinkCard>`
- "Highlighted box / callout / admonition / warning" → `<Aside>` (or `:::note` directive in `.md`)
- "Small inline label / pill / chip" → `<Badge>`
- "Code with file name shown" → fenced code block with a filename comment, or `<Code>` for dynamic content
- "Tree of files/folders" → `<FileTree>`
- "Single icon inline" → `<Icon>` (only works in MDX/Markdoc; for icons in nav/cards/asides, use their `icon=` prop)
- "Big call-to-action button" → `<LinkButton>`
- "Numbered procedure / installation steps" → `<Steps>`
- "Tabs (e.g. npm vs pnpm vs yarn)" → `<Tabs>` + `<TabItem>` with matching `syncKey`s
