Webpack, unpacked

## Contents

Foundations

1. [Why bundlers exist](#why)
2. [What webpack is](#what)
3. [The dependency graph](#graph)
4. [The core ideas](#core)
5. [Your first build](#setup)

Building blocks

6. [Loaders](#loaders)
7. [Asset modules](#assets)
8. [Plugins](#plugins)
9. [Mode](#mode)
10. [Dev server & HMR](#devserver)
11. [Source maps](#sourcemaps)
12. [Resolve & aliases](#resolve)
13. [Environments & configs](#env)

Optimizing

1. [Code splitting](#splitting)
2. [Tree shaking](#treeshaking)
3. [Caching & hashes](#caching)
4. [Minification](#minify)
5. [Faster builds](#speed)

Going deeper

1. [Module Federation](#federation)
2. [How webpack works inside](#internals)
3. [Write a loader & plugin](#custom)

Webpack vs Vite

1. [How Vite thinks](#vite)
2. [Side-by-side](#compare)
3. [Which one to pick](#choose)

Wrap-up

1. [Glossary](#glossary)
2. [Learning path](#path)

# Webpack,unpacked.

Plain-language notes on how webpack turns hundreds of files into a few fast ones, with an everyday analogy for every idea, and an honest comparison with Vite at the end.

Webpack starts at one file, follows every import, and packs everything it finds into bundles.

01

## Why bundlers exist

Modern apps are written as many small files: components, styles, images, helper functions, and hundreds of packages from `node_modules`. That’s great for humans, but browsers have problems with it:

- **Too many requests.** Loading 800 separate files is slow, even on fast networks.
- **Formats browsers don’t understand.** TypeScript, JSX, Sass, and Vue files must be converted first.
- **Packages are written for Node.** `import React from "react"` means nothing to a browser on its own; something has to find that package on disk.
- **Wasted bytes.** Comments, long variable names, and unused code all get downloaded unless something removes them.

A **bundler** solves all four. It reads your project, converts what needs converting, and outputs a small number of optimized files a browser can load quickly.

Think of it like moving house

You could carry every fork, sock, and book to the new place one at a time, but it would take forever. Instead you pack them into labelled boxes, throw out what you don’t need, and load a truck. A bundler is the packing crew for your code.

02

## What webpack is

**Webpack** is a *static module bundler* for JavaScript apps. “Static” means it does its work ahead of time, before anyone opens your site. “Module” means any file your code imports: JavaScript, CSS, images, fonts, JSON.

Released in 2012, it became the default build tool behind Create React App, older versions of Angular CLI and Vue CLI, Next.js (historically), and countless company codebases. Its long-running major version is **webpack 5**. Its superpower is flexibility: almost every step of the build can be customized.

Webpack itself only understands JavaScript and JSON. Everything else is taught to it with **loaders** and **plugins**, which you’ll meet shortly.

Think of it like a factory with an assembly line

Raw materials (your source files) come in one end. Stations along the line (loaders) reshape each part. Supervisors (plugins) watch the whole line and can change how it runs. Finished, boxed products (bundles) leave the other end.

03

## The dependency graph

This is the single most important idea in webpack. Starting from one file, webpack reads every `import` or `require`, opens those files, reads *their* imports, and keeps going until nothing new is found. The result is a map of which file needs which: the **dependency graph**.

Only files on this map end up in your bundle. A file that nothing imports is ignored, even if it sits in your `src` folder.

src/index.js

```
import { render } from "./app.js";     // webpack follows this…
import "./style.css";                // …and this
import logo from "./logo.svg";       // …and this

render(logo);
```

Think of it like a recipe that calls other recipes

Your lasagne recipe says “make the tomato sauce (see page 12)”. Page 12 says “use the stock from page 30”. To cook dinner you follow every reference until you have everything. Recipes nobody points to stay in the book, unused.

04

## The core ideas

Every webpack config is built from the same handful of concepts. Learn these and the rest is detail.

#### Entry

The file where webpack starts building the graph. Default: `./src/index.js`.

#### Output

Where finished bundles are written and what they’re called. Default: `./dist/main.js`.

#### Loaders

Convert one file at a time into something webpack understands (TS to JS, Sass to CSS).

#### Plugins

Hook into the whole build to do bigger jobs: generate HTML, extract CSS, analyze size.

#### Mode

`development` for fast, readable builds; `production` for small, optimized ones.

#### Module · Chunk · Bundle

A module is one source file. A chunk is a group of modules. A bundle is the final file a chunk becomes.

Think of it like a postal sorting center

**Entry** is the intake door. **Loaders** are workers who repackage odd-shaped items so they fit the conveyor. **Plugins** are managers who can reroute, relabel, or inspect any parcel at any stage. **Output** is the loading dock. **Chunks** are crates of related parcels, and **bundles** are those crates once they’re sealed and on the truck.

05

## Your first build

1. Create a project and install webpack plus its command-line tool: `npm init -y` then `npm install -D webpack webpack-cli`.
2. Add `src/index.js` with some code.
3. Create `webpack.config.js` in the project root (below).
4. Add a script to `package.json`: `"build": "webpack"`, then run `npm run build`.
5. Open `dist/` to see the bundle webpack produced.

webpack.config.js

```
const path = require("path");

module.exports = {
  mode: "development",
  entry: "./src/index.js",          // where to start
  output: {
    path: path.resolve(__dirname, "dist"), // must be an absolute path
    filename: "bundle.js",
    clean: true,                     // empty dist/ before each build
  },
};
```

**Zero-config works too.** If you skip the config file, webpack uses `./src/index.js` as entry and `./dist/main.js` as output. The config file exists so you can change those defaults and add loaders and plugins.

You can also have **multiple entries**, which produce multiple bundles. `[name]` in the filename is replaced with each entry’s key.

```
entry: { app: "./src/app.js", admin: "./src/admin.js" },
output: { filename: "[name].bundle.js" }  // app.bundle.js, admin.bundle.js
```

06

## Loaders

Out of the box, webpack only reads JavaScript and JSON. A **loader** is a function that takes a file’s contents and returns transformed contents. You tell webpack which loaders to use for which files with `module.rules`: a `test` (a pattern matching file names) and a `use` (the loaders to apply).

Common loaders: `babel-loader` (modern JS/JSX to widely supported JS), `ts-loader` (TypeScript), `css-loader` (understands `@import` and `url()` in CSS), `style-loader` (injects CSS into the page), `sass-loader` (Sass to CSS), `postcss-loader` (autoprefixing and more).

webpack.config.js

```
module: {
  rules: [
    { test: /\.jsx?$/, exclude: /node_modules/, use: "babel-loader" },
    { test: /\.tsx?$/, use: "ts-loader" },
    {
      test: /\.s[ac]ss$/,
      use: ["style-loader", "css-loader", "sass-loader"], // runs right → left
    },
  ],
},
```

**Order matters, and it runs backwards.** Loaders in a `use` array run from last to first. For the Sass rule above:

styles.scss*→*sass-loader*→*css-loader*→*style-loader*→*\<style> in page

Think of it like a chain of translators

A letter arrives in Portuguese. The first translator turns it into Spanish, the next turns Spanish into English, and the last reads it aloud to the room. Each one only knows its own step, and the order has to be right or the message turns to nonsense.

07

## Asset modules

Images, fonts, and other files used to need extra loaders (`file-loader`, `url-loader`, `raw-loader`). Webpack 5 has this built in as **asset modules**, chosen with `type`:

- `asset/resource`: copies the file to `dist` and gives you its URL. Best for large images and fonts.
- `asset/inline`: embeds the file inside your JS as a data URL. No extra request, but bigger JS.
- `asset/source`: gives you the file’s raw text (for `.txt` or SVG markup).
- `asset`: lets webpack decide. Files under 8 KB are inlined, larger ones become separate files.

```
{ test: /\.(png|jpe?g|gif|svg)$/i, type: "asset" },
{ test: /\.(woff2?|ttf)$/i,         type: "asset/resource" },
```

Think of it like packing a suitcase

Small things like a phone charger go inside the main bag (inline). Big things like a surfboard travel separately with a tag that says where to collect them (resource). With `asset`, webpack weighs each item and decides for you.

08

## Plugins

Loaders work on *one file at a time*. **Plugins** work on *the whole build*: they can add files, change bundles, inject variables, or report statistics. You create them with `new` and list them in `plugins`.

Plugins you’ll use constantly:

- `HtmlWebpackPlugin`: generates `index.html` and adds `<script>` tags for your bundles automatically.
- `MiniCssExtractPlugin`: pulls CSS out of JS into real `.css` files for production (use its loader instead of `style-loader`).
- `DefinePlugin` (built in): replaces names like `process.env.API_URL` with real values at build time.
- `CopyWebpackPlugin`: copies static files (robots.txt, favicons) straight into `dist`.
- `webpack-bundle-analyzer`: shows an interactive map of what’s inside your bundles and how big each part is.

webpack.config.js

```
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const { DefinePlugin } = require("webpack");

module.exports = {
  // …
  module: {
    rules: [{ test: /\.css$/, use: [MiniCssExtractPlugin.loader, "css-loader"] }],
  },
  plugins: [
    new HtmlWebpackPlugin({ template: "./src/index.html" }),
    new MiniCssExtractPlugin({ filename: "[name].[contenthash].css" }),
    new DefinePlugin({ __API_URL__: JSON.stringify("https://api.example.com") }),
  ],
};
```

Think of it like smartphone apps

The phone (webpack) does the basics. Each app you install (a plugin) adds a whole new ability: a camera filter, a calendar, a step counter. Apps don’t change one photo; they can work across everything on the phone.

**Loader or plugin?** If the job is “turn this kind of file into that kind”, it’s a loader. If the job affects the build as a whole, it’s a plugin.

09

## Mode

`mode` switches on a bundle of sensible defaults in one line:

- `development`: fast rebuilds, readable output, helpful error messages, no minification.
- `production`: minification, tree shaking, scope hoisting, and `process.env.NODE_ENV` set to `"production"` so libraries like React drop their dev-only checks.
- `none`: no defaults at all, mostly for learning or special cases.

You can set it in the config or on the command line: `webpack --mode production`.

Think of it like rehearsal vs opening night

At rehearsal (development) actors wear street clothes, stop to fix mistakes, and the director shouts notes. On opening night (production) everything is polished, trimmed, and nothing unnecessary is on stage.

10

## Dev server & Hot Module Replacement

**webpack-dev-server** runs a local server that rebuilds when you save, keeping bundles in memory instead of writing to disk, so it’s fast. Install it with `npm install -D webpack-dev-server` and run `webpack serve`.

**Hot Module Replacement (HMR)** goes further: instead of reloading the whole page, it swaps only the module you changed while the app keeps running. Your form inputs, scroll position, and app state survive. HMR is on by default in recent versions of the dev server; frameworks plug in with tools like React Fast Refresh.

```
devServer: {
  port: 3000,
  open: true,                 // open the browser
  hot: true,                  // HMR
  historyApiFallback: true,   // serve index.html for client-side routes
  proxy: [{ context: ["/api"], target: "http://localhost:8080" }],
},
```

Think of it like changing a light bulb

A full reload is switching off power to the whole house, replacing the bulb, then resetting every clock. HMR is unscrewing just the one bulb with the lights still on. Everything else keeps running exactly as it was.

11

## Source maps

After bundling and minifying, an error might point to “line 1, column 48,213 of bundle.js”. Useless. A **source map** is a separate file that maps every position in the bundle back to your original file and line, so browser dev tools show your real code.

Control it with `devtool`. Good defaults:

- Development: `"eval-source-map"` (accurate, rebuilds fast) or `"eval-cheap-module-source-map"` (even faster, line-level only).
- Production: `"source-map"` (full, separate files you can upload to an error tracker) or `false` to skip them.

Think of it like a translated novel with page references

You’re reading the English edition, but each paragraph has a tiny note: “original, page 87, line 4”. When something seems off, you can jump straight to the source text to check.

12

## Resolve & aliases

`resolve` controls how webpack finds files when you import them. Two settings matter most:

- `extensions`: lets you write `import Button from "./Button"` without `.tsx`.
- `alias`: short names for long paths, so `"../../../components/Button"` becomes `"@/components/Button"`.

```
resolve: {
  extensions: [".tsx", ".ts", ".js"],
  alias: { "@": path.resolve(__dirname, "src") },
},
```

Think of it like phone contacts

You don’t dial a 12-digit number every time you call Mum; you tap “Mum”. An alias is a contact name for a folder you visit often.

13

## Environments & multiple configs

Real projects need different settings for development and production. The common pattern is three files: a shared `webpack.common.js`, plus `webpack.dev.js` and `webpack.prod.js` that merge in their own extras using the `webpack-merge` package.

A config can also be a **function** that receives the CLI’s `--env` values and `argv`, which is handy for small projects.

webpack.prod.js

```
const { merge } = require("webpack-merge");
const common = require("./webpack.common.js");

module.exports = merge(common, {
  mode: "production",
  devtool: "source-map",
});
```

**Environment variables:** webpack doesn’t read `.env` files by itself. Use `DefinePlugin`, `EnvironmentPlugin`, or the `dotenv-webpack` package. Anything you inject ends up in public JavaScript, so never put secrets there.

Think of it like a base recipe with variations

One pancake batter (common config). For weekdays you add nothing and cook fast (dev). For a brunch party you add berries and plate it nicely (prod). You don’t rewrite the batter twice.

14

## Code splitting & lazy loading

One giant bundle means users download your entire app, including the settings page they may never open, before seeing anything. **Code splitting** breaks the bundle into chunks that load only when needed. Three ways to do it:

- **Dynamic imports:** `import("./Chart.js")` returns a promise and tells webpack to put that module in its own chunk, fetched on demand.
- **SplitChunksPlugin:** `optimization.splitChunks` pulls shared code and `node_modules` into separate chunks automatically.
- **Multiple entries:** separate pages get separate bundles.

```
// Loaded only when the button is clicked
button.addEventListener("click", async () => {
  const { drawChart } = await import(/* webpackChunkName: "chart" */ "./chart.js");
  drawChart();
});

// In webpack.config.js — share vendor code across chunks
optimization: {
  splitChunks: { chunks: "all" },
},
```

In React, the same idea is `React.lazy(() => import("./Settings"))` wrapped in `<Suspense>`.

Think of it like a streaming service

Netflix doesn’t make you download the entire season before episode one plays. It streams what you’re watching now and fetches the next episode when you get there.

15

## Tree shaking

**Tree shaking** removes code you import from but never use. If a utility file exports 40 functions and you use 2, only those 2 end up in the production bundle.

It only works when:

- You use **ES module syntax** (`import`/`export`), not `require`. Webpack needs to see the structure without running the code.
- You build in `production` mode (unused exports are marked, then the minifier deletes them).
- Babel isn’t converting your modules to CommonJS (keep `modules: false` in `@babel/preset-env`, which is the default under webpack).
- Packages declare `"sideEffects": false` in `package.json` when importing a file does nothing but export things. If files do have side effects (like CSS imports), list them: `"sideEffects": ["*.css"]`.

```
import { debounce } from "lodash-es";  // ✓ only debounce is kept
import _ from "lodash";                 // ✗ the whole library comes along
```

Think of it like shaking an apple tree

Shake the tree and the dead, loose leaves fall off while everything still attached stays. Code that nothing is “attached” to (nothing imports it) falls out of the bundle.

16

## Caching & content hashes

Browsers cache files so repeat visits are fast. The problem: if you ship a new `bundle.js` with the same name, users may keep the old one. The fix is to put a **content hash** in the filename. The hash is a fingerprint of the file’s contents, so it changes only when the contents change.

```
output: {
  filename: "[name].[contenthash].js",  // main.7f3a9c1e.js
},
optimization: {
  runtimeChunk: "single",            // keep webpack's small runtime separate
  moduleIds: "deterministic",        // stable ids → stable hashes
  splitChunks: { chunks: "all" },     // vendor code rarely changes, so it stays cached
},
```

Now you can tell browsers to cache these files for a year. When you deploy, only files that actually changed get new names and are downloaded again. `HtmlWebpackPlugin` updates the script tags for you.

Think of it like dated labels on jam jars

If every jar just said “jam”, you’d never know which batch is fresh. Write the batch date on the lid and you instantly know which jars are new and which you can keep using.

17

## Minification

In production mode, webpack runs **Terser** to shrink JavaScript: it removes whitespace and comments, shortens variable names, and deletes dead code. For CSS, add `css-minimizer-webpack-plugin`.

Webpack also does **scope hoisting** (module concatenation) in production: instead of wrapping every module in its own function, it merges modules into fewer scopes, which makes code smaller and faster to run.

```
const TerserPlugin = require("terser-webpack-plugin");
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");

optimization: {
  minimize: true,
  minimizer: [new TerserPlugin(), new CssMinimizerPlugin()], // "..." keeps defaults too
},
```

Think of it like vacuum-sealing clothes

The same jumper, squeezed into a bag a third of the size. Nothing you need is lost; the air is just gone.

18

## Making builds faster

Big webpack projects can take minutes to build. The biggest wins, roughly in order:

- **Persistent cache:** `cache: { type: "filesystem" }` saves work to disk so the next build reuses it. Often the single largest speed-up.
- **Narrow your loaders:** add `include: path.resolve(__dirname, "src")` so Babel doesn’t process `node_modules`.
- **Faster transpilers:** swap `babel-loader` for `swc-loader` or `esbuild-loader`, which are written in Rust and Go.
- **Cheaper source maps** in development (see chapter 11).
- **Measure first:** `webpack --profile --json > stats.json` and a bundle analyzer show where time and bytes go.

If you need much more speed but want to keep your config, look at **Rspack**, a Rust-based bundler designed to be largely compatible with webpack’s config and plugin ecosystem.

Think of it like meal prep

Chopping all your vegetables on Sunday (the cache) means weeknight dinners take ten minutes instead of forty. You only chop what’s new.

19

## Module Federation

**Module Federation** (added in webpack 5) lets separately built and deployed apps share code *at runtime*. One app “exposes” modules; another app loads them over the network as if they were local. It’s the foundation of many **micro-frontend** setups, where different teams own different parts of one site.

checkout app — webpack.config.js

```
const { ModuleFederationPlugin } = require("webpack").container;

plugins: [
  new ModuleFederationPlugin({
    name: "checkout",
    filename: "remoteEntry.js",
    exposes: { "./Cart": "./src/Cart" },
    shared: { react: { singleton: true } },  // load React only once
  }),
],
// Another app lists remotes: { checkout: "checkout@https://…/remoteEntry.js" }
// and then: const Cart = React.lazy(() => import("checkout/Cart"));
```

Think of it like a food court

Each stall cooks independently, with its own staff and opening hours. But they share the seating, the trays, and the cash registers. Customers see one food court; behind the scenes, each stall can change its menu without asking the others.

20

## How webpack works inside

You don’t need this to use webpack, but it makes everything else click. When you run a build:

1. **Read the config** and merge it with defaults. A `Compiler` object is created to run the whole process.
2. **Start a Compilation.** Each build (or each rebuild in watch mode) is one `Compilation`.
3. **Resolve the entry** to a real file path on disk.
4. **Run loaders** on that file to get plain JavaScript.
5. **Parse** the result into a syntax tree and find every `import`. Each one becomes a new module to resolve, load, and parse. Repeat until the graph is complete.
6. **Seal:** group modules into chunks, then optimize (tree shaking, splitting, hashing, minifying).
7. **Emit:** turn chunks into bundle files, adding a small **runtime** that knows how to load modules and chunks in the browser, and write them to the output folder.

At every step, webpack fires **hooks** (built on a small library called *Tapable*). Plugins “tap” into these hooks to run their own code at exactly the right moment. In fact, much of webpack itself is written as internal plugins.

Think of it like a theatre production with cue points

The show runs in a fixed order: doors open, curtain up, act one, interval, act two, bows. Each moment is a cue. The lighting crew, sound crew, and stagehands (plugins) each wait for their cue and act on it, without needing to know how the rest of the show works.

21

## Write your own loader & plugin

Both are surprisingly small. A **loader** is just a function that receives a file’s contents as a string and returns new contents:

loaders/banner-loader.js

```
module.exports = function (source) {
  const { text } = this.getOptions();   // options from the config
  return `/* ${text} */\n` + source;
};

// webpack.config.js
{ test: /\.js$/, use: { loader: path.resolve("loaders/banner-loader.js"), options: { text: "Built with ♥" } } }
```

A **plugin** is a class with an `apply` method. Webpack calls it with the compiler, and you tap into whichever hook you need:

plugins/BuildTimePlugin.js

```
class BuildTimePlugin {
  apply(compiler) {
    compiler.hooks.done.tap("BuildTimePlugin", (stats) => {
      const ms = stats.endTime - stats.startTime;
      console.log(`Build finished in ${ms} ms`);
    });
  }
}
module.exports = BuildTimePlugin;

// webpack.config.js → plugins: [new BuildTimePlugin()]
```

Think of it like joining the assembly line

Writing a loader is adding one new workstation that reshapes a specific part. Writing a plugin is hiring a supervisor who listens for the factory bell (a hook) and does their job when it rings.

22

## How Vite thinks differently

**Vite** (French for “quick”, pronounced *veet*) is a newer build tool created by Evan You, the author of Vue. Its big insight: modern browsers already understand `import` natively (ES modules). So during development, why bundle at all?

**In development**, Vite starts a server almost instantly and serves your files *as they’re requested*. When the browser asks for `App.jsx`, Vite transforms just that one file and sends it back. Files for pages you haven’t opened are never touched. Packages in `node_modules` are pre-bundled once, because libraries can be made of hundreds of tiny files.

**For production**, Vite does bundle, because shipping thousands of unbundled files is still slow. Since **Vite 8** (March 2026), both the dev pre-bundling and the production build use **Rolldown**, a Rust-based bundler with a Rollup-compatible plugin API, replacing the older esbuild + Rollup pairing. Vite 8.1 also added an optional bundled dev mode for very large apps.

Other differences you’ll notice: `index.html` is the entry point (not a JS file), CSS/TypeScript/JSON work without any loaders, and environment variables come from `.env` files via `import.meta.env` (only those prefixed with `VITE_` are exposed).

#### Webpack dev server

Read every file in the graph

↓

Run loaders on all of them

↓

Build the whole bundle in memory

↓

Server ready, browser loads bundle

Start-up grows with app size. On a save, affected parts of the bundle are rebuilt.

#### Vite dev server

Pre-bundle dependencies once (cached)

↓

Server ready almost instantly

↓

Browser requests a file via native import

↓

Vite transforms only that file

Start-up stays fast as the app grows. On a save, only the changed module is re-sent.

Think of it like two restaurants

**Webpack’s kitchen** cooks every dish on the menu before opening the doors. The first customer waits while the whole menu is prepared, but then everything is ready.

**Vite’s kitchen** opens immediately and cooks each dish when it’s ordered. Popular staples (dependencies) are prepped in advance. Before a big catered event (production), both kitchens pack everything into boxes ahead of time.

23

## Webpack and Vite side by side

| Topic | Webpack | Vite |
| --- | --- | --- |
| Dev approach | Bundles the app before serving it | Serves source files on demand over native ES modules |
| Dev start-up | Slower as the project grows; persistent cache helps | Near-instant in most projects |
| HMR speed | Good, can slow in very large apps | Very fast and largely independent of app size |
| Production bundler | Webpack itself (Terser for minifying) | Rolldown since Vite 8 (Rollup before that) |
| Configuration | Explicit and detailed; you wire up loaders and plugins | Sensible defaults; small config file, often only a framework plugin |
| CSS, TS, JSON, assets | Need loaders or asset modules configured | Work out of the box |
| TypeScript | `ts-loader` can type-check during the build | Strips types only; run `tsc --noEmit` separately to check |
| Entry point | A JavaScript file | `index.html` |
| Env variables | `DefinePlugin` / `process.env` | `.env` files, `import.meta.env.VITE_*` |
| Plugin ecosystem | Huge, mature, a decade of edge cases covered | Large and growing; Rollup-style plugin API |
| Module Federation | Built in | Available through community plugins |
| Old browsers | Full control via Babel and polyfills | Targets modern browsers; `@vitejs/plugin-legacy` for older ones |
| Customization ceiling | Very high: hook into almost any build step | High, but fewer low-level knobs by design |

Think of it like a manual vs an automatic car

Webpack is a manual gearbox: more to learn, more control, and experts can squeeze out every bit of performance. Vite is a modern automatic: you get moving immediately and it makes good choices for you, though you have fewer levers to pull.

24

## Which one should you pick?

#### Reach for webpack when…

- You’re maintaining an existing webpack project (most companies have one).
- You need Module Federation or micro-frontends.
- You rely on specific loaders or plugins with no Vite equivalent.
- You need very fine-grained control over output.
- Want speed but must keep the config? Try Rspack.

#### Reach for Vite when…

- You’re starting a new project with React, Vue, Svelte, Solid, or plain JS.
- Fast feedback while coding is a priority.
- You prefer minimal configuration.
- You want testing to match your build setup (Vitest uses the same config).

**The honest take:** for most new projects today, Vite is the common default. But the concepts in this guide (graphs, loaders, plugins, chunks, tree shaking, hashing) apply to every bundler, Vite included. Learning webpack teaches you what all of them are doing underneath.

25

## Glossary

Bundle

A final output file containing many modules.

Chunk

A group of modules webpack plans to output together; becomes a bundle.

Module

Any imported file: JS, CSS, image, JSON.

Dependency graph

The map of which module imports which.

Entry

Where webpack starts building the graph.

Output

Where and how bundles are written.

Loader

Transforms one file type into another.

Plugin

Extends or changes the whole build via hooks.

HMR

Swapping changed modules in a running app without a full reload.

Source map

File linking bundled code back to the original source.

Tree shaking

Removing exports that nothing uses.

Code splitting

Breaking the app into chunks loaded on demand.

\[contenthash\]

Fingerprint in a filename that changes only when contents change.

Runtime

Small piece of webpack code in the browser that loads modules and chunks.

Scope hoisting

Merging modules into fewer function scopes for smaller, faster code.

ESM

ES modules, the standard `import`/`export` syntax browsers understand.

CommonJS

Node’s older `require`/`module.exports` system; harder to tree-shake.

Rolldown

Rust-based bundler powering Vite since version 8.

Rspack

Rust-based bundler that is largely webpack-compatible.

26

## A learning path

1. **Build from zero.** Empty folder, one JS file, no config. Run webpack and read the output in `dist`.
2. **Add a config** with entry, output, and `HtmlWebpackPlugin`.
3. **Add CSS and images** with `css-loader`, `style-loader`, and asset modules.
4. **Add Babel or TypeScript** and a dev server with HMR.
5. **Split dev and prod configs.** Add content hashes, CSS extraction, and source maps.
6. **Optimize:** lazy-load a route, run the bundle analyzer, and shrink the biggest chunk.
7. **Go deep:** write a tiny loader and a plugin from chapter 21.
8. **Rebuild the same app in Vite** and compare start-up time, config size, and output.

For reference, the official docs live at [webpack.js.org](https://webpack.js.org) and [vite.dev](https://vite.dev).

Notes current as of October 2026: webpack 5, Vite 8. Tooling moves quickly, so check the official docs for the newest options.