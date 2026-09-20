# SIDC – C2 – Command & Control (companion tool)

> Source: repo `github.com/GerSkyking/SIDC---C2---Command-Control` (AGPL-3.0) · image `ghcr.io/gerskyking/sidc---c2---command-control` · local dev at `C:\Users\Sky\Documents\GitHub\SIDC - C2 - Command & Control`. Verified against `README.md`, `docs/HANDBUCH.md`, `CHANGELOG.md`, `PLAN.md`, `deploy/.env.example`. **Version 1.0.0** (2026-09-07).

## 1. What it is

**SIDC – C2** is a browser‑based, **multi‑user operational planning tool**. It reuses the map and
military‑symbol logic of **[ATAKmaps](ATAKmaps)** / the **[SIDC-Framework](SIDC-Framework)** mod,
but has **no connection to a running game** — nothing is read from or written to Reforger. Users
log in, pick a map, and place SIDC markers and drawings **together in real time**.

Think of it as: the ATAKmaps web map, turned into a collaborative "sand table" that runs on a
server, with accounts, permissions, plans, folders, phases and version history.

| | ATAKmaps | SIDC – C2 |
|---|---|---|
| Data source | live game files (`$profile:.../LocalMapData/`) | its own database, no game link |
| Users | one local player | many, with accounts & roles |
| Runs | on the player's / server's machine | as a Docker stack on a Linux server |
| Persistence | the game | PostgreSQL (plans, markers, versions) |
| Realtime | SSE from a file watcher | WebSocket, authoritative server |
| Shared with ATAKmaps | map packages, milsymbol rendering, SIDC catalogs, wizard/quick‑menu, grid | |

It is shipped as a **Docker Compose stack** (`db` + `redis` + `backend`). The backend container is
the single entry point and serves the frontend, the REST API and the WebSocket on one port.

## 2. Architecture

| Component | Tech |
|-----------|------|
| Backend | FastAPI (Python 3.12), SQLAlchemy 2.0 ORM |
| Database | PostgreSQL |
| Realtime backplane | Redis Pub/Sub |
| Frontend | Vanilla TypeScript + Vite 6 + MapLibre‑GL 5 + **milsymbol@3**, built into the backend image |
| Auth | `pwdlib` argon2 hashes + signed HttpOnly session cookie (`itsdangerous`), optional OIDC via `authlib`, login rate limit |
| TLS / external access | **external** reverse proxy (e.g. Nginx Proxy Manager) — no proxy inside the stack |

- **Schema management:** no Alembic. On start the backend runs `Base.metadata.create_all()` plus a
  small auto‑migration (`ALTER TABLE ADD COLUMN` for new simple columns). New tables are created
  automatically.
- **Volumes:** `pgdata` (database), `maps` → `/data/maps` (imported map packages), `uploads` →
  `/data/uploads` (SIDC catalogs at `/data/uploads/catalog`, generated `secret_key`).
- **Realtime:** `backend/app/routers/live.py` gates every WebSocket operation server‑side against the
  caller's effective capabilities and the per‑marker lock. Create / freehand draw are optimistic on
  the client and then server‑confirmed; move / modify / delete / lock are server‑authoritative only.

## 3. Install & deploy

```bash
cp deploy/.env.example .env
# edit .env: passwords, SECRET_KEY (or leave empty -> auto), FORWARDED_ALLOW_IPS, MAP_IMPORT_MAX_MB
docker compose up -d
```

Then, in your reverse proxy, add a proxy host pointing at `http://<docker-host>:8080` and
**enable WebSocket support**. First login uses `BOOTSTRAP_ADMIN_USER` / `BOOTSTRAP_ADMIN_PASSWORD`
from `.env` (idempotently created on every start).

Update an existing deployment:

```bash
cd /opt/sidc-c2 && git pull && docker compose up -d --build
```

### Key `.env` settings

| Var | Meaning |
|-----|---------|
| `HTTP_PORT` | container port (default `8080`) |
| `FORWARDED_ALLOW_IPS` | IP(s) of the proxy that `X-Forwarded-*` is trusted from |
| `COOKIE_SECURE` | `auto` / `true` / `false` — Secure flag of the session cookie |
| `SECRET_KEY` | cookie signing key; empty ⇒ generated on first start |
| `DATABASE_URL` / `REDIS_URL` | connection strings (defaults point at the compose services) |
| `BOOTSTRAP_ADMIN_USER` / `_PASSWORD` | local admin account |
| `MAP_IMPORT_MAX_MB` | hard limit for map‑package uploads (default `2048`) |
| `OIDC_ENABLED` + `OIDC_*` | optional external login; local login stays active |

### Large map uploads (multi‑GB packages)

Two things must be raised together:

