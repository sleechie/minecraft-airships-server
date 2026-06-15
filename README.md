# Minecraft — "Airships & Skylands" Server (Railway)

A dedicated **Fabric 1.20.1** server for the *Airships & Skylands* modpack
(Valkyrien Skies airships over an Aetherial Islands floating-islands world),
hosted on Railway with the [`itzg/minecraft-server`](https://github.com/itzg/docker-minecraft-server)
image — the de-facto standard MC server container.

Companion to the client setup doc: `minecraft-airships-skylands-setup.md`.

---

## How it's wired

| | |
|---|---|
| **Image** | `itzg/minecraft-server` (Docker, deployed directly — no build) |
| **Loader** | Fabric, MC `1.20.1`, loader `0.19.3` (auto-installed by the image) |
| **Mods** | 25 server-side jars, seeded from `mods.tgz` via `GENERIC_PACK` |
| **World** | the single-player "New World" save, seeded from `world.tar.gz` via `WORLD` |
| **Persistence** | Railway volume mounted at `/data` |
| **Connectivity** | Railway TCP proxy → `25565` |
| **Memory** | 6 GB heap |
| **Security** | `online-mode=TRUE`, whitelist enforced (InfantDropper) |

The world and mods are **not** committed to git — they're attached to the
GitHub **release** (`world.tar.gz` ≈ 936 MB, `mods.tgz` ≈ 77 MB) and pulled by
the container on first boot. After first boot the world lives on the `/data`
volume and is never re-fetched (`WORLD` only seeds when `/data/world` is absent).
`GENERIC_PACK` (mods) is re-applied on every boot, so the mods asset must stay
reachable.

### Environment variables (set on the Railway service)

```
EULA=TRUE
TYPE=FABRIC
VERSION=1.20.1
MEMORY=6G
MAX_TICK_TIME=-1          # no watchdog — heavy modded ticks won't self-kill
USE_AIKAR_FLAGS=true      # tuned G1GC flags for modded servers
ONLINE_MODE=TRUE
ENFORCE_WHITELIST=TRUE
OVERRIDE_WHITELIST=TRUE
WHITELIST=InfantDropper
OPS=InfantDropper
DIFFICULTY=normal
VIEW_DISTANCE=8
ENABLE_RCON=false
MOTD=Airships & Skylands
GENERIC_PACK=<release URL>/mods.tgz
WORLD=<release URL>/world.tar.gz
```

The 25 mods (client-only ones — Sodium, Iris, Indium, Continuity,
EntityCulling, Logical Zoom, Xaero's Minimap, REI, Mod Menu — are present in the
pack but auto-skipped by Fabric on a dedicated server via their `environment`
flag; the server-relevant ones — Valkyrien Skies, Eureka, Aetherial Islands,
Lithostitched, No Void Structures, VS Grapples, Grappling Hook Mod, Gliders,
Lithium, FerriteCore, Fabric API, libs — load and run).

---

## Operating it

- **Stop when not playing** (billing is per-runtime): Railway dashboard → service
  → remove the active deployment. It will **not** auto-restart.
- **Wake it**: Railway dashboard → Deploy (redeploy the service).
- **Add a friend to the whitelist**: append to `WHITELIST` env (comma-separated)
  and redeploy, or run `/whitelist add <name>` in the server console as op.
- **Add/replace a mod**: update `mods.tgz`, re-upload to the release, redeploy.
  `GENERIC_PACK` re-syncs `/data/mods` from the pack each boot.
- **Reset the world to the seeded copy**: set `FORCE_WORLD_COPY=TRUE` for one
  boot (replaces `/data/world` from the asset), then remove it.

---

*Built 2026-06-14. Personal project — Braden's personal Railway, not AMA.*
