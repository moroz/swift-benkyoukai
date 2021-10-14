## Integrating Vite.js with Phoenix 1.6
### 勉強会 (benky&#333;kai) by Karol Moroz
### October 2021


### What happened to Webpack?

Phoenix 1.6 replaces Webpack with esbuild. This means that the Phoenix core team does not have to spend time maintaining Webpack configs, but Webpack is a powerful and battle-tested tool.

As we know, Phoenix does not depend on any specific build tool so we can use anything we want that compiles assets to `priv/static`.


### Why not ESBuild?

ESBuild is very low level. It is basically just a Go library for compiling JavaScript. You can configure it to compile all your JS and CSS without touching NPM or Yarn. Some people even built Hex packages for SASS and Bulma.

If you have ever used the Rails asset pipeline with `bootstrap-sass`, you know how this ends...


### What is Vite.js?

Vite.js is a build tool made by the creator of Vue.

It uses ESBuild under the hood and provides some nice configs OOTB, and supports Vue, React, Svelte, and TypeScript.


### Getting started

Install Latest Phoenix:

```
mix archive.install hex phx_new
```

Create a new project:

```
mix phx.new vite_demo
```
