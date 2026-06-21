# AppRun Docs — Review & Improvement Suggestions

_Review date: 2026-06-21 · Reviewed against AppRun `v6.0.0` (`../apprun`)_

This review covers the user-guide content under `docs/`, checked against the actual
AppRun source (`apprun.d.ts`, `src/router.ts`, `README.md`). Items are ordered by
impact. Each item lists the affected file(s) and a concrete fix.

---

## Executive summary

The docs are friendly and example-driven, and the interactive `<apprun-play>` snippets
are a real strength. The main problems are **drift from the v6 codebase** (especially
routing), **navigation gaps** (several important pages are unreachable), and
**inconsistent conventions** (routing prefixes and event syntax differ page to page).
There is also no consolidated **API reference**, and the prose carries a lot of typos.

Priorities, in order:

1. Fix/rewrite **routing** docs to match v6 (path routing, hierarchical routing, `basePath`).
2. Fix the **navigation** — add orphaned pages, reconsider section order.
3. Add a **canonical "which syntax" guide** and apply one convention consistently.
4. Add an **API reference** page.
5. Sweep typos and broken sample code.

---

## 1. Critical — content that is wrong or outdated

### 1.1 `routing.md` is outdated and contradicts the rest of the docs
`src/router.ts` shows v6 natively supports:
- path routing `/path`, hash `#path`, and `#/path`
- **hierarchical routing** (progressively tries parent routes)
- route patterns `:param` and `*` (added 2026-06-19)
- `app.basePath` for sub-directory deployments (`IApp.basePath` in `apprun.d.ts:145`)
- `app.addComponents(element, ComponentRoute)` for declarative route→component mapping

But `routing.md` only documents **hash** routing and tells users that for "pretty links"
they must "implement a new router yourself or use the third-party `apprun-router` package"
(`routing.md:42`). That advice is obsolete — path routing is built in. The
`new NoRouteComponent().mount( on some element )` snippet (`routing.md:37`) is pseudo-code
that won't run.

**Fix:** Rewrite `routing.md` to cover, with runnable examples: hash vs path vs `#/`,
hierarchical routing, `:param`/`*` patterns, `app.basePath`, `addComponents`, and
`ROUTER_EVENT`/`ROUTER_404_EVENT`. Remove the "write your own router" framing.

### 1.2 Routing convention is inconsistent across the guide
- `tutorial.md` SPA example uses `#home`, `#contact`, `#about` (`tutorial.md:85-95`)
- `index.md`/README use `/home`, `/contact`, `/about` (path routing)
- `spa.md` uses `'#,#home'` (`spa.md:71`)

A new reader can't tell which is the recommended approach. **Fix:** pick one canonical
style (path routing is the v6 default), use it everywhere, and add one short note
explaining hash vs path.

### 1.3 Bugged example in `reactivity.md`
`reactivity.md:42-45` — both buttons are labelled `+1`, and the "decrement" handler is
attached to the first button:
```js
<button $onclick={state => state - 1}>+1</button>   // label should be -1
<button $onclick={state => state + 1}>+1</button>
```
**Fix:** relabel the first button `-1`.

### 1.4 Analytics and copyright are stale
- `mkdocs.yml:46-47` uses a **Universal Analytics** `UA-123786786-1` property. UA stopped
  processing data in July 2023 — this collects nothing. Migrate to GA4 (`G-XXXX`) or remove.
- `mkdocs.yml:13` copyright reads `2015 - 2025`; today is 2026. (README is already
  `2015-2025` too — bump both.)

