---
title: "Clojure on CF Workers via WASM, Part 1: Compiling"
description: "Can real Clojure (JVM) code run on Cloudflare Workers? Using GraalVM Web Image to compile Clojure to WASM — and it works."
publishDate: 2026-06-16
tags: ["clojure", "wasm", "cloudflare-workers"]
---

This is part 1 of an exploration into running real Clojure on Cloudflare Workers. Not ClojureScript. Not Squint. Actual Clojure, compiled to WebAssembly.

## The question

Cloudflare Workers runs V8 isolates: JavaScript and WASM only. No JVM. ClojureScript compiles Clojure syntax to JavaScript and works fine. But I wanted to know: can you take a real Clojure program — compiled JVM bytecode — and run it on CF Workers?

The answer turned out to be yes.

## The path

GraalVM 25 ships an experimental feature called **Web Image** (`--tool:svm-wasm`). It takes a JVM application and compiles it to WebAssembly using the same AOT pipeline as GraalVM native-image, but targeting WASM instead of a native binary.

This is different from:
- ClojureScript / Squint — compile Clojure syntax to JavaScript
- Babashka — native binary, no WASM target
- Chicory — a JVM *in* WASM, not the other way around

## Toolchain

- Oracle GraalVM 25.0.3 (via Homebrew: `brew install graalvm-jdk@25`)
- Binaryen v130 (`brew install binaryen`)
- Clojure CLI tools

## The program

A minimal Clojure namespace:

```clojure
(ns clj-wasm-worker.core
  (:gen-class))

(defn greet [name]
  (str "Hello from Clojure WASM, " name "!"))

(defn -main [& _args]
  (println (greet "world")))
```

Build it into an uberjar with AOT compilation:

```clojure
;; deps.edn
{:paths ["src"]
 :deps {org.clojure/clojure {:mvn/version "1.12.0"}}
 :aliases
 {:uberjar
  {:deps {com.github.seancorfield/depstar {:mvn/version "2.1.303"}}
   :exec-fn hf.depstar/uberjar
   :exec-args {:jar "target/app.jar"
               :main-class clj-wasm-worker.core
               :aot true}}}}
```

```bash
clojure -X:uberjar
```

## Compiling to WASM

```bash
export GRAALVM_HOME=/Library/Java/JavaVirtualMachines/graalvm-25.jdk/Contents/Home
export PATH=$GRAALVM_HOME/bin:$PATH

native-image --tool:svm-wasm \
  -jar target/app.jar \
  -H:Name=target/wasm/app \
  --no-fallback \
  --initialize-at-build-time
```

The `--initialize-at-build-time` flag is critical. Without it, Clojure tries to dynamically load `clojure.core` from the classpath at runtime — which does not exist in a WASM sandbox. The flag bakes the entire Clojure runtime into the image at compile time.

Output:

```
Build artifacts:
  target/wasm/app.js       (92KB  — JS glue/runtime)
  target/wasm/app.js.wasm  (5.3MB — the WASM binary)
  target/wasm/app.js.wat   (116MB — human-readable text, not deployed)
```

Gzipped WASM: **2.44MB**. CF Workers free tier limit is 3MB compressed. We are inside it.

## Verify locally

```bash
node target/wasm/app.js
# Hello from Clojure WASM, world!
```

It runs. Real Clojure executing inside a WASM runtime in Node.js.

## What's next

Getting this to actually run *in Cloudflare Workers* required two more patches to the GraalVM JS glue — the auto-run at module load, and the WASM loading mechanism. That is Part 2.
