# The Brass Line

A Create-focused survival modpack for Minecraft 1.20.1 (Forge) — railways, automation, and industrial charm, managed with [packwiz](https://packwiz.infra.link/) for automatic client/server sync.

This repo **is** the pack. Both the dedicated server and every player's client point at `pack.toml` in this repo and stay in sync automatically — no manual mod-matching, no "you're a version behind" errors.

## Minecraft / Loader

| | |
|---|---|
| Minecraft | 1.20.1 |
| Modloader | Forge |
| Forge build | see `pack.toml` — client and server must match exactly |

## For players — Prism Launcher setup

1. Create a new instance on **Minecraft 1.20.1** with the **Forge** version listed in `pack.toml`.
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

# Add something new
packwiz mr add <slug>

# Always refresh + commit + push after changes
packwiz refresh
git add .
git commit -m "Update mods"
git push
```

Push, then restart the server and relaunch Prism — both pull the change automatically. No re-uploading jars, no resending zip files.

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
JEI · Jade · JourneyMap · AppleSkin · Mouse Tweaks · Inventory Profiles Next · Controlling · Controlify: Forgified · YetAnotherConfigLib (YACL)

> Note: JEI, Jade, JourneyMap, and AppleSkin are actually tagged **both** in this pack, not client-only — each of them needs a server-side counterpart to sync data the vanilla protocol doesn't send by default (recipes, block/entity info, saturation values). See `packwiz list -s server` for the authoritative current list.

### Performance (client only)
FerriteCore · Entity Culling · Embeddium

### Required libraries (both)
Moonlight Lib · Zeta · Citadel · Balm · TerraBlender · DragonLib · Kotlin for Forge · libIPN · Searchables

## Why packwiz?

Modded Minecraft requires the client and server to run the *exact* same mod versions. Before this pack existed, keeping everyone in sync meant manually diffing jar files after every server restart. Now:

- The server pulls its mod list from this repo via `PACKWIZ_URL`.
- Every player's Prism instance pulls the same list via a pre-launch hook.
- One `git push` here updates both, automatically, next restart/launch.