1. **Reverse proxy** (per proxy host → *Advanced*):
   ```nginx
   client_max_body_size 5120m;
   proxy_request_buffering off;
   proxy_read_timeout 3600s;
   proxy_send_timeout 3600s;
   client_body_timeout 3600s;
   send_timeout 3600s;
   ```
   `proxy_request_buffering off;` is essential — otherwise the whole upload is buffered in the proxy
   before it reaches the backend.
2. **Backend** `.env`: `MAP_IMPORT_MAX_MB=5120` (the backend streams the body straight to disk and
   rejects anything above this limit).

The `maps` volume needs roughly **2×** the package size free during an import (zip + unpacked copy).

## 4. Roles & permissions

Two levels:

- **Global role** – `admin` or `user`. Admin sees the admin area; unknown OIDC users become `user`.
- **Capability** – `can_create_plans` (per user or group), controls whether someone may create plans.

Per‑plan access is an **ACL** of user/group entries, each with a level:

| Level | Can |
|-------|-----|
| `viewer` | open the plan, see markers/drawings, read notes & versions |
| `editor` | + place / move / delete markers, draw — **each of the four toggled individually** |
| `owner` | + rename, manage ACL, public shares, restore versions, delete; can override marker locks |

A marker can be **locked**; only an owner (or the lock holder) can then move/modify/delete it.

## 5. Plans, folders & versions

- **Folder tree** with sub‑folders; plans and folders are re‑organised by **drag & drop**. Cycles
  are prevented; deleting a folder lifts its content one level up.
- **Clone** a whole plan into a target folder, optionally with a new name and the ACL copied.
- **Version history** – an editor/owner saves a named version at any time; the list shows label,
  date, author and marker/stroke counts. **Restore** rolls the plan back to that state and first
  writes an automatic backup version.
- **Trash** – deleted plans are soft‑deleted.

## 6. Map view

### 6.1 Toolbar (left, edit permission only)

Pan · **Move marker** (mode, or hold the middle mouse button in pan mode) · Point ("laser" cursor,
also for viewers) · Straight line (ends on **right‑click**) · **Ruler** (two points → dashed
distance line, click again to clear) · Eraser (also deletes drawn lines) · **Place marker** (click
the position first, *then* the wizard opens) · Favorites.

### 6.2 Marker wizard

Quick‑menu (affiliation → category → symbol) + full catalog search + free `SIDC` field. Only the
four affiliations green / blue / red / unknown (no "assumed"). **Advanced** panel: modifier
dropdowns for the current symbol, read from `SIDC_ModifierCatalog.json` and spliced into SIDC
digits 6 / 7 / 16–19. Fields: unit text, extra text, channel, lock, timestamp visibility. Line
symbols repeat until a **right‑click**; favorites that build lines behave the same.

### 6.3 Marker label & hover

- **Permanent label** under every marker: name (unit text, otherwise the catalog symbol name) +
  the set modifiers, comma‑separated.
- **Hover** a marker → tooltip with **channel**, **creator** and **phase**.

### 6.4 Marker editing

Clicking a marker (in pan or move mode) opens a centred window (click outside / **Esc** closes):
unit & extra text, icon rotation, **phase**, lock, **Advanced** (modifiers read from the SIDC and
editable), **Clone**, **Save as favorite**, **Delete**.

### 6.5 Symbol rendering (hybrid)

Primary: **milsymbol.js** (`standard: "APP6"`, 30‑digit SIDC). Fallback: a pre‑rendered PNG from the
catalog. Last resort: a coloured dot. **Direction arrows** are drawn as vector geometry, oriented
like ATAKmaps (air = extended line + arrowhead, ground = fixed stem + rotating arrow).

### 6.6 Topbar

| Element | Function |
|---------|----------|
| **3D** | toggle terrain / pitch; 2D allows rotation but no pitch |
| **Compass ↑** | shows north; click = ease bearing back to north |
| **Date/time field** | overrides "now" for the screenshot timestamp |
| **📷 Screenshot** | PNG of map content only (no UI, no MapLibre logo); military DTG `DDHHMMZ MMM YY` bottom‑left; filename `<Plan>_<Phase>_<DDMMYYYY-HHMMSS>.png` |
| **Channel** | radio channel from `SIDC_ChannelSettings.json` |
| **Timeline** | phases — see 7 |
| **🗒️ Notes** | movable phase‑notes window — see 7 |
| **☰ Layers** | Sat / Grid / Terrain, Contour lines, Elevation points, place groups — each with a **0–100 % opacity slider** (stored per browser) |
| **🕑 Versions** | version history — see 5 |
| **? Help** | overlay: tools, topbar, keybinds (DE/EN) |
| Cursor HUD | world X / Y (via calibration) + elevation from the terrain DEM, in 2D as well |

