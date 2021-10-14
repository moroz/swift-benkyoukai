## Integrating Vite.js with Phoenix 1.6
### 勉強会 (benky&#333;kai) by Karol Moroz
### October 2021


### But yo what's up with Webpack?

Phoenix 1.6 replaces Webpack with esbuild. This means that the Phoenix core team does not have to spend time maintaining Webpack configs, but Webpack is a powerful and battle-tested tool.

As we know, Phoenix does not depend on any specific build tool so we can use anything we want that compiles assets to `priv/static`.


### Why not ESBuild?

ESBuild is very low level. It is basically just a Go library for compiling JavaScript. You can configure it to compile all your JS and CSS without touching NPM or Yarn. Some people even built Hex packages for SASS and Bulma.

If you have ever used the Rails asset pipeline with `bootstrap-sass`, you know this is not the optimal solution.


### What is Vite.js?

Vite.js is a build tool made by the creator of Vue.

It uses ESBuild under the hood and provides some nice configs and supports Vue, React, Svelte, Lit, Preact, Vanilla JS and TypeScript out of the box.


### Getting started

Install Latest Phoenix:

```
mix archive.install hex phx_new
```

Create a new project:

```
mix phx.new vite_demo
cd vite_demo
mix do deps.get, ecto.setup
```


### What we get in the project (1)

The project comes with the package `esbuild` preinstalled:

```elixir
# mix.exs
{:esbuild, "~> 0.2", runtime: Mix.env() == :dev},
```


### What we get in the project (2)

In `config/dev.exs`, there is a watcher set for `esbuild`:

```elixir
config :vite_demo, ViteDemoWeb.Endpoint,
  http: [ip: {127, 0, 0, 1}, port: 4000],
  check_origin: false,
  code_reloader: true,
  debug_errors: true,
  watchers: [
    esbuild: {
      Esbuild, :install_and_run,
      [:default, ~w(--sourcemap=inline --watch)]
    }
  ]
```


### Creating a config for Vite.js

Generate a Vite.js project using `yarn create`:

```
yarn create vite --template react-ts vite-project
```


### Copy files over to Phoenix

Copy the whole Vite project to the `assets` directory of your phoenix application, and remove `index.html`.


### Uninstall esbuild

Remove the `esbuild` dependency in `mix.exs` and unlock it:

```
mix deps.unlock esbuild
```

Remove the block starting with `config :esbuild` inside `config/config.exs`.


### Replace watcher

In `config/dev.exs`, replace the code that launches esbuild with a call to the new Vite watcher:

```elixir
config :vite_demo, ViteDemoWeb.Endpoint,
  http: [ip: {127, 0, 0, 1}, port: 4000],
  check_origin: false,
  code_reloader: true,
  debug_errors: true,
  # ...
  watchers: [
    node: [
      "node_modules/vite/bin/vite.js",
      cd: Path.expand("../assets", __DIR__)
    ]
  ]
```


### Install SASS and Phoenix JS deps

```
cd assets
yarn add -D sass @types/{phoenix,node}
yarn add bulma phoenix phoenix_html phoenix_live_view
```

We install all the usual libraries that Phoenix uses by default, as well as SASS and Bulma.


### Configure Vite (1)

Instruct Vite to terminate at the same time as Phoenix:

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig(({ command }: any) => {
  if (command !== "build") {
    // Terminate the watcher when Phoenix quits
    process.stdin.on("close", () => {
      process.exit(0);
    });
    process.stdin.resume();
  }
  // ...
```


### Configure Vite (2)

```typescript
  return {
    publicDir: "static",
    plugins: [react()],
    build: {
      target: "esnext",
      outDir: "../priv/static", // build assets to priv/static
      emptyOutDir: true,
      sourcemap: true,
      rollupOptions: {
        input: {
          main: "./js/app.js"
        }
      }
    },
  }});
