# reallog — landing page Worker for reallog.ai

> ✅ **What this repo actually is:** a tiny Cloudflare Worker that serves a
> single static HTML page (the landing page for the `reallog.ai` domain).
> That's the whole codebase. The Worker has **no application logic** — it is a
> five-line asset passthrough.

This repo is **not** the Python package it advertises. The landing page describes
`reallog-agent`, a "Real-World Scene Logger" that turns camera frames into text
descriptions and writes them to PLATO as queryable tiles. That package is
published on PyPI and its source lives in a **separate** repo
([`SuperInstance/reallog-agent`](https://github.com/SuperInstance/reallog-agent)).
This repo is just the billboard that points at it.

If you arrived here expecting camera/vision/PLATO code, you want the `-agent`
package, not this repo.

## What problem does the *advertised* package aim at?

Most camera systems are alert machines: they detect motion, send a notification,
and the moment is gone. "Who came to the front door last Tuesday?" is left to
scrubbing video. `reallog-agent` is a stab at giving a camera durable memory:
camera frames → short text descriptions → PLATO tiles, so a later agent can
answer questions grounded in what the camera actually saw.

## Repository layout

```
reallog/
├── src/index.ts        # The Worker. Five lines. Delegates every request to env.ASSETS.
├── public/index.html   # The landing page (HTML + inlined CSS). The only served asset.
├── wrangler.jsonc      # Cloudflare Workers config: name="reallog", assets dir=./public
├── family/             # Shared design-system skeleton (see family/README.md)
└── .gitignore
```

**`src/index.ts`** in full:

```ts
export default {
  async fetch(request: Request, env: { ASSETS: Fetcher }): Promise<Response> {
    return env.ASSETS.fetch(request);
  },
};
```

Every request is handed to the Workers [Static Assets](https://developers.cloudflare.com/workers/static-assets/)
binding, which serves `public/index.html`. There is no routing, no server-side
rendering, no database, no secrets.

**`family/`** is a shared design system (CSS tokens + base components + an
"honesty" provenance panel) copied identically across a sibling family of
`*-log` landing-page repos. It is documented in its own
[`family/README.md`](family/README.md). Per this site's plan, the family CSS is
meant to be **inlined at build time** into each site's `<style>` block — and
indeed `public/index.html` already contains that CSS inlined verbatim.

## Run it

Requirements: Node.js and the Wrangler CLI (`npm i -g wrangler` or use `npx`).

```bash
# Local dev server (serves the landing page at http://localhost:8787)
npx wrangler dev

# Production deploy (deploys the Worker + static asset)
npx wrangler deploy
```

✅ Verified: `npx wrangler deploy --dry-run` succeeds for the sibling
`personallog` repo (identical Worker shape) — it reads `public/index.html` as
the single asset and binds `env.ASSETS`. There is **no test suite** in this repo
and no `package.json`; nothing to install beyond Wrangler itself.

## The landing page's quickstart (verified verbatim)

`public/index.html` shows this example and its banner claims it is "copied
verbatim from its README":

```python
from reallog_agent import RealLogAgent

agent = RealLogAgent(camera_id="security_cam_1")
agent.log_scene("front_door", "person approaching at 14:32", motion=True)
agent.log_scene("back_yard", "bird detected", motion=False)
print(agent.ask("who came to the front door?"))
```

✅ **Verified true.** I fetched the upstream README at
`SuperInstance/reallog-agent` and the code block above matches it character for
character. The upstream README also backs the architecture claims on the landing
page (camera frames → described text → PLATO tiles; agent reads PLATO → answers
visual questions; *"don the shell"* → view from another camera's perspective).

That said — this example describes the **package's** API, not anything in this
repo. This repo does not contain `RealLogAgent` or any Python.

## Honesty markers (verified, not assumed)

The org's discipline is that docs must not oversell code. Everything below was
traced against real artifacts, not copied from existing copy.

About **this repo** (the landing page Worker):

- ✅ **Real today:** builds and serves a static landing page via a Cloudflare
  Worker (same shape as its verified siblings). The HTML/CSS is real and
  self-contained.
- 🔮 **Not in this repo:** there is no PLATO integration, no vision/camera code,
  no `RealLogAgent`, and no build pipeline beyond Wrangler. The `family/README.md`
  describes a `build.mjs` inlining step as the intended pattern, but **no such
  build script exists in this repo** — the CSS is already inlined in
  `public/index.html` by hand, so the "build step" is currently a manual edit.

About the **advertised package** `reallog-agent` (separate repo/PyPI — I
verified these against PyPI and the upstream GitHub README, not assumed):

- ✅ **Real today:** genuinely published on PyPI as `reallog-agent` `0.1.0`
  (summary: *"Vision Turbo-Shell for reallog.ai — camera vision → PLATO →
  agent"*). HTTP 200 on `https://pypi.org/pypi/reallog-agent/json`.
- ✅ **Quickstart verified verbatim** from the upstream README (see above).
- ⚠️ **Real but not a finished product:** the landing page's **"SCENE LOGGER"**
  banner is precise — "a working library, not a polished product. The
  camera-to-text pipeline and PLATO integration are real; a consumer app around
  them is not shipped yet." I verified the *claims* against the upstream README,
  but I did **not** execute the package, so I cannot confirm the
  camera-to-text→PLATO path actually runs end-to-end.
- 🔮 **Not verified here:** whether PLATO itself (the shared tile-storage /
  agent-memory substrate) is running, and what backs the "camera frames →
  described text" step (an external vision model/service). Those live outside
  this repo.

## The sibling family

This repo is one of several near-identical landing-page Workers that each
advertise a different PLATO domain agent and share the `family/` design system:

| repo | domain | advertises | accent |
|---|---|---|---|
| personallog | personallog.ai | `personallog-agent` (personal notes) | `--depth` |
| playerlog | playerlog.ai | `playerlog-agent` (game-player activity) | `--antifoul` |
| **reallog** | reallog.ai | `reallog-agent` (camera scene logging) | `--lattice` |
| studylog | studylog.ai | `studylog-agent` (study-partner CLI) | `--reef` |

Each Worker is identical (`src/index.ts` is byte-for-byte the same); only the
`wrangler.jsonc` `name`, the landing-page HTML content, and the accent color
differ. See `family/README.md` for the shared design-system rationale.
