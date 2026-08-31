# GitGTA

Turn any GitHub profile into a drivable open-world city. Commit activity raises the skyline, repos become landmarks, and you drive/walk through it GTA-style — all rendered live in the browser with Three.js.

```
github.com/<you>  →  gitgta.com/<you>
```

This repo is meant to be forked. It's a working example of a pattern — **turn structured data from an API into a 3D world you can explore** — and everything needed to swap "GitHub profile" for your own data source is laid out below.

## Live demo

Paste any GitHub username and drive their city. WASD to walk/drive, Shift to sprint/boost, Space to jump/handbrake, E to enter/exit a vehicle, mouse to look, LMB to punch, RMB for the pistol.

## Stack

- **Next.js 16** (App Router, Turbopack) + React 19 + TypeScript
- **Three.js** for the 3D scene — no game engine, no React Three Fiber, just a plain render loop
- **GitHub GraphQL API** as the only external dependency (contribution calendar + repos)
- Tailwind for the 2D UI chrome (landing page, HUD)

No database. No auth. Every page is generated on request from a GitHub username in the URL.

## How it works

```
app/[owner]/page.tsx          → intro screen (fetches the profile, shows "Enter city")
app/api/city/[owner]/route.ts → API route: username in, CityPayload JSON out
lib/github-client.ts          → GraphQL query against api.github.com/graphql
lib/city-from-github.ts       → the CityPayload type (the data contract)
components/GameCanvas.tsx     → mounts the Three.js canvas, hands it the payload
game/boot.ts                  → wires up the render loop, systems, and UI screens
game/world/district.ts        → turns CityPayload.days into a city layout (this is the core trick)
game/render/*                 → building/hero meshes, textures, camera
game/systems/*                → vehicle physics, collision, combat, missions, NPCs, wanted level
```

**The core trick** is in `game/world/district.ts`: GitHub's contribution calendar is a 52×7 grid of `{ date, count }`. That grid *is* the city grid — each day becomes a lot, `count` becomes building height, and repos from the profile get placed as named landmarks. Swap that one file's input for a different time-series-shaped dataset (Spotify listening history, Strava activity, stock volume, whatever) and you have a different city generator on the same engine.

## Quickstart

```bash
git clone https://github.com/okaybelal/GitGTA.git
cd GitGTA
npm install
cp .env.example .env
```

Add a GitHub token to `.env`:

```
GITHUB_TOKEN=github_pat_xxxxxxxxxxxx
```

Generate one at [github.com/settings/tokens](https://github.com/settings/tokens) — a fine-grained token with **read-only access to public repositories** is enough. No scopes beyond that are needed unless you also want organization profiles to work (`read:org`).

```bash
npm run dev
```

Open `http://localhost:3000`, paste a GitHub username, drive.

## Making it your own

This is meant as a starting point, not a finished product to just re-skin. A few places to start:

- **Different data source** — replace `lib/github-client.ts` + `lib/city-from-github.ts` with a fetch against any API that returns a time series. Keep the `CityPayload` shape (or adapt `district.ts` to a new shape) and the rest of the engine — physics, vehicles, camera, HUD — keeps working.
- **Different city rules** — `game/world/district.ts` is where counts become geometry. Change block size, height scaling, road layout, or what triggers a landmark.
- **Different vehicles/characters** — drop new `.glb` models into `public/models/` and point `game/render/*` at them.
- **Different game feel** — `game/systems/mission.ts`, `wanted.ts`, and `combat.ts` are self-contained systems you can strip out or extend independently.

If you build something with this, tag it — I'd like to see it.

## Deploying

Deploys as-is on [Vercel](https://vercel.com/new): import the repo, set `GITHUB_TOKEN` as an environment variable, deploy. No other infra required.

## Acknowledgements

Originally built by [filiksyos](https://github.com/filiksyos/gitgta). This repo is a fork, restructured and documented as a reusable template.

## License

MIT — see [LICENSE](LICENSE). Do whatever you want with it.