```


### Import Vite assets in layout (1)

Add a function to your `LayoutView` to tell you whether you are running in `dev`:

```elixir
@env Mix.env() # remember value at compile time
def dev_env?, do: @env == :dev
```


### Import Vite assets in layout (2)

Add a partial called `_preamble.html.heex` in the directory for layout templates:

```eex
<%= if dev_env?() do %>
<script type="module">
  import RefreshRuntime from
    "http://localhost:3000/@react-refresh";
  RefreshRuntime.injectIntoGlobalHook(window);
  window.$RefreshReg$ = () => {};
  window.$RefreshSig$ = () => (type) => type;
  window.__vite_plugin_react_preamble_installed__ = true;
</script>
<script type="module" src="http://localhost:3000/@vite/client">
</script>
<script type="module" src="http://localhost:3000/src/main.tsx">
</script>
<% else %>
<link phx-track-static rel="stylesheet"
  href={Routes.static_path(@conn, "/assets/main.css")}/>
<script defer phx-track-static type="text/javascript"
  src={Routes.static_path(@conn, "/assets/main.js")}></script>
<% end %>
```


### Serve raw assets with Phoenix

In your `endpoint.ex`, add a `Plug.Static` to serve raw files from `assets`:

```elixir
if Mix.env() == :dev do
  plug Plug.Static,
    at: "/",
    from: "assets",
    gzip: false
end
```


### Replace default content

Remove the header in layouts and swap the contents of `page/index.html.heex` with a container for our React application:

```html
<div id="root"></div>
```


### Create a stylesheet hierarchy (1)

Under `assets/src`, create a folder called `sass` with three files: `app.sass`, `_variables.sass`, and `bulma.sass`.


### `_variables.sass`

```scss
@import url('https://fonts.googleapis.com/css?family=Nunito:400,700')

$purple: #8A4D76
$pink: #FA7C91
$brown: #757763
$beige-light: #D0D1CD
$beige-lighter: #EFF0EB

$family-sans-serif: "Nunito", sans-serif
$grey-dark: $brown
$grey-light: $beige-light
$primary: $purple
$link: $pink
$widescreen-enabled: false
$fullhd-enabled: false

$body-background-color: $beige-lighter
$control-border-width: 2px
$input-border-color: transparent
$input-shadow: none
```


### `bulma.sass`

```scss
@charset "utf-8"
/*! bulma.io v0.9.3 | MIT License | github.com/jgthms/bulma */

@import "../../node_modules/bulma/sass/utilities/_all.sass"
@import "../../node_modules/bulma/sass/base/_all.sass"
@import "../../node_modules/bulma/sass/elements/button.sass"
@import "../../node_modules/bulma/sass/elements/container.sass"
@import "../../node_modules/bulma/sass/elements/title.sass"
@import "../../node_modules/bulma/sass/form/_all.sass"
@import "../../node_modules/bulma/sass/components/navbar.sass"
@import "../../node_modules/bulma/sass/layout/hero.sass"
@import "../../node_modules/bulma/sass/layout/section.sass"
```


### `app.sass`

```scss
@import variables

@import bulma
```


### Import the SASS files

Inside `main.tsx`, import the SASS entry file:

```typescript
import React from "react";
import ReactDOM from "react-dom";
import App from "./App";
import "./sass/app.sass";

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById("root")
);
```


### Replace content inside React

Inside `App.tsx`, replace the default content with a simple page:

```typescript
import { useState, useCallback } from "react";
import logo from "./logo.svg";

function App() {
  const [count, setCount] = useState(0);
  const increment = useCallback(() => {
    setCount((count) => count + 1);
  }, [setCount]);

  return (
    <section className="section">
      <div className="container">
        <img src={logo} alt="React logo" width={120} />
        <h1 className="title">Hello World</h1>
        <p className="subtitle">
          A React app running on top of
          <strong>Phoenix</strong> and with
          support for <strong>Bulma</strong>
          and <strong>SASS</strong>!
        </p>
        <button className="button is-primary" onClick={increment}>
          Click me: {count}
        </button>
      </div>
    </section>
  );
}

export default App;
```
