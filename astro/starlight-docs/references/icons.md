# Built-in Icons

Starlight's `icon=` props (on `<Card>`, `<Aside>`, `<LinkButton>`, `<TabItem>`, hero `actions`, `social` config entries, etc.) and the `<Icon name="…">` component accept names from a fixed catalog of bundled icons.

## Usage refresher

```js
import { Icon } from '@astrojs/starlight/components';
```

```mdx
<Icon name="star" />
<Icon name="starlight" label="The Starlight logo" />
```

When typing icon names in TypeScript, the `StarlightIcon` type is exported:

```ts
import type { StarlightIcon } from '@astrojs/starlight/types';

function getIconLabel(icon: StarlightIcon) {
  // icon is one of the names below
}
```

If the user passes a name that isn't in the catalog, Starlight will error at build time. Always check this list before using a name you're not certain about — close matches often differ by punctuation (e.g. `seti:c-sharp` vs. `seti:csharp`).

## All icon names

### General-purpose

`up-caret`, `down-caret`, `right-caret`, `left-caret`, `up-arrow`, `down-arrow`, `right-arrow`, `left-arrow`, `bars`, `translate`, `pencil`, `pen`, `document`, `add-document`, `setting`, `external`, `download`, `cloud-download`, `moon`, `sun`, `laptop`, `open-book`, `information`, `magnifier`, `forward-slash`, `close`, `error`, `warning`, `approve-check-circle`, `approve-check`, `rocket`, `star`, `puzzle`, `list-format`, `random`, `comment`, `comment-alt`, `heart`

### Social platforms / source-code hosts / communities

`github`, `gitlab`, `bitbucket`, `codePen`, `farcaster`, `discord`, `gitter`, `twitter`, `x.com`, `mastodon`, `codeberg`, `youtube`, `threads`, `linkedin`, `twitch`, `azureDevOps`, `microsoftTeams`, `instagram`, `stackOverflow`, `telegram`, `rss`, `facebook`, `email`, `phone`, `reddit`, `patreon`, `signal`, `slack`, `matrix`, `hackerOne`, `openCollective`, `blueSky`, `discourse`, `zulip`, `pinterest`, `tiktok`, `sourcehut`, `substack`

### Astro ecosystem and adjacent tools

`astro`, `alpine`, `pnpm`, `biome`, `bun`, `mdx`, `apple`, `linux`, `homebrew`, `nix`, `starlight`, `pkl`, `node`, `cloudflare`, `vercel`, `netlify`, `deno`, `jsr`, `nostr`, `backstage`, `confluence`, `jira`, `storybook`, `vscode`, `jetbrains`, `zed`, `vim`, `figma`, `sketch`, `npm`

### Browsers

`chrome`, `edge`, `firefox`, `safari`

### Seti file-type icons (for use with `<FileTree>` and inline)

These come from the Seti UI icon theme — useful for indicating file types in tutorials and file trees.

`seti:folder`, `seti:bsl`, `seti:mdo`, `seti:salesforce`, `seti:asm`, `seti:bicep`, `seti:bazel`, `seti:c`, `seti:c-sharp`, `seti:html`, `seti:cpp`, `seti:clojure`, `seti:coldfusion`, `seti:config`, `seti:crystal`, `seti:crystal_embedded`, `seti:json`, `seti:css`, `seti:csv`, `seti:xls`, `seti:cu`, `seti:cake`, `seti:cake_php`, `seti:d`, `seti:word`, `seti:elixir`, `seti:elixir_script`, `seti:hex`, `seti:elm`, `seti:favicon`, `seti:f-sharp`, `seti:git`, `seti:go`, `seti:godot`, `seti:gradle`, `seti:grails`, `seti:graphql`, `seti:hacklang`, `seti:haml`, `seti:mustache`, `seti:haskell`, `seti:haxe`, `seti:jade`, `seti:java`, `seti:javascript`, `seti:jinja`, `seti:julia`, `seti:karma`, `seti:kotlin`, `seti:dart`, `seti:liquid`, `seti:livescript`, `seti:lua`, `seti:markdown`, `seti:argdown`, `seti:info`, `seti:clock`, `seti:maven`, `seti:nim`, `seti:github`, `seti:notebook`, `seti:nunjucks`, `seti:npm`, `seti:ocaml`, `seti:odata`, `seti:perl`, `seti:php`, `seti:pipeline`, `seti:pddl`, `seti:plan`, `seti:happenings`, `seti:powershell`, `seti:prisma`, `seti:pug`, `seti:puppet`, `seti:purescript`, `seti:python`, `seti:react`, `seti:rescript`, `seti:R`, `seti:ruby`, `seti:rust`, `seti:sass`, `seti:spring`, `seti:slim`, `seti:smarty`, `seti:sbt`, `seti:scala`, `seti:ethereum`, `seti:stylus`, `seti:svelte`, `seti:swift`, `seti:db`, `seti:terraform`, `seti:tex`, `seti:default`, `seti:twig`, `seti:typescript`, `seti:tsconfig`, `seti:vala`, `seti:vite`, `seti:vue`, `seti:wasm`, `seti:wat`, `seti:xml`, `seti:yml`, `seti:prolog`, `seti:zig`, `seti:zip`, `seti:wgt`, `seti:illustrator`, `seti:photoshop`, `seti:pdf`, `seti:font`, `seti:image`, `seti:svg`, `seti:sublime`, `seti:code-search`, `seti:shell`, `seti:video`, `seti:audio`, `seti:windows`, `seti:jenkins`, `seti:babel`, `seti:bower`, `seti:docker`, `seti:code-climate`, `seti:eslint`, `seti:firebase`, `seti:firefox`, `seti:gitlab`, `seti:grunt`, `seti:gulp`, `seti:ionic`, `seti:platformio`, `seti:rollup`, `seti:stylelint`, `seti:yarn`, `seti:webpack`, `seti:lock`, `seti:license`, `seti:makefile`, `seti:heroku`, `seti:todo`, `seti:ignored`

## When the icon you want isn't here

The catalog is finite. If the user needs an icon that isn't in it:

- **A close substitute exists** — suggest the nearest match (e.g. `star` for a generic "favorite" icon).
- **It needs to be SVG** — use Astro's standard SVG support (inline `<svg>` in a custom component) instead of `<Icon>`. The `<Icon>` component is specifically for the built-in catalog.
- **A whole new icon set is needed** — point them at community icon libraries like `astro-icon` which integrate well with Astro projects, or use SVG sprites manually.