### 1.5 `README` counter uses a non-functional JSX handler
`../apprun/README.md:44-45` (mirrored into the guide's mental model) uses
`onclick="app.run('-1')"` — a **string** in JSX, whereas `index.md:18` uses
`onclick={()=>app.run('-1')}`. Mixing the two teaches readers an inconsistent (and in JSX,
fragile) pattern. **Fix:** standardize on the function form, or the `$onclick` directive.

---

## 2. High — navigation & structure

### 2.1 Orphaned pages (exist but not in `nav`)
These files are not referenced in `mkdocs.yml` `nav` and are only reachable by luck:
- `routing.md` — **core feature, completely missing from the menu**
- `dev-server.md`
- `ssr.md`
- `esm.md` (only linked inline from `tutorial.md`)

**Fix:** add `routing.md` under "Concepts" or "Advanced"; place `esm.md` under
"Getting started"; decide whether `ssr.md`/`dev-server.md` are still current (see 4.x) and
either fold them into AppRun-Site docs or list them explicitly.

### 2.2 Section order buries the on-ramp
`nav` puts **Concepts → Architecture** (deep theory, the 398-line `architecture.md` with
Ceremony-vs-Essence essays) *before* **Getting started → Installation/Tutorial**. New users
hit philosophy before they install anything.

**Fix:** Order as Home → Getting started → Concepts → Advanced → AppRun Site → Resources.

### 2.3 No API reference
The home page claims "only three functions: `app.start`, `app.run`, `app.on`"
(`index.md:38`), but `IApp` (`apprun.d.ts:125`) exposes many more that the guide uses or
should document: `once`, `off`, `run`, `runAsync`, `find`, `render`, `route`, `basePath`,
`addComponents`, `webComponent`, `use_render`, `use_react`, `version`, plus
`Component` methods (`mount`, `start`, `unmount`, `setState`, `run`, `runAsync`,
`add_action`) and exports `on`, `customElement`, `Fragment`, `trustedHTML`,
`ROUTER_EVENT`, `ROUTER_404_EVENT`.

**Fix:** add a single **API Reference** page enumerating these with signatures and a
one-line description. Keep the "three functions to get started" framing as a learning aid,
but link to the full reference.

### 2.4 New/undocumented v6 features
Present in `apprun.d.ts` but absent from the guide:
- `trustedHTML()` (and `safeHTML()` is now `@deprecated` — `apprun.d.ts:201-203`)
- `runAsync()` / `app.runAsync()`
- `customElement` decorator + `CustomElementOptions` (`apprun.d.ts:186, 113`)
- `app.start` `AppStartOptions` (`history`, `rendered`, `mounted`, route options, etc.)
- `setState` with `ActionOptions & EventOptions`

**Fix:** document these where relevant (security note for `trustedHTML`, options table for
`app.start`).

---

## 3. Medium — consistency & completeness

### 3.1 Pick one event syntax and teach the differences explicitly
Across pages, the same counter is written ~5 ways:
`onclick={()=>app.run('-1')}`, `onclick="app.run('-1')"`, `$onclick="-1"`,
`$onclick={fn}`, `$onclick={['add', 1]}`, `@click=${run('add',-1)}`, `@click=${run(fn,-1)}`.

Each is valid, but they're scattered without a map. **Fix:** add a short
**"Choosing your event syntax"** section (probably in `event-pubsub.md` or `directive.md`)
that contrasts: global `app.run` vs local `this.run`, named-event string vs function value
vs `[name, ...args]` tuple, and the JSX `$on` directive vs the lit-html `run` directive.

### 3.2 CDN/module paths are inconsistent
The guide references at least four entry points without explaining the difference:
`dist/apprun-html.js` (tutorial), `dist/apprun-html.esm.js` (esm), `dist/apprun.esm.js`
(esm htm example), `https://esm.run/apprun` (esm lit-html). **Fix:** add a short table:
which bundle exposes which globals (`app`/`html`/`svg`/`run`/`Component`), JSX vs html,
and when to use each.

### 3.3 `installation.md` is too thin (24 lines)
It covers `npm install` and `create-apprun-app` only. Missing: TypeScript/JSX setup
(`tsconfig` `jsxFactory`/`jsxImportSource`), the script-tag-vs-bundler decision, and where
the global `app`/`html`/`Component` come from in the script-tag build. **Fix:** expand, or
explicitly hand off to `esm.md`, `strong-typing.md`, and `create-apprun-app.md`.

### 3.4 `ssr.md` has no actual content
23 lines that only link to an external Glitch demo and a Medium post (`ssr.md`). The Glitch
host (`apprun-ssr.glitch.me`) is likely dead (Glitch sunset project hosting). Meanwhile
real SSR guidance lives under **AppRun Site → Server-Side Rendering**. **Fix:** either fold
`ssr.md` into the AppRun-Site SSR page or replace it with current, runnable content; verify
the external links resolve.

### 3.5 `dev-server.md` may describe a superseded tool
It documents `apprun-dev-server` (a forked live-server) and the "global module → unpkg"
rewriting trick from the Snowpack era, while `create-apprun-app` now scaffolds esbuild/
webpack/vite (`create-apprun-app.md:27-31`). **Fix:** confirm whether `apprun-dev-server`
is still the recommended dev workflow; if not, retire the page or mark it historical.

### 3.6 `component.md` lifecycle wording is muddled
- `component.md:161` says `mounted` "is only called in the child component," and the Parent
  example comment says it "will NOT be called when component is created using the
  constructor." The actual rule (mounted runs when the component receives props, i.e. when
  rendered via JSX with props vs. constructed directly) should be stated plainly.
