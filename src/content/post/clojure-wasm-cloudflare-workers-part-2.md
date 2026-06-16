---
title: "Clojure on CF Workers via WASM, Part 2: Deploying"
description: "Three patches to the GraalVM JS glue and Clojure WASM runs live on Cloudflare Workers."
publishDate: 2026-06-16
tags: ["clojure", "wasm", "cloudflare-workers"]
---

In [Part 1](/posts/clojure-wasm-cloudflare-workers-part-1/) I compiled Clojure to WASM using GraalVM Web Image and ran it in Node.js. Now let us get it into Cloudflare Workers.

## What GraalVM Web Image produces

Two files:

- `app.js.wasm` - the WASM binary, 5.3MB raw, 2.44MB gzipped
- `app.js` - 92KB JS runtime that loads the WASM, sets up imports, and starts the VM

The JS glue was written for Node.js and SpiderMonkey. CF Workers is neither. Three things needed fixing.

## Fix 1: WASM loading

CF Workers modules format uses static imports for WASM:

```js
import appWasm from './app.wasm';
// appWasm is a WebAssembly.Module
```

The GraalVM glue loads WASM by fetching a file path:

```js
async function wasmInstantiate(config, args) {
    const wasmPath = config.wasm_path || runtime.getCurrentFile() + ".wasm";
    const file = await runtime.fetchData(wasmPath);
    const result = await WebAssembly.instantiate(file, wasmImports);
    ...
}
```

There is no filesystem in CF Workers. I patched it to accept a pre-compiled `WebAssembly.Module` directly:

```js
async function wasmInstantiate(config, args) {
    if (config.wasm_module instanceof WebAssembly.Module) {
        const instance = await WebAssembly.instantiate(config.wasm_module, wasmImports);
        return {
            instance: instance,
            memory: instance.exports.memory,
        };
    }
    // original path loading below...
}
```

## Fix 2: Remove auto-run

At the bottom of the generated JS file there is this:

```js
const config = new GraalVM.Config();
GraalVM.run(load_cmd_args(), config).catch(console.error);
```

This runs when the module is imported in CF Workers. At that point we have not passed the WASM module yet. Remove it.

## Fix 3: ES module export

The glue uses `var GraalVM = {}` but does not export it. Add at the end:

```js
export { GraalVM };
```

## The worker

```js
import { GraalVM } from './graalvm-runtime.js';
import appWasm from './app.wasm';

export default {
  async fetch(request, env) {
    try {
      const config = new GraalVM.Config();
      config.wasm_module = appWasm;
      await GraalVM.run([], config);
      return new Response('Clojure WASM ran on Cloudflare Workers', {
        headers: { 'content-type': 'text/plain' },
      });
    } catch (e) {
      return new Response(`Error: ${e.message}\n\n${e.stack}`, { status: 500 });
    }
  },
};
```

Note: no `[wasm_modules]` in wrangler.toml. That is for the Service Worker format. Static import handles it in modules format.

## Deploy

```bash
pnpm wrangler deploy
# Total Upload: 5525.32 KiB / gzip: 2519.77 KiB
# Deployed: https://clj-wasm-worker.oddiy.workers.dev
```

```bash
curl https://clj-wasm-worker.oddiy.workers.dev
# Clojure WASM ran on Cloudflare Workers
```

It works.

## Performance

I measured against a plain JS worker returning the same response. Used `hey` with 200 requests at 20 concurrency.

| | JS | Clojure WASM |
|---|---|---|
| Bundle (gzip) | 0.21 KB | 2,520 KB |
| p50 latency | 27ms | 67ms |
| p90 latency | 82ms | 117ms |
| p99 latency | 75ms | 349ms |
| Average (warm) | 34ms | 85ms |

Warm p50 is about 2.5x slower. The p99 spike at 349ms is probably the GraalVM WASM runtime booting on a new isolate.

Bundle size is the bigger concern. At 2.52MB gzipped this hello-world fits inside the free tier limit of 3MB. A real application will push past it.

## Is this actually useful

Probably not for most use cases. The WASM sandbox has no filesystem, no threads, no network I/O. The JVM libraries that make Clojure useful on the server do not work here. Ring, http-kit, JDBC, none of it.

The one case where this could make sense: you have pure business logic already written in Clojure, things like validation, calculations, data transformation, and you want to run that same code at the edge without rewriting it. One codebase, two targets.

Beyond that it is mostly an interesting exploration.

## What does not work yet

The `-main` function runs but I cannot call individual Clojure functions from the request handler yet. GraalVM Web Image has annotations for exporting functions to JavaScript. That is the next thing to explore.
