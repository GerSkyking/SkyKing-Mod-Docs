# SkyKing Mod Docs

GitHub‑Wiki‑style documentation for the SkyKing Arma Reforger mods (SkyMap-X / SpeedUI / SIDC-Framework / Throw Back Grenade) and the companion tools (ATAKmaps, SIDC – C2 – Command & Control).
Bilingual: every page has the English version first, then a `---`, then the German version (Deutsch).

## Using this as a GitHub Wiki

The files use GitHub‑Wiki naming conventions:

- `Home.md` — wiki landing page
- `_Sidebar.md` / `_Footer.md` — wiki chrome
- other `*.md` — one page each; links use bare page names, e.g. `[Keybinds](Keybinds)`

To publish: push the contents of this folder to the `*.wiki.git` repo of the target project
(e.g. `git clone https://github.com/<you>/SkyMap-X.wiki.git`, copy files in, commit, push).
Or keep it as a normal docs folder and read the `.md` files directly.

## Pages

| Page | Content |
|------|---------|
| `Home` | Overview & entry points |
| `SkyMap-X` | SMX — overview, usage, settings, keybinds, WIP |
| `SpeedUI` | SUI — overview, usage, settings |
| `SIDC-Framework` | SIDC — overview, usage, SIDC digits, quick‑marker config, architecture |
| `SIDC-Framework-Channels` | SIDC channel & physical‑channel system, visibility matrix |
| `Throw-Back-Grenade` | TBG — overview, usage, keybind, modding, architecture |
| `ATAKmaps` | Companion web‑map tool — setup, server API, SIDC data interface |
| `SIDC-C2-Command-Control` | Multi‑user planning tool (Docker stack) — deploy, roles, plans/phases/versions, map view, contour/peak layers |
| `Keybinds` | Combined keybind reference for all mods |
| `Server-Admin-Guide` | All server / admin configuration |
| `Client-Settings-Guide` | All per‑player settings |
| `Glossary` | Terms |

## Source of truth

Content is derived from the repos `SkyMap-X-DEV`, `SpeedUI`, `SIDC-Framework`, the
`Throw Back Grenade` Workbench addon, and the `ATAKmaps` tool at `D:\Mods\ATAKmaps`
(READMEs, `Doku/`, `CLAUDE.md`, `Configs/**`, `Scripts/**`).
When the mods or the tool change, update the matching page here.
