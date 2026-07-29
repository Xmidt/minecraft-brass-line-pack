# The Brass Line

A Create-focused survival modpack for Minecraft 1.20.1 (Forge) — railways, automation, and industrial charm, managed with [packwiz](https://packwiz.infra.link/) for automatic client/server sync.

This repo **is** the pack. Both the dedicated server and every player's client point at `pack.toml` in this repo and stay in sync automatically — no manual mod-matching, no "you're a version behind" errors.

## Minecraft / Loader

| | |
|---|---|
| Minecraft | 1.20.1 |
| Modloader | Forge |
| Forge build | 47.4.10 |

## For players — Prism Launcher setup

1. Create a new instance on **Minecraft 1.20.1** with **Forge 47.4.10**.
2. Download [`packwiz-installer-bootstrap.jar`](https://github.com/packwiz/packwiz-installer-bootstrap/releases) and place it in the instance's `.minecraft` folder (same folder as `options.txt`).
3. In Prism: **Edit Instance → Settings → Custom Commands** → enable Custom Commands, and set the **pre-launch command** to:
   ```
   "$INST_JAVA" -jar packwiz-installer-bootstrap.jar https://raw.githubusercontent.com/Xmidt/minecraft-brass-line-pack/main/pack.toml
   ```
4. Launch. Every time you start the game, Prism checks this repo and pulls down anything new or changed — you'll never be out of sync with the server again.

## For the server

The server (itzg's `docker-minecraft-server` image via Podman Quadlet) has built-in packwiz support. Point it at the same URL:

```
Environment=PACKWIZ_URL=https://raw.githubusercontent.com/Xmidt/minecraft-brass-line-pack/main/pack.toml
```

```bash
systemctl --user daemon-reload
systemctl --user restart minecraft-brass-line
```

## Updating the pack

```bash
cd brass-line-pack

# Update everything to the latest compatible version
packwiz update --all

# Or update one mod
packwiz update biomes-o-plenty

# Add something new — pin by exact version URL, not just the slug (see gotcha below)
packwiz mr add https://modrinth.com/mod/<slug>/version/<version>

# Always refresh + commit + push after changes
packwiz refresh
git add .
git commit -m "Update mods"
git push
```

Push, then restart the server and relaunch Prism — both pull the change automatically. No re-uploading jars, no resending zip files.

## ⚠️ Known gotcha: JourneyMap and beta channels

**Do not run `packwiz update --all` or a bare `packwiz mr add journeymap` without checking the result.** packwiz does not distinguish release vs. beta channels when resolving "latest" — it picked JourneyMap's `6.0.0-beta.x` line for us once, purely because it was the most recently published file matching 1.20.1 + Forge, alpha/beta status notwithstanding.

This isn't a random pick to avoid — **Create ships its own JourneyMap compatibility mixins**, written against JourneyMap 5.x's internal class layout (`GridRenderer`, `MapRenderer`, `Fullscreen`, etc.). The 6.0.0 beta is a ground-up rewrite that renamed those exact classes, so Create's compat mixins fail to find their target and the whole client refuses to launch (`MixinTransformerError` / `ClassMetadataNotFoundException`). As long as Create is part of this pack, JourneyMap must stay on the `5.10.x` release line.

**After any update touching JourneyMap**, always verify:
```bash
cat mods/journeymap.pw.toml | grep filename
```
It should read `journeymap-1.20.1-5.10.3-forge.jar` (or a later `5.10.x` release) — never a `6.0.0-beta.x` build.

## Mod list

Mods are tagged `client`, `server`, or `both` in their `.pw.toml` files — packwiz filters automatically depending on which side is asking (`PACKWIZ_URL` on the server pulls `server` + `both`; the Prism pre-launch pulls `client` + `both`).

Check what's currently in the pack, and on which side, with:
```bash
packwiz list -s client
packwiz list -s server
```

### Create ecosystem (both)
Create · Create: Steam 'n' Rails · Create Railways Navigator · Create Crafts & Additions · Create: Bells & Whistles · Create: Interiors · Create Slice & Dice

### Survival & world (both)
Farmer's Delight · Supplementaries · Amendments · Quark · Waystones · Alex's Mobs · Biomes O' Plenty · GlitchCore · Serene Seasons · Open Parties and Claims

### UI / QoL (client only)
Mouse Tweaks · Inventory Profiles Next · Controlling · Controlify: Forgified · YetAnotherConfigLib (YACL)

> JEI, Jade, JourneyMap, and AppleSkin are tagged **both**, not client-only — each needs a server-side counterpart to sync data the vanilla protocol doesn't send by default (recipes, block/entity info, saturation values).

### Performance (client only)
FerriteCore · Entity Culling · Embeddium

### Required libraries (both)
Moonlight Lib · Zeta · Citadel · Balm · TerraBlender · DragonLib · Kotlin for Forge · libIPN · Searchables

### Auto-pulled dependencies
A handful of libraries get pulled in automatically by the mods above and aren't added directly — Flywheel & Ponder (Create), Architectury, KumaAPI & Common Networking (Controlify), TRender & MixinSquared (EntityCulling family), TRansition, and JourneyMap API. These don't need managing by hand; packwiz keeps them in step with whatever mod requires them.

`packwiz list` (no `-s` flag) is always the authoritative full list — this README is a categorized summary for humans, not the source of truth.

## Why packwiz?

Modded Minecraft requires the client and server to run the *exact* same mod versions. Before this pack existed, keeping everyone in sync meant manually diffing jar files after every server restart. Now:

- The server pulls its mod list from this repo via `PACKWIZ_URL`.
- Every player's Prism instance pulls the same list via a pre-launch hook.
- One `git push` here updates both, automatically, next restart/launch.