## 7. Phases & phase notes

- Creating a plan makes a default phase **"Base"**. Add / delete phases; markers of a deleted phase
  become "global".
- Each marker is assigned to a phase (or global). Markers **outside the current phase** render at a
  configurable low opacity (slider, 0–100 % in 5 % steps).
- **Phase notes**: a movable, resizable window (black background, white text) with **Markdown**
  rendering and **one tab per phase**. Switching phase switches the tab. Closable with the ✕,
  re‑opened from the topbar. Auto‑saved (debounced).

## 8. Layers from the heightmap: contour lines & elevation points

Two extra map layers derived from the game heightmap, **pre‑computed once in the ATAKmaps importer**
(`pipeline/terrain_features.py`, numpy/scipy only) and shipped inside the map package as
`contours.geojson` / `peaks.geojson`. Both consumers (ATAKmaps viewer **and** SIDC – C2) only
display them; default **off**, toggled in the ☰ layers panel.

| Layer | How it is built |
|-------|-----------------|
| **Contour lines** | vectorised marching‑squares on a 2 m grid, one line every 10 m, every 5th line bold + labelled (white text, black outline), Douglas‑Peucker‑simplified |
| **Elevation points** | *local* prominence (Prim bottleneck search within a radius) on a 4 m grid, hard prominence cut‑off + spatial non‑maximum suppression, rough shape class (hill / ridge / plateau / dominant peak); rendered as `▲ NNN m` |

The parameters (contour interval, bold interval, min. prominence, min. distance, search radius,
work‑cell size) are exposed in the importer's *Heightmap CSV* box. The button **"Nur Höhenlinien /
Höhenpunkte neu berechnen"** re‑generates the two files without re‑tiling; then re‑export the map
package and re‑upload it to C2.

To get **more** elevation points: lower *min. distance* (biggest lever, e.g. 250 → 130) and
*min. prominence* (e.g. 20 → 12), optionally lower *search radius* (300 → 150).

## 9. Realtime collaboration

WebSocket per plan at `/plans/{plan_id}/live`. The server (`live.py`) checks every op against the
caller's capabilities and marker locks:

- `marker.create`, `stroke.*` — client renders optimistically with a temp id, server confirms with
  the real id (`marker.upsert` incl. `cid`) or rejects.
- `marker.move` / `marker.modify` / `marker.lock` / `marker.delete` — server‑authoritative,
  broadcast to the plan room.
- `presence.cursor` — always allowed (also for viewers: the "point" tool).

## 10. Public share links

An owner creates a token; `#/p/<token>` opens the plan **read‑only, without login** — map, markers,
drawings, places, contour/peak layers. Deleting the token invalidates the link.

## 11. Admin area

Users, groups, **catalog upload** (5 files, see 12), maps (import / update / DLC / delete), audit
log. UI is bilingual **DE / EN**; map place labels support 13 languages (independent of the UI
language).

## 12. Catalog files

C2 uses the **same `LocalMapData` catalog files** as the [SIDC-Framework](SIDC-Framework) mod. The
admin uploads them (raw JSON body, not multipart — a large file may need the proxy's
`client_max_body_size` raised):

| File | Used for |
|------|----------|
| `SIDC_AllMarkersCatalog.json` | full marker catalog; the `subCategory` field keys into the modifier catalog |
| `SIDC_ModifierCatalog.json` | modifier options per sub‑category (the "Advanced" dropdowns) |
| `SIDC_ChannelSettings.json` | radio channels |
| `SIDC_QuickMenuSettings.json` | quick‑marker menu layout |
| `SIDC_PhaseLineSettings.json` | line colours / widths |

After changing `SIDC_AllMarkersCatalog.json` (e.g. the added `subCategory` field) also re‑upload the
matching `SIDC_ModifierCatalog.json`.

## 13. Map packages

C2 imports the exact same `*_mappack_v*.zip` packages produced by the **ATAKmaps importer**
(`pipeline/map_packages.py`, *Export* tab). Layout:

```
mappack.json          manifest (optional)
calibration.json      origin + per‑axis scale (game metres ↔ fake WGS‑84 degrees)
mbtiles/sat.mbtiles   required
mbtiles/terrain.mbtiles   optional — 3D + elevation readout (Terrarium RGB)
mbtiles/grid.mbtiles      optional — coordinate grid raster
mbtiles/dlc/<layer>_z<N>.mbtiles   optional — high‑res zoom levels
topo.geojson          optional — roads/paths (display currently disabled)
locations.json         optional — named places, grouped by baseType
contours.geojson       optional — contour lines (see 8)
peaks.geojson          optional — dominant elevation points (see 8)
```

