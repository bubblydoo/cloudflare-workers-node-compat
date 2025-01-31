# Cloudflare Workers Compat

> [!IMPORTANT]
> The shims in this module are probably not needed anymore, because of [node_compat_v2](https://developers.cloudflare.com/workers/runtime-apis/nodejs/).
> It might still be useful to set up esbuild externals, or to fix edge-cases.

This module is highly experimental, and incomplete.

The goal is to provide shim modules for every builtin Node module, so that you can use existing Node code in Cloudflare Workers.

It compiles Deno's node compatibility layer to modules that are Cloudflare Workers-compatible, or uses other browser-compatible modules.

It also includes shims for Deno, in case you want to use a Deno module in a Cloudflare Worker (e.g. [@bubblydoo/cloudflare-workers-postgres-client](https://github.com/bubblydoo/cloudflare-workers-postgres-client)).

Next to that `eval` and `new Function` are also replaced with warning functions.

```bash
npm i cloudflare-workers-compat # do not install as a dev dependency
```

To use it, use esbuild:

```ts
// build.mjs

import { build } from "esbuild";
import bundlerConfig from "cloudflare-workers-compat/bundler-config";
import { aliasPlugin, outputReplacesPlugin, aliasWithPrefixPlugin } from "cloudflare-workers-compat/esbuild";

const compatConfig = await bundlerConfig({
  // replaces node builtins like `assert`, `util`, ...
  nodeBuiltinModules: true,
  // replaces Node globals like `global`
  nodeGlobals: true,
  // replaces `eval` and `new Function` with warning functions
  workerIncompatibles: true,
  // replaces browser globals like `navigator`
  browserGlobals: true,
  // keeps supported node modules external and prefixes them with `node:`,
  // for when the "nodejs_compat" compatibility flag is enabled
  workersNodejsCompatV2: true,
});

await build({
  ...,
  metafile: true,
  define: compatConfig.define,
  inject: compatConfig.inject,
  externals: compatConfig.externals,
  plugins: [
    aliasPlugin(compatConfig.aliases),
    outputReplacesPlugin(compatConfig.replaces),
    aliasWithPrefixPlugin(compatConfig.aliasWithPrefix)
  ]
});
```

Shims are included for:

- `assert` (through `assert`)
- `buffer` (through `buffer`)
- `console` (through `console-browserify`)
- `crypto` (through `crypto-browserify`)
- `events` (through `https://deno.land/std/node/events.ts`)
- `fs` (empty object)
- `os` (through `os-browserify/browser`)
- `path` (through `path-browserify`)
- `process` (through `https://deno.land/std/node/process.ts`)
- `querystring` (through `qs`)
- `stream` (through `stream-browserify`)
- `string_decoder` (through `https://deno.land/std/node/string_decoder.ts`)
- `timers` (exports `setTimeout`, `clearTimeout`, `setInterval`, `clearInterval`)
- `tty` (through `https://deno.land/std/node/tty.ts`)
- `url` (through `url-shim`)
- `util` (through `https://deno.land/std/node/util.ts`)
