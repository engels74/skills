# Plugins

Plugins are the highest rung of the Starlight customization ladder. They let you bundle config changes, component overrides, route middleware, translation strings, and Astro integrations into a single reusable extension. The community catalog is at `https://starlight.astro.build/resources/plugins/`.

To *use* an existing plugin, just import it and add it to the `plugins` array in your `starlight()` config — see the configuration reference. This document is about *building* plugins.

## Plugin shape

A Starlight plugin is an object with a `name` and a `hooks` map:

```ts
interface StarlightPlugin {
  name: string;
  hooks: {
    'i18n:setup'?: (options: {
      injectTranslations: (translations: Record<string, Record<string, string>>) => void;
    }) => void | Promise<void>;
    'config:setup': (options: {
      config: StarlightUserConfig;
      updateConfig: (newConfig: StarlightUserConfig) => void;
      addIntegration: (integration: AstroIntegration) => void;
      addRouteMiddleware: (config: { entrypoint: string; order?: 'pre' | 'post' | 'default' }) => void;
      astroConfig: AstroConfig;
      command: 'dev' | 'build' | 'preview';
      isRestart: boolean;
      logger: AstroIntegrationLogger;
      useTranslations: (lang: string) => I18nT;
      absolutePathToLang: (path: string) => string;
    }) => void | Promise<void>;
  };
}
```

A plugin can be a plain object, or (more commonly) a function that returns one — the function form lets you accept options:

```ts
export default function myPlugin(options) {
  return {
    name: 'my-plugin',
    hooks: {
      // …
    },
  };
}
```

Typing hook arguments without re-declaring the whole signature is supported via `HookParameters`:

```ts
import type { HookParameters } from '@astrojs/starlight/types';

function configSetup(options: HookParameters['config:setup']) {
  options.useTranslations('en');
}
```

## `name`

A required unique string identifier. Used as a prefix in log output and as a way for other plugins to detect this plugin's presence.

## Hooks

Hooks are functions Starlight calls at specific lifecycle points.

### `i18n:setup`

Called when Starlight initializes. Use this hook to inject translation strings so your plugin's UI strings are available everywhere translations are accessed — in `Astro.locals.t()` inside components and in the plugin's own `config:setup` hook via `useTranslations()`.

The one argument: `{ injectTranslations }`.

#### `injectTranslations(translations)`

`(translations: Record<string, Record<string, string>>) => void`

Adds or updates translation strings. Outer keys are BCP-47 language tags, inner keys are translation keys, inner values are translated strings.

```ts
export default {
  name: 'plugin-with-translations',
  hooks: {
    'i18n:setup'({ injectTranslations }) {
      injectTranslations({
        en: { 'myPlugin.doThing': 'Do the thing' },
        fr: { 'myPlugin.doThing': 'Faire le truc' },
      });
    },
  },
};
```

Once injected, components in user projects can use the key like any other built-in translation:

```astro
{Astro.locals.t('myPlugin.doThing')}
```

#### Typing plugin translations

User projects automatically get types for the strings your plugin injects. But while developing the plugin itself, you'll need to declare types yourself so `locals.t` is typed inside your plugin's source. In an env declaration file (e.g. `env.d.ts`):

```ts
declare namespace App {
  type StarlightLocals = import('@astrojs/starlight').StarlightLocals;
  interface Locals extends StarlightLocals {}
}

declare namespace StarlightApp {
  interface I18n {
    'myPlugin.doThing': string;
  }
}
```

To avoid repeating yourself, you can infer the `I18n` interface from a source-of-truth object:

```ts
// src/ui-strings.ts
export const UIStrings = {
  en: { 'myPlugin.doThing': 'Do the thing' },
  fr: { 'myPlugin.doThing': 'Faire le truc' },
};
```

```ts
// env.d.ts
declare namespace StarlightApp {
  type UIStrings = typeof import('./ui-strings').UIStrings.en;
  interface I18n extends UIStrings {}
}
```

### `config:setup`

The main plugin lifecycle hook, called during Astro's `astro:config:setup` integration phase. Use it to mutate Starlight's config, add Astro integrations the plugin depends on, register route middleware, log progress, and consult i18n strings.

Receives an options object with the following members.

#### `config`

`StarlightUserConfig` (read-only)

A snapshot of the user's Starlight config — including changes already applied by other plugins that ran before this one. Use this when you need to inspect or build on the user's existing config (e.g. "add to their existing `social` array rather than overwriting it").

#### `updateConfig(newConfig)`

`(newConfig: StarlightUserConfig) => void`

Apply changes to the Starlight config. Provide root-level keys you want to override; nested objects must be passed in full (no deep merging).

To *extend* a config value rather than replace it, spread the current value into your update. Example — append a social link without clobbering the user's existing ones:

```ts
export default {
  name: 'add-twitter-plugin',
  hooks: {
    'config:setup'({ config, updateConfig }) {
      updateConfig({
        social: [
          ...config.social,
          { icon: 'twitter', label: 'Twitter', href: 'https://twitter.com/astrodotbuild' },
        ],
      });
    },
  },
};
```

#### `addIntegration(integration)`

`(integration: AstroIntegration) => void`

Register an Astro integration that your plugin depends on. Common pattern: check whether the user already has the integration before adding it, to avoid duplicating it.

```ts
import react from '@astrojs/react';

export default {
  name: 'plugin-using-react',
  hooks: {
    'config:setup'({ addIntegration, astroConfig }) {
      const alreadyLoaded = astroConfig.integrations.find(
        ({ name }) => name === '@astrojs/react'
      );
      if (!alreadyLoaded) addIntegration(react());
    },
  },
};
```

#### `addRouteMiddleware(config)`

`(config: { entrypoint: string; order?: 'pre' | 'post' | 'default' }) => void`

Register a [route middleware handler](customization.md#customizing-route-data-middleware). `entrypoint` is a module specifier that exports an `onRequest` handler — for a published plugin, this is typically a sub-path of the plugin's own package:

```ts
export default {
  name: '@example/starlight-plugin',
  hooks: {
    'config:setup'({ addRouteMiddleware }) {
      addRouteMiddleware({
        entrypoint: '@example/starlight-plugin/route-middleware',
      });
    },
  },
};
```

`order` controls execution position:

- `'pre'` — run before user middleware.
- `'post'` — run after all other middleware.
- `'default'` (default) — runs in the order plugins were added.

Two plugins requesting the same `order` value run in the order they were added.

#### `astroConfig`

`AstroConfig` (read-only). The user's full Astro config. Useful for things like detecting which integrations are already installed (see the React example above).

#### `command`

`'dev' | 'build' | 'preview'`. Which command is currently running. Useful for conditional behavior (e.g. only inject analytics during `build`).

#### `isRestart`

`boolean`. `false` on initial dev server start, `true` on subsequent reloads (e.g. when the user edits `astro.config.mjs`).

#### `logger`

`AstroIntegrationLogger`. Standard Astro integration logger; messages are automatically prefixed with the plugin's `name`.

```ts
'config:setup'({ logger }) {
  logger.info('Starting long process…');
  // [my-plugin] Starting long process…
}
```

#### `useTranslations(lang)`

`(lang: string) => I18nT`

Get a translation function for a specific BCP-47 language tag. The returned function behaves like `Astro.locals.t()` available in components — same API, same interpolation/pluralization support.

```ts
'config:setup'({ useTranslations, logger }) {
  const t = useTranslations('zh-CN');
  logger.info(t('builtWithStarlight.label'));
  // [my-plugin] 基于 Starlight 构建
}
```

If your plugin injected its own strings via `i18n:setup`, those are accessible from `useTranslations()` here too.

#### `absolutePathToLang(path)`

`(path: string) => string`

Given an absolute file path, return the locale that file belongs to. Particularly useful when writing remark/rehype plugins that operate on Markdown content — the [vfile](https://github.com/vfile/vfile) format used by those plugins includes `file.path`, which you can pass here.

Combined with `useTranslations()`, this lets remark plugins generate localized output:

```ts
'config:setup'({ absolutePathToLang, useTranslations, logger }) {
  const lang = absolutePathToLang(
    '/absolute/path/to/project/src/content/docs/fr/index.mdx'
  );
  const t = useTranslations(lang);
  logger.info(t('aside.tip'));
  // [my-plugin] Astuce
}
```

## Common plugin patterns

- **Add a config value** — `updateConfig({ social: [...config.social, ...] })`. Always spread the existing value to avoid stomping on the user's config.
- **Add a component override** — `updateConfig({ components: { ...config.components, MyOverride: '@my-plugin/path/to/component.astro' } })`. Use a package sub-path for the entry so users don't need to copy files.
- **Add a custom CSS file** — `updateConfig({ customCss: [...(config.customCss ?? []), '@my-plugin/styles.css'] })`.
- **Add an Astro integration the plugin needs** — `addIntegration()`. Check `astroConfig.integrations` first to avoid double-adding.
- **Add route middleware** — `addRouteMiddleware({ entrypoint: '@my-plugin/middleware' })`.
- **Inject UI strings** — `i18n:setup` hook with `injectTranslations({ en: {...}, fr: {...} })`.

## Where the plugin lives

Most plugins are published to npm and installed by users. The conventional package name is `starlight-<thing>` (community) or `@astrojs/starlight-<thing>` (first-party). Document the user-facing options the plugin accepts, and what behavior they should expect — what config it touches, which components it overrides, what translation keys it adds.
