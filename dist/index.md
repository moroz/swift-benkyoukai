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
yarn add -D sass @types/{phoenix,postcss-url}
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
    base: BASE_URL,
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
  }
});
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