All raw‑data processing (`.topo` → GeoJSON, `mapLocations` → `locations.json`, heightmap →
`contours`/`peaks`) happens **once** in the ATAKmaps importer at pack‑build time. Import in C2 via
the admin area (download link or direct upload).

### 13.1 Map server for ATAKmaps (API tokens)

C2 can act as the **map source of the ATAKmaps client** (see [ATAKmaps](ATAKmaps)): players load maps from C2 instead of installing packages. Markers, live position and calibration stay on the player's machine.

1. **User:** *Settings (gear) → API tokens → Create token* (name, optional expiry). Copy the token — it is shown **once**; only its SHA‑256 hash is stored. Tokens can be revoked at any time (the client loses access immediately).
2. **ATAKmaps:** Setup GUI → tab *Kartenserver* → name, address (`https://…`), token.

Rules:
- A token has the scope `maps:read` and works **only on the read‑only map routes**: `GET /api/maps`, `/api/maps/{id}/tiles/…`, `style.json`, `topo.geojson`, `locations.json`, `contours.geojson`, `peaks.geojson` (plus `GET /api/client/ping` as a connection test). Plans, markers, drawings, user data, map management and token management are cookie‑session only — a token never gets through there. Rate‑limited per token.
- All signed‑in users may load all maps; there are no per‑user map permissions.
- The client gets whatever quality C2 holds (imported base map, optional DLC). Locally installed maps/DLC take precedence and can add higher zoom levels.
- A re‑import of a map in C2 changes its `imported_at`, which makes clients drop their cached tiles of that map.

## 14. Related

- [ATAKmaps](ATAKmaps) — the game‑connected companion tool C2 is derived from; produces the map
  packages and catalogs.
- [SIDC-Framework](SIDC-Framework) — the mod that defines the SIDC symbols, catalogs and quick menu.
- [SIDC-Framework-Channels](SIDC-Framework-Channels) — the channel model reused for marker channels.
- Full user manual in the repo: `docs/HANDBUCH.md` (German). Changelog: `CHANGELOG.md`.

---

# SIDC – C2 – Command & Control (Begleit-Tool) — Deutsch

> Quelle: Repo `github.com/GerSkyking/SIDC---C2---Command-Control` (AGPL-3.0) · Image `ghcr.io/gerskyking/sidc---c2---command-control` · lokal `C:\Users\Sky\Documents\GitHub\SIDC - C2 - Command & Control`. Verifiziert gegen `README.md`, `docs/HANDBUCH.md`, `CHANGELOG.md`, `PLAN.md`, `deploy/.env.example`. **Version 1.0.0** (2026-09-07).

## 1. Kurzbeschreibung

**SIDC – C2** ist ein browser­basiertes, **mehrbenutzerfähiges Einsatz-Planungstool**. Es nutzt die
Karten- und Militärsymbol-Logik von **[ATAKmaps](ATAKmaps)** bzw. dem
**[SIDC-Framework](SIDC-Framework)**-Mod, hat aber **keine Verbindung zu einem laufenden Spiel** —
es wird nichts aus Reforger gelesen oder geschrieben. Nutzer melden sich an, wählen eine Karte und
setzen **gemeinsam in Echtzeit** SIDC-Marker und Zeichnungen.

Kurz: die ATAKmaps-Web-Karte, umgebaut zu einem kollaborativen „Sandkasten" auf einem Server — mit
Konten, Rechten, Plänen, Ordnern, Phasen und Versionsverlauf.

| | ATAKmaps | SIDC – C2 |
|---|---|---|
| Datenquelle | Live-Spieldateien (`$profile:.../LocalMapData/`) | eigene Datenbank, keine Spielanbindung |
| Nutzer | ein lokaler Spieler | viele, mit Konten & Rollen |
| Läuft | auf der Maschine des Spielers/Servers | als Docker-Stack auf einem Linux-Server |
| Persistenz | das Spiel | PostgreSQL (Pläne, Marker, Versionen) |
| Echtzeit | SSE aus einem Datei-Watcher | WebSocket, autoritativer Server |
| Gemeinsam mit ATAKmaps | Kartenpakete, milsymbol-Rendering, SIDC-Kataloge, Wizard/QuickMenü, Grid | |

Auslieferung als **Docker-Compose-Stack** (`db` + `redis` + `backend`). Der Backend-Container ist
der einzige Einstiegspunkt und liefert Frontend, REST-API und WebSocket auf einem Port.

## 2. Architektur

