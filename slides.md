---
theme: default
titleTemplate: '%s'
favicon: favicon.png
class: text-center
background: reinventing-rollup.jpeg
title: The Evolution of Rollup
aspectRatio: "16/10"
---

# The Evolution of Rollup

Dr. Lukas Taegert-Atkinson<br>
TNG Technology Consulting

Maintainer of RollupJS

---
layout: image
background: past-endeavours.jpeg
---

# State of the Rust Migration

Improvements since ViteConf 23

---
layout: small-image-right
image: past-endeavours.jpeg
---

## What did we achieve so far?

<v-click>
<div class="statement">20% faster<br>bundling speed</div>
<div>without plugins, as compared to v3.29.4</div>
</v-click>

<div v-click class="statement">5% reduced memory consumption</div>
<v-click>

<div style="margin-top: 3rem">Biggest improvements are yet to come.</div>

</v-click>

---
layout: small-image-right
image: past-endeavours.jpeg
---

## Next steps

<v-clicks class="click-fade">

* Move cache to binary format
  * Faster warm start if used
* New syntax tree parsing and -walking API for plugins
  * Parsing happens in a different thread
  * AST nodes are generated lazily
* Migrate tree-shaking
  * will take long
  * impact will be massive

</v-clicks>

---
layout: image
background: past-endeavours.jpeg
---

# Improved Tree-Shaking

---
layout: small-image-right
image: past-endeavours.jpeg
---

## Function argument tracking

<div class="sub-title">for functions only called once</div>


<v-click>
<div>Input</div>

```javascript
function query(target, config) {
  if (config.log) console.log('Hello')
  return fetch(target);
}
query('/api', {log: false});
```

</v-click>
<v-click at="2">

<div style="margin-top: 1rem">Output (Rollup <span v-click.hide="3" style="position:absolute">&lt;</span><span v-click="3">&ge;</span> v4.16.0)</div>

````md magic-move{at:3}
```javascript
function query(target, config) {
  if (config.log) console.log('Hello')
  return fetch(target);
}
query('/api', {log: false});
```
```javascript
function query(target, config) {
  return fetch(target);
}
query('/api');
```
````

</v-click>

---
layout: small-image-right
image: past-endeavours.jpeg
---

## Function argument tracking

<div class="sub-title">for functions called multiple times</div>

<div>Input</div>

````md magic-move{at:2}
```javascript
function query(target, config) {
  if (config.log) console.log('Hello')
  return fetch(target);
}
query('/api', {log: false});
query('/api', {log: false});
```
```javascript
function query(target, config) {
  if (config.log) console.log('Hello')
  return fetch(target);
}
const config = {log: false};
query('/api', config);
query('/api', config);
```
```javascript
function query(target, config) {
  if (config) console.log('Hello')
  return fetch(target);
}
query('/api', false);
query('/api', false);
```
````

<div style="margin-top: 1rem;">
<span v-click.hide="2" style="position: absolute"><v-click at="1">With inline object values: Not optimized</v-click></span>
<span v-click.hide="3" style="position: absolute"><v-click at="2">With the same variable: Optimized</v-click></span>
<span v-click="3">With the same primitive value: Optimized</span>
</div>
<v-click>

````md magic-move{at:2}
```javascript
function query(target, config) {
  if (config.log) console.log('Hello')
  return fetch(target);
}
query('/api', {log: false});
query('/api', {log: false});
```
```javascript
function query(target, config) {
  return fetch(target);
}
query('/api');
query('/api');
```
```javascript
function query(target, config) {
  return fetch(target);
}
query('/api');
query('/api');
```
````

</v-click>

---
layout: small-image-right
image: past-endeavours.jpeg
---

## Tree-shaking for dynamic imports

<div class="sub-title">Property access tracking (in preparation)</div>

<div v-click>
<div>Input</div>

