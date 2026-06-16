---
title: "Clojure on CF Workers via WASM, Part 2: Deploying"
description: "Two patches to the GraalVM JS glue and a static WASM import — Clojure WASM now runs live on Cloudflare Workers."
publishDate: 2026-06-16
tags: ["clojure", "wasm", "cloudflare-workers"]
---

In [Part 1](/posts/clojure-wasm-cloudflare-workers-part-1/) we compiled a Clojure program to WASM using GraalVM Web Image and verified it runs in Node.js. This part gets it running on Cloudflare Workers.

## What GraalVM Web Image produces

The compiler outputs two files:

- `app.js.wasm` — the WASM binary (5.3MB raw, 2.44MB gzipped)
- `app.js` — a 92KB JS runtime/glue layer that loads the WASM, sets up imports, and starts the VM

The JS glue is designed for Node.js and SpiderMonkey. CF Workers is neither. Three things needed fixing.

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

No filesystem in CF Workers. Patch it to accept a pre-compiled `WebAssembly.Module`:

```js
async function wasmInstantiate(config, args) {
    if (config.wasm_module instanceof WebAssembly.Module) {
        const instance = await WebAssembly.instantiate(config.wasm_module, wasmImports);
        return {
            instance: instance,
            memory: instance.exports.memory,
        };
    }
    // original path-based loading below ...
}
```

## Fix 2: Remove auto-run

At the bottom of the generated JS, the runtime immediately runs the Clojure main method:

```js
const config = new GraalVM.Config();
GraalVM.run(load_cmd_args(), config).catch(console.error);
```

This runs at module import time in CF Workers — before we have a chance to pass our WASM module. Remove it.

## Fix 3: Add ES module export

The glue uses `var GraalVM = {}` in global scope but does not export it. Add at the end:

```js
export { GraalVM };
```

## The worker

```js
// index.js
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

## wrangler.toml

```toml
name = "clj-wasm-worker"
main = "index.js"
compatibility_date = "2025-07-18"
```

No `[wasm_modules]` — that is for the Service Worker format. Static import handles it in modules format.

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

It is live.

## Measurements

I measured against a plain JS worker returning the same response, using `hey` with 200 requests at 20 concurrency.

| Metric | JS | Clojure WASM |
|--------|----|--------------|
| Bundle (gzip) | 0.21 KB | 2,520 KB |
| p50 latency | 27ms | 67ms |
| p90 latency | 82ms | 117ms |
| p99 latency | 75ms | 349ms |
| Avg (warm) | 34ms | 85ms |
| Build time | — | ~14s |

The warm p50 is about 2.5× slower. The p99 spike (349ms) is probably the GraalVM WASM runtime booting on a fresh isolate — each time CF Workers spins up a new instance, the Clojure VM has to initialize.

Bundle size is the bigger concern. At 2.52MB gzipped the hello-world fits inside the free tier (3MB). Any real application will likely push past it.

## What does not work yet

The `-main` function runs but there is no way to call individual Clojure functions from the request handler yet. The GraalVM Web Image API has annotations for exporting functions to JavaScript — that is the next step to explore.