| Komponente | Technik |
|------------|---------|
| Backend | FastAPI (Python 3.12), SQLAlchemy 2.0 |
| Datenbank | PostgreSQL |
| Realtime-Backplane | Redis Pub/Sub |
| Frontend | Vanilla-TypeScript + Vite 6 + MapLibre-GL 5 + **milsymbol@3**, in das Backend-Image gebaut |
| Auth | `pwdlib`-argon2-Hashes + signierter HttpOnly-Session-Cookie (`itsdangerous`), optional OIDC (`authlib`), Login-Ratelimit |
| TLS / externer Zugriff | **externer** Reverse Proxy (z. B. Nginx Proxy Manager) — kein Proxy im Stack |

- **Schema:** kein Alembic. Beim Start `Base.metadata.create_all()` + Mini-Automigration
  (`ALTER TABLE ADD COLUMN` für neue einfache Spalten). Neue Tabellen entstehen automatisch.
- **Volumes:** `pgdata` (DB), `maps` → `/data/maps` (importierte Kartenpakete), `uploads` →
  `/data/uploads` (SIDC-Kataloge unter `/data/uploads/catalog`, generierter `secret_key`).
- **Realtime:** `backend/app/routers/live.py` prüft jede WebSocket-Operation serverseitig gegen die
  effektiven Rechte des Aufrufers und den Marker-Lock. Anlegen / Freihandmalen laufen clientseitig
  optimistisch und werden dann server-bestätigt; Bewegen / Ändern / Löschen / Sperren sind rein
  server-autoritativ.

## 3. Installation & Deploy

```bash
cp deploy/.env.example .env
# .env anpassen: Passwörter, SECRET_KEY (oder leer -> auto), FORWARDED_ALLOW_IPS, MAP_IMPORT_MAX_MB
docker compose up -d
```

Dann im Reverse Proxy einen Proxy-Host auf `http://<docker-host>:8080` anlegen und
**WebSocket-Unterstützung aktivieren**. Erst-Login mit `BOOTSTRAP_ADMIN_USER` /
`BOOTSTRAP_ADMIN_PASSWORD` aus der `.env` (bei jedem Start idempotent angelegt).

Bestehendes Deployment aktualisieren:

```bash
cd /opt/sidc-c2 && git pull && docker compose up -d --build
```

### Wichtige `.env`-Werte

| Variable | Bedeutung |
|----------|-----------|
| `HTTP_PORT` | Container-Port (Default `8080`) |
| `FORWARDED_ALLOW_IPS` | IP(s) des Proxys, dem `X-Forwarded-*` geglaubt wird |
| `COOKIE_SECURE` | `auto` / `true` / `false` — Secure-Flag des Session-Cookies |
| `SECRET_KEY` | Signierschlüssel; leer ⇒ beim ersten Start erzeugt |
| `DATABASE_URL` / `REDIS_URL` | Connection-Strings (Defaults zeigen auf die Compose-Services) |
| `BOOTSTRAP_ADMIN_USER` / `_PASSWORD` | lokales Admin-Konto |
| `MAP_IMPORT_MAX_MB` | hartes Limit für Kartenpaket-Uploads (Default `2048`) |
| `OIDC_ENABLED` + `OIDC_*` | optionaler externer Login; lokaler Login bleibt aktiv |

### Große Karten-Uploads (Pakete mit mehreren GB)

Zwei Dinge müssen zusammen hoch:

1. **Reverse Proxy** (pro Proxy-Host → *Advanced*):
   ```nginx
   client_max_body_size 5120m;
   proxy_request_buffering off;
   proxy_read_timeout 3600s;
   proxy_send_timeout 3600s;
   client_body_timeout 3600s;
   send_timeout 3600s;
   ```
   `proxy_request_buffering off;` ist entscheidend — sonst puffert der Proxy den ganzen Upload,
   bevor er das Backend erreicht.
2. **Backend** `.env`: `MAP_IMPORT_MAX_MB=5120` (das Backend streamt den Body direkt auf die Platte
   und lehnt alles über dem Limit ab).

Das `maps`-Volume braucht während eines Imports ca. **2×** Paketgröße frei (ZIP + entpackte Kopie).

## 4. Rollen & Rechte

Zwei Ebenen:

- **Globale Rolle** – `admin` oder `user`. Admin sieht den Admin-Bereich; unbekannte OIDC-User
  werden `user`.
- **Capability** – `can_create_plans` (pro User oder Gruppe): darf jemand Pläne anlegen.

Der Zugriff pro Plan ist eine **ACL** aus User-/Gruppen-Einträgen mit je einer Stufe:

| Stufe | Darf |
|-------|------|
| `viewer` | Plan öffnen, Marker/Zeichnungen sehen, Notizen & Versionen lesen |
| `editor` | + Marker setzen / bewegen / löschen, malen — **jedes der vier einzeln schaltbar** |
| `owner` | + umbenennen, ACL verwalten, öffentliche Freigaben, Versionen wiederherstellen, löschen; kann Marker-Locks übergehen |

