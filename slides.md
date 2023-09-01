---
theme: default
titleTemplate: '%s'
favicon: favicon.png
class: text-center
background: reinventing-rollup.jpeg
title: Reinventing Rollup
aspectRatio: "16/10"
---

# Reinventing Rollup

Dr. Lukas Taegert-Atkinson<br>
TNG Technology Consulting

Maintainer of RollupJS

---
layout: image
background: past-endeavours.jpeg
---

# Past endeavors

Important milestones since ViteConf 22

---
layout: small-image-right
image: past-endeavours.jpeg
---

## Features for end users

> Try examples in StackBlitz (bottom of window).

<v-clicks class="click-fade">

* Much faster code-splitting algorithm
* Named export tree-shaking for dynamic imports
* `__NO_SIDE_EFFECTS__` annotation for function declarations
* `experimentalMinChunkSize` option
  * Works, but results not 100% satisfactory
  * Heavily relies on side-effect-free code → motivated new features
* `experimentalLogSideEffects` to find out what Rollup considers as side effects
* `treeshake.manualPureFunctions` to eliminate side effects manually

</v-clicks>

---
layout: small-image-right
image: past-endeavours.jpeg
---

## Features for plugin authors

> Try examples in StackBlitz (bottom of window).

<v-clicks class="click-fade">

* Tree-shaking for emitted assets:<br>
  `this.emitFile()` with `needsCodeReference: true`
* Support for `"prebuilt-chunk"` emission in `this.emitFile()`
* `resolvedBy` field in `this.resolve()` response for easier debugging
* New logging API
  - `this.debug()` (hidden by default) and `this.info()` (shown by default)
  - React to and filter logs via `onLog` hook

</v-clicks>

---
layout: small-image-right
image: past-endeavours.jpeg
---

<p>

→ Solid improvements, but not a quantum leap.

</p>

<v-click>

### Major complaints

<p>

* Bundling speed
* Memory consumption

</p>

</v-click>

---
layout: image
background: replacing-the-engine-while-driving.jpeg
---

# Replacing the engine while driving

Addressing performance issues<br>without compromising other features

---
layout: small-image-right
image: replacing-the-engine-while-driving-narrow.jpeg
---

## Profiling bundling speed

Rollup browser build as input, no plugins

<v-click class="highlight">

- build: 823ms
  * generate module graph: 447ms
    <ul>
    <li class="highlighted">generate syntax tree (acorn): 180ms</li>
    <li>analyze syntax tree (Rollup): 262ms</li>
    </ul>
  * bind modules: 31ms
  * tree-shaking: 345ms
- generate: 79ms

</v-click>

---
layout: small-image-right
image: replacing-the-engine-while-driving-narrow.jpeg
clicks: 6
---

## Replacing the parser


<style>
.slidev-layout h3 {
    margin-top: 2rem;
    margin-bottom: -1rem;
}
</style>

### Running SWC from JavaScript

Naïve approach: replace acorn with swc via `@swc/core`.

```js
import { parseSync } from '@swc/core';
const result = swc.parseSync(code);
```

<p v-click="1">
Time: 270ms <span v-click="2">(vs. 180ms for acorn!?)</span>
</p>

<div v-click="3">

### Running SWC from Rust

Time: 51ms <span v-click="4">→ JSON serialization and deserialization is very costly!</span>

</div>

<div v-click="4">

### More problems:

<p>

<v-clicks class="click-fade" at="4">

- Non-standard AST not easily usable by Rollup
- AST node positions are UTF-8 offsets vs. UTF-16 in JavaScript

</v-clicks>

</p>

</div>

---
layout: small-image-right
image: replacing-the-engine-while-driving-narrow.jpeg
---

## Can we do better?

<v-clicks class="click-fade">

- Use SWC's parser from Rust
- Convert AST to custom binary format in Rust
- While doing this
  - convert UTF-8 to UTF-16 offsets
  - make AST ESTree-compatible
- Directly consume ArrayBuffer in JavaScript

</v-clicks>

---
layout: small-image-right
image: replacing-the-engine-while-driving-narrow.jpeg
---

## Trying it out

Total parse time including conversion: 108ms (vs. 180ms for acorn)

<v-click at="0">

→ Not a quantum leap, but much better!

Breaking down the numbers:

<v-clicks class="click-fade" at="1">

- SWC parse: 51ms
- Convert to binary: 8ms
- Convert to AST in JavaScript: 47ms

</v-clicks>

</v-click>

---
layout: small-image-right
image: replacing-the-engine-while-driving-narrow.jpeg
clicks: 1
---

## We gained more

<v-clicks class="click-fade" at="-1">

- ArrayBuffers can be passed to WebWorkers without copying<br>
  → trivial to parallelize!
- ArrayBuffer is about 1/4 the size of the JSON representation<br>
  → efficient caching with fast deserialization

</v-clicks>

---
layout: small-image-right
image: replacing-the-engine-while-driving-narrow.jpeg
clicks: 4
---

## And this is the start

We can now grow the native part from there<br>(not part of initial release):

<v-clicks class="click-fade" at="-1">

- Avoid JSON deserialization and directly consume ArrayBuffer
- Move scope analysis to Rust
- Offer efficient and fast AST traversal and mutation API to plugins
- Directly consume/convert TypeScript or JSX
- Move tree-shaking to Rust...

</v-clicks>

---
layout: small-image-right
image: replacing-the-engine-while-driving-narrow.jpeg
clicks: 3
---

## What does it cost us?

<v-clicks class="click-fade" at="-1">

- About 2.5 MB additional native code<br>
  → we only include some parts of SWC
- We have binaries for the most common platforms and architectures
- Otherwise, use `@rollup/wasm-node`
- `@rollup/browser` also includes a `.wasm` file

</v-clicks>

---
layout: small-image-right
image: replacing-the-engine-while-driving-narrow.jpeg
---

## Try it out

<p>

```
npm install rollup@beta
```

</p>

unless this is stable by now 😉

---
layout: small-image-right
image: replacing-the-engine-while-driving-narrow.jpeg
---

## What about Rolldown?

> Fast JavaScript/TypeScript bundler in Rust with Rollup-compatible API

<v-click>

### Problem: The rewrite dilemma

> Unless well-staffed and fully committed,<br>a rewrite drains resources from the old project without ever replacing it.

</v-click>

<v-click>

<p style="text-align: center;">
<strong>I am not opposed to this project!</strong>
</p>

If there is a chance that Rolldown

- covers nearly 100% of Rollup
- with similar output quality
- in reasonable time

I will switch to that project and make it the next major Rollup version.

</v-click>

---
layout: image
background: future-aspirations.jpeg
---

# Future aspirations
Where could we go with Rollup beyond making it fast?

---
layout: small-image-right
image: future-aspirations.jpeg
clicks: 2
---

## Some pipe dreams

<v-clicks class="click-fade" at="-1">

- Statement level code-splitting<br>
  → Take apart large modules
- Object property tree-shaking<br>
  → Remove unused object properties
- Merge chunks and duplicate modules<br>
  for optimized initial load via custom runtime loader

</v-clicks>

---
layout: image
class: text-center
background: reinventing-rollup.jpeg
---

# Thank you

Dr. Lukas Taegert-Atkinson<br>
TNG Technology Consulting

<div>

<a href="https://m.webtoo.ls/@lukastaegert"><logos-mastodon-icon /> @lukastaegert@webtoo.ls</a>

<a href="https://rollupjs.org/"><logos-rollup/> rollupjs.org</a>

slides: <a href="https://lukastaegert.github.io/viteconf23-rollup">lukastaegert.github.io/viteconf23-rollup</a>

</div>