- The `mounted` example body `(props, children) => { ...state, ...props }`
  (`component.md:168`) is missing the object-literal parens `({ ...state, ...props })` and
  references `state` that isn't a parameter there.

**Fix:** clarify the trigger condition and correct the snippet.

### 3.7 `spa.md` main example mixes element ids
`spa.md:50-54` starts `Layout` on `#main`, then grabs `#my-app` into `element` and
starts/mounts pages on it; the dynamic-loading section (`spa.md:71-84`) references a bare
`element` that was never defined in that block. **Fix:** use consistent, defined element
references; show a complete, runnable `main.tsx`. Also fix the `Layour.tsx` typo in the
file tree (`spa.md:22`).

---

## 4. Low — typos, grammar, broken markdown

A non-exhaustive list (there are many more):

| File | Issue |
|------|-------|
| `architecture.md:3` | "which **ont** only enhances" → "not only" |
| `architecture.md:85` | "a **bitter** complicated" → "a bit more complicated" |
| `architecture.md:178` | "_pure**__**function_" — stray double underscores break italics |
| `architecture.md:49-53` | View example uses `${state}` template syntax inside JSX (copy-paste from html version) |
| `tutorial.md:80` | "single-page **page** (SPA)" → "single-page app" |
| `reactivity.md:5-6` (frontmatter) | tags `#apprun #vue ...` render as a heading in some processors; also "Two-**binding**", "one-way **bing**" |
| `reactivity.md:33` | "create a virtual" — missing "DOM" |
| `dev-server.md:10` | "a **globe** module" → "global module" |
| `spa.md:22` | `Layour.tsx` → `Layout.tsx` |
| `state-management.md:3` | "**Modal** and State" → "Model and State" |
| `directive.md:3` | "AppRun **two** out-of-the-box directives" → "AppRun has two…" |
| `event-pubsub.md:7` | sentence ends mid-thought: "when the correspondent event" |
| `README` (`../apprun`):125,166 | links to `docs/requirements/...` (internal spec paths, not public docs); `use_react`/`use_render` shown without `app.` prefix |

**Fix:** run a spell/grammar pass; the double-underscore `_word__` patterns specifically
break MkDocs italics rendering and should be searched for globally.

---

## 5. Suggested concrete action plan

1. **Routing overhaul** (`routing.md` rewrite + add to nav + unify `#`/`/` convention across
   `tutorial.md`, `spa.md`, `index.md`). _Highest impact._
2. **Navigation fix** (`mkdocs.yml`): add `routing.md`, `esm.md`; reorder Getting-started
   ahead of Concepts; resolve `ssr.md`/`dev-server.md` status.
3. **API Reference page** generated from `apprun.d.ts` (`IApp`, `Component`, exports).
4. **"Choosing your event syntax" + "Which bundle/CDN"** explainer sections.
5. **Freshness sweep**: GA4 migration, copyright year, dead external links (Glitch, old
   lit-html/polymer URLs, animate.css 3.7.0), deprecated `safeHTML`.
6. **Copy edit**: typo/grammar pass, fix `reactivity.md` button bug and broken sample code
   in `routing.md`/`spa.md`/`component.md`.

---

## Appendix — files reviewed

Read in full and checked against source: `index.md`, `installation.md`, `tutorial.md`,
`architecture.md`, `component.md`, `routing.md`, `spa.md`, `directive.md`,
`event-pubsub.md`, `state-management.md`, `reactivity.md`, `esm.md`, `ssr.md`,
`dev-server.md`, `create-apprun-app.md`, `mkdocs.yml`, and `../apprun/README.md` +
`apprun.d.ts` + `src/router.ts`.

Not deep-reviewed (worth a follow-up pass): `architecture-practices.md`,
`view-patterns.md`, `svg.md`, `react.md`, `3rd-party-libs.md`, `cli-in-console.md`,
`strong-typing.md`, `unit-testing.md`, `notebooks.md`, the `apprun-site-*.md` set,
`architecture-ideas/*`, `showcase.md`, `resources.md`, `about.md`.