Ein Marker kann **gesperrt** werden; danach kann nur ein Owner (bzw. der Lock-Halter) ihn
bewegen/ändern/löschen.

## 5. Pläne, Ordner & Versionen

- **Ordnerbaum** mit Unterordnern; Pläne und Ordner werden per **Drag & Drop** umsortiert. Zyklen
  werden verhindert; ein gelöschter Ordner schiebt seinen Inhalt eine Ebene nach oben.
- **Klonen** eines ganzen Plans in einen Zielordner, optional mit neuem Namen und kopierter ACL.
- **Versionsverlauf** – ein Editor/Owner speichert jederzeit eine benannte Version; die Liste zeigt
  Label, Datum, Autor und Marker-/Strich-Anzahl. **Wiederherstellen** setzt den Plan auf diesen
  Stand zurück und schreibt vorher automatisch eine Sicherungs-Version.
- **Papierkorb** – gelöschte Pläne sind zunächst nur „soft-deleted".

## 6. Kartenansicht

### 6.1 Werkzeugleiste (links, nur mit Bearbeitungsrecht)

Karte bewegen · **Marker verschieben** (Modus, oder mittlere Maustaste im Bewegen-Modus halten) ·
Zeigen („Laser"-Cursor, auch für Viewer) · Gerade Linie (endet per **Rechtsklick**) · **Lineal**
(zwei Punkte → gestrichelte Distanzlinie, erneuter Klick löscht) · Radierer (löscht auch
gezeichnete Linien) · **Marker setzen** (erst Position klicken, *dann* öffnet der Wizard) ·
Favoriten.

### 6.2 Marker-Wizard

QuickMenü (Zugehörigkeit → Kategorie → Symbol) + volle Katalogsuche + freies `SIDC`-Feld. Nur die
vier Zugehörigkeiten grün / blau / rot / unbekannt (kein „assumed"). **Advanced**-Panel:
Modifikator-Dropdowns für das aktuelle Symbol, gelesen aus `SIDC_ModifierCatalog.json` und in die
SIDC-Stellen 6 / 7 / 16–19 eingesetzt. Felder: Einheitstext, Zusatztext, Channel, Sperren,
Zeitstempel. Linien-Symbole wiederholen bis zum **Rechtsklick**; linienbildende Favoriten genauso.

### 6.3 Marker-Beschriftung & Hover

- **Dauerhafte Beschriftung** unter jedem Marker: Name (Einheitstext, sonst der Symbolname aus dem
  Katalog) + die gesetzten Modifikatoren, durch Komma getrennt.
- **Maus über einen Marker** → Tooltip mit **Channel**, **Ersteller** und **Phase**.

### 6.4 Marker bearbeiten

Klick auf einen Marker (Modus Bewegen/Verschieben) öffnet ein zentriertes Fenster (Klick außerhalb
/ **Esc** schließt): Einheits- & Zusatztext, Icon-Drehung, **Phase**, Sperren, **Advanced**
(Modifikatoren aus dem SIDC gelesen und änderbar), **Klonen**, **Als Favorit speichern**,
**Löschen**.

### 6.5 Symbol-Rendering (hybrid)

Primär: **milsymbol.js** (`standard: "APP6"`, 30-stelliges SIDC). Fallback: ein vorgerendertes PNG
aus dem Katalog. Letzte Stufe: ein farbiger Punkt. **Richtungspfeile** als Vektor-Geometrie,
orientiert wie ATAKmaps (Luft = verlängerte Linie + Pfeilspitze, Boden = fester Steg + drehender
Pfeil).

### 6.6 Topbar

| Element | Funktion |
|---------|----------|
| **3D** | Terrain / Pitch umschalten; 2D erlaubt Drehen, aber kein Kippen |
| **Kompass ↑** | zeigt Norden; Klick = Blickrichtung sanft auf Nord |
| **Datum/Zeit-Feld** | überschreibt „jetzt" für den Screenshot-Zeitstempel |
| **📷 Screenshot** | PNG nur mit Karteninhalt (keine UI, kein MapLibre-Logo); militärischer DTG `DDHHMMZ MMM YY` unten links; Dateiname `<Plan>_<Phase>_<DDMMYYYY-HHMMSS>.png` |
| **Channel** | Funkkanal aus `SIDC_ChannelSettings.json` |
| **Zeitstrahl** | Phasen — siehe 7 |
| **🗒️ Notizen** | verschiebbares Phasen-Notizfenster — siehe 7 |
| **☰ Ebenen** | Sat / Grid / Terrain, Höhenlinien, Höhenpunkte, Orts-Gruppen — je mit **0–100 %-Deckkraft-Regler** (pro Browser gespeichert) |
| **🕑 Versionen** | Versionsverlauf — siehe 5 |
| **? Hilfe** | Overlay: Werkzeuge, Topbar, Tastenkürzel (DE/EN) |
| Cursor-HUD | Welt-X / Y (über Kalibrierung) + Höhe aus dem Terrain-DEM, auch in 2D |

## 7. Phasen & Phasen-Notizen

- Beim Anlegen eines Plans entsteht die Standard-Phase **„Base"**. Phasen hinzufügen / löschen;
  Marker einer gelöschten Phase werden „global".
- Jeder Marker gehört zu einer Phase (oder global). Marker **außerhalb der aktuellen Phase** werden
  mit einstellbar niedriger Deckkraft gezeichnet (Slider, 0–100 % in 5er-Schritten).
- **Phasen-Notizen**: ein verschiebbares, größenverstellbares Fenster (schwarzer Hintergrund, weiße
  Schrift) mit **Markdown**-Rendering und **einem Reiter je Phase**. Phasenwechsel wechselt den
  Reiter. Mit ✕ schließbar, aus der Topbar wieder aufrufbar. Automatisch gespeichert (entprellt).

## 8. Ebenen aus der Heightmap: Höhenlinien & Höhenpunkte

Zwei zusätzliche Kartenebenen aus der Spiel-Heightmap, **einmalig im ATAKmaps-Importer vorberechnet**
(`pipeline/terrain_features.py`, nur numpy/scipy) und als `contours.geojson` / `peaks.geojson` im
Kartenpaket. Beide Consumer (ATAKmaps-Viewer **und** SIDC – C2) zeigen sie nur an; Default **aus**,
Umschalten im ☰-Ebenen-Panel.

| Ebene | Erzeugung |
|-------|-----------|
| **Höhenlinien** | vektorisiertes Marching-Squares auf 2-m-Raster, alle 10 m eine Linie, jede 5. fett + beschriftet (weiße Schrift, schwarze Umrandung), Douglas-Peucker-vereinfacht |
| **Höhenpunkte** | *lokale* Prominenz (Prim-Bottleneck-Suche im Radius) auf 4-m-Raster, harter Prominenz-Cutoff + räumliches NMS, grobe Formklasse (Kuppe / Grat / Plateau / dominanter Gipfel); Anzeige `▲ NNN m` |

Die Parameter (Linienabstand, Fett-Intervall, min. Prominenz, min. Abstand, Suchradius,
Arbeitsraster) stehen in der Importer-Box *Heightmap CSV*. Der Button **„Nur Höhenlinien /
Höhenpunkte neu berechnen"** erzeugt die zwei Dateien ohne neues Kacheln; danach Kartenpaket neu
exportieren und in C2 neu hochladen.

Für **mehr** Höhenpunkte: *min. Abstand* senken (größter Hebel, z. B. 250 → 130) und
*min. Prominenz* (z. B. 20 → 12), optional *Suchradius* senken (300 → 150).

## 9. Echtzeit-Kollaboration

WebSocket pro Plan unter `/plans/{plan_id}/live`. Der Server (`live.py`) prüft jede Operation gegen
die Rechte des Aufrufers und Marker-Locks:

- `marker.create`, `stroke.*` — Client rendert optimistisch mit temporärer ID, Server bestätigt mit
  der echten ID (`marker.upsert` inkl. `cid`) oder lehnt ab.
- `marker.move` / `marker.modify` / `marker.lock` / `marker.delete` — server-autoritativ, an den
  Plan-Raum gebroadcastet.
- `presence.cursor` — immer erlaubt (auch für Viewer: das „Zeigen"-Werkzeug).

## 10. Öffentliche Freigabe-Links

Ein Owner erzeugt einen Token; `#/p/<token>` öffnet den Plan **nur lesend, ohne Login** — Karte,
Marker, Zeichnungen, Orte, Höhenlinien/-punkte. Löschen des Tokens macht den Link ungültig.

## 11. Admin-Bereich

Nutzer, Gruppen, **Katalog-Upload** (5 Dateien, siehe 12), Karten (Import / Update / DLC / Löschen),
Audit-Log. UI zweisprachig **DE / EN**; Karten-Ortslabel in 13 Sprachen (unabhängig von der
UI-Sprache).

## 12. Katalog-Dateien

C2 nutzt **dieselben `LocalMapData`-Katalogdateien** wie der [SIDC-Framework](SIDC-Framework)-Mod.
Der Admin lädt sie hoch (roher JSON-Body, kein Multipart — bei großen Dateien ggf. das
`client_max_body_size` des Proxys erhöhen):

| Datei | Wofür |
|-------|-------|
| `SIDC_AllMarkersCatalog.json` | voller Marker-Katalog; das Feld `subCategory` verweist in den Modifier-Katalog |
| `SIDC_ModifierCatalog.json` | Modifikator-Optionen je Sub-Kategorie (die „Advanced"-Dropdowns) |
| `SIDC_ChannelSettings.json` | Funkkanäle |
| `SIDC_QuickMenuSettings.json` | Layout des Quick-Marker-Menüs |
| `SIDC_PhaseLineSettings.json` | Linienfarben / -breiten |

Nach einer Änderung an `SIDC_AllMarkersCatalog.json` (z. B. dem neuen `subCategory`-Feld) auch die
passende `SIDC_ModifierCatalog.json` neu hochladen.

## 13. Kartenpakete

C2 importiert exakt dieselben `*_mappack_v*.zip`-Pakete, die der **ATAKmaps-Importer** erzeugt
(`pipeline/map_packages.py`, Reiter *Export*). Aufbau:

```
mappack.json          Manifest (optional)
calibration.json      Ursprung + Skalierung je Achse (Spielmeter ↔ Fake-WGS-84-Grad)
mbtiles/sat.mbtiles   Pflicht
mbtiles/terrain.mbtiles   optional — 3D + Höhenanzeige (Terrarium RGB)
mbtiles/grid.mbtiles      optional — Koordinatengitter-Raster
mbtiles/dlc/<layer>_z<N>.mbtiles   optional — hochauflösende Zoomstufen
topo.geojson          optional — Straßen/Wege (Anzeige derzeit deaktiviert)
locations.json        optional — benannte Orte, nach baseType gruppiert
contours.geojson      optional — Höhenlinien (siehe 8)
peaks.geojson         optional — dominante Höhenpunkte (siehe 8)
```

Alle Rohdaten-Verarbeitung (`.topo` → GeoJSON, `mapLocations` → `locations.json`, Heightmap →
`contours`/`peaks`) passiert **einmalig** im ATAKmaps-Importer beim Paketbau. Import in C2 über den
Admin-Bereich (Download-Link oder Direkt-Upload).

### 13.1 Kartenserver für ATAKmaps (API-Tokens)

C2 kann als **Kartenquelle des ATAKmaps-Clients** dienen (siehe [ATAKmaps](ATAKmaps)): Spieler laden Karten aus C2, statt Pakete zu installieren. Marker, Live-Position und Kalibrierung bleiben auf dem Rechner des Spielers.

1. **Nutzer:** *Einstellungen (Zahnrad) → API-Tokens → Token erstellen* (Name, optionales Ablaufdatum). Token kopieren — er wird **einmalig** angezeigt, gespeichert wird nur der SHA‑256‑Hash. Tokens lassen sich jederzeit widerrufen (der Client verliert sofort den Zugriff).
2. **ATAKmaps:** Setup-GUI → Reiter *Kartenserver* → Name, Adresse (`https://…`), Token.

Regeln:
- Ein Token hat den Scope `maps:read` und funktioniert **nur auf den lesenden Karten-Routen**: `GET /api/maps`, `/api/maps/{id}/tiles/…`, `style.json`, `topo.geojson`, `locations.json`, `contours.geojson`, `peaks.geojson` (dazu `GET /api/client/ping` als Verbindungstest). Pläne, Marker, Zeichnungen, Nutzerdaten, Kartenverwaltung und Token-Verwaltung sind nur mit Cookie-Session erreichbar — ein Token kommt dort nie durch. Rate-Limit pro Token.
- Alle angemeldeten Nutzer dürfen alle Karten laden; es gibt keine Kartenrechte pro Nutzer.
- Der Client bekommt die Qualität, die C2 vorhält (importierte Basiskarte, optional DLC). Lokal installierte Karten/DLC haben Vorrang und können höhere Zoomstufen ergänzen.
- Ein Re-Import einer Karte in C2 ändert deren `imported_at`; Clients verwerfen dann ihren Kachel-Cache dieser Karte.

## 14. Verwandtes

- [ATAKmaps](ATAKmaps) — das spielverbundene Begleit-Tool, von dem C2 abgeleitet ist; erzeugt die
  Kartenpakete und Kataloge.
- [SIDC-Framework](SIDC-Framework) — der Mod, der die SIDC-Symbole, Kataloge und das QuickMenü
  definiert.
- [SIDC-Framework-Channels](SIDC-Framework-Channels) — das Kanalmodell, das für die Marker-Channels
  wiederverwendet wird.
- Vollständiges Handbuch im Repo: `docs/HANDBUCH.md` (deutsch). Änderungen: `CHANGELOG.md`.
