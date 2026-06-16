---
title: "Clojure on CF Workers via WASM, Part 1: Compiling"
description: "Can real Clojure (JVM) code run on Cloudflare Workers? Using GraalVM Web Image to compile Clojure to WASM, and it works."
publishDate: 2026-06-16
tags: ["clojure", "wasm", "cloudflare-workers"]
---

I wanted to know if real Clojure can run on Cloudflare Workers. Not ClojureScript. Not Squint. Actual JVM Clojure, compiled to WebAssembly.

The short answer is yes.

## What is the path

Cloudflare Workers runs V8 isolates. Only JavaScript and WASM are allowed. No JVM.

GraalVM 25 has an experimental feature called Web Image. The flag is `--tool:svm-wasm`. It compiles a JVM application to WebAssembly using the same AOT pipeline as native-image, but targeting WASM instead of a native binary.

This is different from:
- ClojureScript / Squint - these compile Clojure syntax to JavaScript
- Babashka - native binary, no WASM target
- Chicory - a JVM running inside WASM, not the other way

## Did anyone do this before

I searched and did not find a working example of JVM Clojure on CF Workers. The closest thing I found:

- [graal-clojure-wasm](https://github.com/roman01la/graal-clojure-wasm) by roman01la - shows Clojure to WASM compilation working in Node.js. No CF Workers.
- ClojureScript on CF Workers has examples, but that compiles to JavaScript, not WASM.
- JVM languages on CF Workers in general seem unexplored. Kotlin and Scala users have asked about it but no working examples.

If you have seen this done somewhere, I would like to know.

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

## Compile to WASM

```bash
export GRAALVM_HOME=/Library/Java/JavaVirtualMachines/graalvm-25.jdk/Contents/Home
export PATH=$GRAALVM_HOME/bin:$PATH

native-image --tool:svm-wasm \
  -jar target/app.jar \
  -H:Name=target/wasm/app \
  --no-fallback \
  --initialize-at-build-time
```

The `--initialize-at-build-time` flag is important. Without it, Clojure tries to load `clojure.core` from the classpath at runtime. There is no classpath in a WASM sandbox. The flag bakes the Clojure runtime into the image at compile time.

Output:

```
Build artifacts:
  target/wasm/app.js       (92KB  - JS glue)
  target/wasm/app.js.wasm  (5.3MB - the WASM binary)
  target/wasm/app.js.wat   (116MB - text format, not deployed)
```

Gzipped WASM is 2.44MB. CF Workers free tier limit is 3MB. We are inside it, for now.

## Run locally

```bash
node target/wasm/app.js
# Hello from Clojure WASM, world!
```

Real Clojure running in WASM. Now we need to get it into CF Workers. That is Part 2.