```javascript{all|3,6-7|1-6}{at:2}
// main.js
const module = await import('./dep.js');
console.log(module.foo);

// dep.js
export const foo = 'foo';
export const bar = 'bar';
```

</div>
<v-click at="3">

With property access tracking,<br>`bar` will not be included in the bundle.

</v-click>
<v-click at="4">

But why stop here?

</v-click>

---
layout: small-image-right
image: past-endeavours.jpeg
---

## Tree-shaking for object properties

<div class="sub-title">(in preparation)</div>

<div>Input</div>

```javascript{all|2,4}{at:2}
function checkOptions(options) {
  console.log(options.enabled);
}
const config = {enabled: true, other: 'ignored'};
checkOptions(config);
```

<v-click>
<div style="margin-top: 1rem">Output with object property tree-shaking</div>

```javascript{hide|all|2,4}{at:1}
function checkOptions(options) {
  console.log(options.enabled);
}
const config = {enabled: true};
checkOptions(config);
```

</v-click>

---
layout: image
background: past-endeavours.jpeg
---

# New Supported Syntax

---
layout: small-image-right
image: past-endeavours.jpeg
---

<div style="margin-bottom: 1rem">Now supported (no transpilation):</div>
<v-click>

## Explicit resource management

<div style="margin: -1rem 0 2rem">

```javascript{all|4,9}{at:2}
function getResource() {
  /* allocate resourc here */;
  return {
    [Symbol.dispose]() { /* free resource here */ }
  };
}

// disposed when out of scope
using resource = getResource();
```

</div>
</v-click>

---
layout: small-image-right
image: past-endeavours.jpeg
---

<div style="margin-bottom: 1rem">Now supported (no transpilation):</div>

## Decorators

<div style="margin: -1rem 0 2rem">

```javascript{all|1,3,7,8,11}
function logAsync(target, name, descriptor) {
  const method = descriptor.value;
  descriptor.value = async function (...args) {
    const result = await method.apply(this, args);
    console.log('method complete');
    return result;
  }
}

class Foo {
  @logAsync
  async run() {}
}

new Foo().run();
```

</div>
<v-click>
But there is something else...
</v-click>

---
layout: small-image-right
image: past-endeavours.jpeg
---

## Experimental JSX Support
<div class="sub-title">(being finalized)</div>

<div v-click>
<div>Input</div>

````md magic-move{at:3}
```javascript
import React from 'react';
export const Foo = () => <div key="foo">Hello!</div>;
```
```javascript
export const element = <div key="foo">Hello!</div>;
```
````

</div>
<div style="margin-top: 1rem" v-click="2">
  <span style="position: absolute" v-click.hide="3"><code>jsx.mode: "classic"</code></span>
  <span style="position: absolute" v-click.hide="4"><code v-click="3">jsx.mode: "automatic"</code></span>
  <span><code v-click="4">jsx.mode: "preserve"</code></span>

````md magic-move{at:3}
```javascript
import react from 'react';
const Foo = () =>
  /*#__PURE__*/react.createElement(div, { key: "foo" }, "Hello!");
export { Foo };
```
```javascript
import { jsx } from 'react/jsx-runtime';
const element =
  /*#__PURE__*/jsx("div", { children: "Hello!" }, "foo");
export { element };
```
```javascript
export const element = <div key="foo">Hello!</div>;
```
````

</div>
<div v-click="4">with full tree-shaking and deconflicting support.</div>

<h3 v-click="5" style="margin-top: 2rem">Why add support for non-JavaScript syntax?</h3>

<v-clicks at="6" class="click-fade">

* Important use case for libraries
* No longer possible to extend syntax via plugins
* Already supported by parser
* Big performance improvement over plugins

</v-clicks>

<h3 v-click="10" style="margin-top:2rem">☞ We are already looking at TypeScript support</h3>

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

slides: <a href="https://lukastaegert.github.io/viteconf24-rollup">lukastaegert.github.io/viteconf24-rollup</a><br>

</div>
