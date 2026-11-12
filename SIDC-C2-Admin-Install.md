# SIDC – C2 — Admin & install

**What is this page about?** Running C2: Docker deploy, `.env` settings, large map uploads, backups, the admin area, SIDC catalog files, map packages, the map server for ATAKmaps (API tokens) and the technical architecture. **For whom?** Server admins. Users and roles are explained on [Roles & visibility](SIDC-C2-Roles-Visibility); overview: [SIDC – C2](SIDC-C2-Command-Control).

## 1. Architecture

| Component | Tech |
|---|---|
| Backend | FastAPI (Python 3.12), SQLAlchemy 2.0, Alembic migrations |
| Database | PostgreSQL 17 |
| Realtime backplane | Redis Pub/Sub |
| Frontend | Vanilla TypeScript + Vite 6 + MapLibre‑GL 5 + **milsymbol@3**, built into the backend image |
| Auth | `pwdlib` argon2 hashes + signed HttpOnly session cookie (`itsdangerous`), optional OIDC via `authlib`, login rate limit |
| TLS / external access | **external** reverse proxy (e.g. Nginx Proxy Manager) — no proxy inside the stack |

The **Docker Compose stack** has the services `db`, `backup` (rotating `pg_dump`), `redis`, `backend` and `discord-bot` (Discord bridge, idle without `DISCORD_TOKEN`). The backend container is the single entry point and serves frontend, REST API and WebSocket on one port.

- **Schema:** on start the backend runs `alembic upgrade head`; an older database built with `create_all` is stamped to the baseline `0001_baseline` once. `create_all` + an automatic `ALTER TABLE ADD COLUMN` stay as a safety net.
- **Volumes:** `pgdata` (database), `maps` → `/data/maps` (imported map packages), `uploads` → `/data/uploads` (SIDC catalogs at `/data/uploads/catalog`, generated `secret_key`), `backups` (dumps), `botdata` (bot).
- **Realtime:** WebSocket per plan at `/plans/{plan_id}/live`. `backend/app/routers/live.py` checks every operation against the caller's effective rights, the per‑marker lock and the phase visibility (player / builder / Zeus):
  - `marker.create`, `stroke.*` — the client renders optimistically with a temp id, the server confirms with the real id (`marker.upsert` incl. `cid`) or rejects.
  - `marker.move` / `marker.modify` / `marker.lock` / `marker.delete` — server‑authoritative, broadcast to the plan room.
  - `presence.cursor` — always allowed (also for viewers: the *Point* tool).
  - Builder content is only delivered to mission‑builder sockets (and Zeus sockets for released parts). Losing a right closes open sockets; after a reconnect the client resyncs.

## 2. Install & deploy

```bash
cp deploy/.env.example .env
# edit .env: passwords, SECRET_KEY (or leave empty -> auto), FORWARDED_ALLOW_IPS, MAP_IMPORT_MAX_MB
docker compose up -d
```

Then, in your reverse proxy, add a proxy host pointing at `http://<docker-host>:8080` and
**enable WebSocket support**. First login uses `BOOTSTRAP_ADMIN_USER` / `BOOTSTRAP_ADMIN_PASSWORD`
from `.env` (idempotently created on every start; placeholder passwords are rejected).

Update an existing deployment (migrations run on start):

```bash
cd /opt/sidc-c2 && git pull && docker compose up -d --build
```

### `.env` settings

| Var | Meaning |
|-----|---------|
| `HTTP_PORT` | container port (default `8080`) |
| `FORWARDED_ALLOW_IPS` | IP(s) of the proxy that `X-Forwarded-*` is trusted from |
| `COOKIE_SECURE` | `auto` / `true` / `false` — Secure flag of the session cookie (production behind TLS: `true`) |
| `SECRET_KEY` | cookie signing key; empty ⇒ generated on first start |
| `POSTGRES_USER` / `_PASSWORD` / `_DB`, `DATABASE_URL` | database |
| `REDIS_URL` | WebSocket backplane |
| `BACKUP_KEEP` / `BACKUP_INTERVAL` | number of dumps to keep (default 14) / seconds between dumps (default 86400) |
| `BOOTSTRAP_ADMIN_USER` / `_PASSWORD` | local admin account |
| `DEFAULT_USER_GROUP` | put new users (local + first OIDC login) into this group automatically; empty = off |
| `PUBLIC_URL` | public address — notifications contain a link |
| `NOTIFY_ALLOW_PRIVATE_HOSTS` | `true` = allow LAN targets (self‑hosted ntfy/Gotify/Matrix); default only public addresses |
| `BRIDGE_TOKEN` | long random value for the Discord/Matrix bridge; empty = off |
| `DISCORD_TOKEN`, `DISCORD_OWNER_ID`, `BOT_PREFIX` | Discord bot in the stack; empty token = bot stays off |
| `SMTP_HOST` / `_PORT` / `_USER` / `_PASSWORD` / `_FROM` | e‑mail channel; without `SMTP_HOST` it is greyed out |
| `MAP_IMPORT_MAX_MB` | hard limit for map‑package uploads (default `2048`) |
| `ROAD_COLOR_MAIN` / `_SECONDARY` / `_PATH` | default road colours (users can override them per browser) |
| `AUDIT_LOG_RETENTION_DAYS` | audit entries older than N days are deleted daily (default 180, 0 = off) |
| `OIDC_ENABLED` + `OIDC_ISSUER` / `_CLIENT_ID` / `_CLIENT_SECRET` / `_REDIRECT_URL` | optional external login; local login stays active |

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

### Backup & restore

The `backup` container writes `pg_dump` dumps into the volume `backups` (it only rotates after a successful dump).

```bash
gunzip -c <dump>.sql.gz | docker compose exec -T db psql -U "$POSTGRES_USER" -d "$POSTGRES_DB"
# list dumps:  docker compose run --rm backup ls -lt /backups
```

## 3. Admin area

Sidebar **Administration**:

| Item | Content |
|---|---|
| **Users & groups** | create local users, role, *may create plans*, *Mission builder*, *Zeus*, active/inactive, password reset (min. 12 characters; revokes the user's API tokens), Discord user ID, delete; groups with members and the same checkboxes |
| **Log** | login attempts and all important actions, filterable |
| **Storage** | storage use per project (images, documents, number of plans), largest first |
| **Settings** | SIDC catalog upload, **maps** (from link, upload, update, DLC, delete), **repo sources** (Git / Gitea: list `.zip` / `.json` / `.xlsx` and import as map or catalog), **legal** (imprint and privacy policy as `.md`/`.txt`), **Discord bridge**, **Restart backend** (~5 s outage) |

The UI is bilingual **DE / EN**; map place labels support 13 languages (independent of the UI language).

## 4. Catalog files

C2 uses the **same `LocalMapData` catalog files** as the [SIDC-Framework](SIDC-Framework) mod (format unchanged). The admin uploads them under *Settings → SIDC catalog* (or imports them from a repo source); max. 8 MB per file, a large file may need the proxy's `client_max_body_size` raised.

| File | Used for |
|------|----------|
| `SIDC_AllMarkersCatalog.json` | full marker catalog; the `subCategory` field keys into the modifier catalog |
| `SIDC_QuickMarkerMenuCatalog.json` | quick‑menu layout |
| `SIDC_PhaseLineStyleCatalog.json` | line colours / widths |
| `SIDC_ModifierCatalog.json` | modifier options per sub‑category (the *Advanced* dropdowns) |
| `SIDC_Translations.xlsx` (game localisation export) | symbol names and hover descriptions DE/EN — converted to `SIDC_Translations.json` on upload |

After changing `SIDC_AllMarkersCatalog.json` (e.g. the `subCategory` field) also re‑upload the matching `SIDC_ModifierCatalog.json`. Without a marker catalog the wizard says "No marker catalog uploaded".

## 5. Map packages

C2 imports the exact same `*_mappack_v*.zip` packages produced by the **ATAKmaps importer**
(`pipeline/map_packages.py`, *Export* tab):

```
mappack.json                       manifest (optional)
calibration.json                   origin + per-axis scale (game metres <-> fake WGS-84 degrees)
mbtiles/sat.mbtiles                required
mbtiles/terrain.mbtiles            optional - 3D + elevation readout (Terrarium RGB)
mbtiles/grid.mbtiles               optional - coordinate grid raster
mbtiles/dlc/<layer>_z<N>.mbtiles   optional - high-res zoom levels
topo.geojson                       optional - roads/paths
locations.json                     optional - named places, grouped by baseType
contours.geojson / peaks.geojson   optional - contour lines / elevation points
buildings|roads|water|powerlines|trees.geojson   optional - Enfusion TopoExport layers
```

All raw‑data processing (`.topo` → GeoJSON, `mapLocations` → `locations.json`, heightmap →
`contours`/`peaks`) happens **once** in the ATAKmaps importer at pack‑build time; C2 only displays. Older
packs with a raw `mapLocations_locations.json` must be processed in the importer and re‑exported. How
contour lines/elevation points are computed: [Map & tools](SIDC-C2-Map-Tools).

Import in *Settings → Maps*: download link, direct upload or repo source (same map ID ⇒ the map is
replaced). *Update…* loads a new ZIP. The map cannot be changed in an existing plan.

## 6. Map server for ATAKmaps (API tokens)

C2 can act as the **map source of the ATAKmaps client** (see [ATAKmaps](ATAKmaps)): players load maps from C2 instead of installing packages. Markers, live position and calibration stay on the player's machine.

1. **User:** gear → *API tokens* → name (e.g. Laptop), optional expiry in days → *Create token*. Copy the token — it is shown **once**; only its SHA‑256 hash is stored. *Revoke* at any time (the client loses access immediately).
2. **ATAKmaps:** setup GUI → tab *Kartenserver* → name, address (`https://…`), token.

Rules:
- A token has the scope `maps:read` and works **only on the read‑only map routes**: `GET /api/maps`, `/api/maps/{id}/tiles/…`, `style.json`, `topo.geojson`, `locations.json`, `contours.geojson`, `peaks.geojson` (plus `GET /api/client/ping` as a connection test). Plans, markers, drawings, user data, map management and token management are cookie‑session only. Rate‑limited per token.
- All signed‑in users may load all maps; there are no per‑user map permissions.
- The client gets whatever quality C2 holds (imported base map, optional DLC). Locally installed maps/DLC take precedence and can add higher zoom levels.
- A re‑import of a map changes its `imported_at`, which makes clients drop their cached tiles of that map.

## 7. Development

```bash
# Backend
cd backend && python -m venv .venv && .venv/bin/pip install -r requirements.txt
.venv/bin/uvicorn app.main:app --reload --port 8080
# Frontend (second terminal)
cd frontend && npm install && npm run dev
# Tests
cd backend && pip install -r requirements-dev.txt && pytest -q
```

If the local repo path contains `&` (like "SIDC - C2 - Command & Control"), npm scripts break on Windows — use `node ./node_modules/vite/bin/vite.js build` instead of `npm run build`. The Docker build is not affected.

---

# SIDC – C2 — Admin & Installation

**Worum geht's?** Betrieb von C2: Docker-Deploy, `.env`-Werte, große Karten-Uploads, Backups, der Admin-Bereich, SIDC-Katalogdateien, Kartenpakete, der Kartenserver für ATAKmaps (API-Tokens) und die technische Architektur. **Für wen?** Server-Admins. Nutzer und Rollen erklärt [Rollen & Sichtbarkeit](SIDC-C2-Roles-Visibility); Überblick: [SIDC – C2](SIDC-C2-Command-Control).

## 1. Architektur

| Komponente | Technik |
|------------|---------|
| Backend | FastAPI (Python 3.12), SQLAlchemy 2.0, Alembic-Migrationen |
| Datenbank | PostgreSQL 17 |
| Realtime-Backplane | Redis Pub/Sub |
| Frontend | Vanilla-TypeScript + Vite 6 + MapLibre-GL 5 + **milsymbol@3**, in das Backend-Image gebaut |
| Auth | `pwdlib`-argon2-Hashes + signierter HttpOnly-Session-Cookie (`itsdangerous`), optional OIDC (`authlib`), Login-Ratelimit |
| TLS / externer Zugriff | **externer** Reverse Proxy (z. B. Nginx Proxy Manager) — kein Proxy im Stack |

Der **Docker-Compose-Stack** hat die Dienste `db`, `backup` (rotierender `pg_dump`), `redis`, `backend` und `discord-bot` (Discord-Brücke, ohne `DISCORD_TOKEN` untätig). Der Backend-Container ist der einzige Einstiegspunkt und liefert Frontend, REST-API und WebSocket auf einem Port.

- **Schema:** Beim Start läuft `alembic upgrade head`; eine ältere, per `create_all` gebaute Datenbank wird einmalig auf die Baseline `0001_baseline` gestampt. `create_all` + automatisches `ALTER TABLE ADD COLUMN` bleiben als Sicherheitsnetz.
- **Volumes:** `pgdata` (DB), `maps` → `/data/maps` (importierte Kartenpakete), `uploads` → `/data/uploads` (SIDC-Kataloge unter `/data/uploads/catalog`, generierter `secret_key`), `backups` (Dumps), `botdata` (Bot).
- **Realtime:** WebSocket pro Plan unter `/plans/{plan_id}/live`. `backend/app/routers/live.py` prüft jede Operation gegen die effektiven Rechte des Aufrufers, den Marker-Lock und die Phasen-Sichtbarkeit (Spieler / Missionsbau / Zeus):
  - `marker.create`, `stroke.*` — Client rendert optimistisch mit temporärer ID, Server bestätigt mit der echten ID (`marker.upsert` inkl. `cid`) oder lehnt ab.
  - `marker.move` / `marker.modify` / `marker.lock` / `marker.delete` — server-autoritativ, an den Plan-Raum gebroadcastet.
  - `presence.cursor` — immer erlaubt (auch für Betrachter: das Werkzeug *Zeigen*).
  - Missionsbau-Inhalte gehen nur an Missionsbau-Sockets (und für freigegebene Teile an Zeus-Sockets). Wer ein Recht verliert, verliert offene Sockets; nach einem Reconnect synchronisiert der Client neu.

## 2. Installation & Deploy

```bash
cp deploy/.env.example .env
# .env anpassen: Passwörter, SECRET_KEY (oder leer -> auto), FORWARDED_ALLOW_IPS, MAP_IMPORT_MAX_MB
docker compose up -d
```

Dann im Reverse Proxy einen Proxy-Host auf `http://<docker-host>:8080` anlegen und
**WebSocket-Unterstützung aktivieren**. Erst-Login mit `BOOTSTRAP_ADMIN_USER` /
`BOOTSTRAP_ADMIN_PASSWORD` aus der `.env` (bei jedem Start idempotent angelegt; Platzhalter-Passwörter werden abgelehnt).

Bestehendes Deployment aktualisieren (Migrationen laufen beim Start):

```bash
cd /opt/sidc-c2 && git pull && docker compose up -d --build
```

### `.env`-Werte

| Variable | Bedeutung |
|----------|-----------|
| `HTTP_PORT` | Container-Port (Default `8080`) |
| `FORWARDED_ALLOW_IPS` | IP(s) des Proxys, dem `X-Forwarded-*` geglaubt wird |
| `COOKIE_SECURE` | `auto` / `true` / `false` — Secure-Flag des Session-Cookies (Produktion hinter TLS: `true`) |
| `SECRET_KEY` | Signierschlüssel; leer ⇒ beim ersten Start erzeugt |
| `POSTGRES_USER` / `_PASSWORD` / `_DB`, `DATABASE_URL` | Datenbank |
| `REDIS_URL` | WebSocket-Backplane |
| `BACKUP_KEEP` / `BACKUP_INTERVAL` | wie viele Dumps behalten (Default 14) / Sekunden zwischen Dumps (Default 86400) |
| `BOOTSTRAP_ADMIN_USER` / `_PASSWORD` | lokales Admin-Konto |
| `DEFAULT_USER_GROUP` | neue Nutzer (lokal + OIDC-Erstlogin) automatisch dieser Gruppe zuordnen; leer = aus |
| `PUBLIC_URL` | öffentliche Adresse — Benachrichtigungen enthalten einen Link |
| `NOTIFY_ALLOW_PRIVATE_HOSTS` | `true` = LAN-Ziele erlauben (selbst gehostetes ntfy/Gotify/Matrix); Standard nur öffentliche Adressen |
| `BRIDGE_TOKEN` | langer Zufallswert für die Discord/Matrix-Brücke; leer = aus |
| `DISCORD_TOKEN`, `DISCORD_OWNER_ID`, `BOT_PREFIX` | Discord-Bot im Stack; leerer Token = Bot bleibt aus |
| `SMTP_HOST` / `_PORT` / `_USER` / `_PASSWORD` / `_FROM` | E-Mail-Kanal; ohne `SMTP_HOST` ausgegraut |
| `MAP_IMPORT_MAX_MB` | hartes Limit für Kartenpaket-Uploads (Default `2048`) |
| `ROAD_COLOR_MAIN` / `_SECONDARY` / `_PATH` | Standardfarben der Straßen (Nutzer können sie pro Browser überschreiben) |
| `AUDIT_LOG_RETENTION_DAYS` | Audit-Einträge älter als N Tage werden täglich gelöscht (Default 180, 0 = aus) |
| `OIDC_ENABLED` + `OIDC_ISSUER` / `_CLIENT_ID` / `_CLIENT_SECRET` / `_REDIRECT_URL` | optionaler externer Login; lokaler Login bleibt aktiv |

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

### Backup & Wiederherstellung

Der `backup`-Container legt `pg_dump`-Dumps im Volume `backups` ab (rotiert nur nach erfolgreichem Dump).

```bash
gunzip -c <dump>.sql.gz | docker compose exec -T db psql -U "$POSTGRES_USER" -d "$POSTGRES_DB"
# Dumps auflisten:  docker compose run --rm backup ls -lt /backups
```

## 3. Admin-Bereich

Sidebar **Administration**:

| Eintrag | Inhalt |
|---|---|
| **Benutzer & Gruppen** | lokale Nutzer anlegen, Rolle, *darf Pläne erstellen*, *Missionsbau*, *Zeus*, aktiv/deaktiviert, Passwort-Reset (min. 12 Zeichen; widerruft die API-Tokens des Nutzers), Discord-Nutzer-ID, löschen; Gruppen mit Mitgliedern und denselben Häkchen |
| **Log** | Login-Versuche und alle wichtigen Aktionen, filterbar |
| **Speicher** | Speicherverbrauch je Projekt (Bilder, Dokumente, Anzahl Pläne), größte zuerst |
| **Einstellungen** | SIDC-Katalog-Upload, **Karten** (per Link, Hochladen, Aktualisieren, DLC, Löschen), **Repo-Quellen** (Git / Gitea: `.zip` / `.json` / `.xlsx` auflisten und als Karte oder Katalog importieren), **Rechtliches** (Impressum und Datenschutz als `.md`/`.txt`), **Discord-Brücke**, **Backend neu starten** (~5 s Ausfall) |

Die UI ist zweisprachig **DE / EN**; Karten-Ortslabel gibt es in 13 Sprachen (unabhängig von der UI-Sprache).

## 4. Katalog-Dateien

C2 nutzt **dieselben `LocalMapData`-Katalogdateien** wie der [SIDC-Framework](SIDC-Framework)-Mod (Format unverändert). Der Admin lädt sie unter *Einstellungen → SIDC-Katalog* hoch (oder importiert sie aus einer Repo-Quelle); max. 8 MB je Datei, bei großen Dateien ggf. `client_max_body_size` im Proxy erhöhen.

| Datei | Wofür |
|-------|-------|
| `SIDC_AllMarkersCatalog.json` | voller Marker-Katalog; das Feld `subCategory` verweist in den Modifier-Katalog |
| `SIDC_QuickMarkerMenuCatalog.json` | Layout des QuickMenüs |
| `SIDC_PhaseLineStyleCatalog.json` | Linienfarben / -breiten |
| `SIDC_ModifierCatalog.json` | Modifikator-Optionen je Sub-Kategorie (die *Erweitert*-Dropdowns) |
| `SIDC_Translations.xlsx` (Lokalisierungs-Export des Spiels) | Symbolnamen und Hover-Beschreibungen DE/EN — wird beim Upload zu `SIDC_Translations.json` konvertiert |

Nach einer Änderung an `SIDC_AllMarkersCatalog.json` (z. B. dem Feld `subCategory`) auch die
passende `SIDC_ModifierCatalog.json` neu hochladen. Ohne Marker-Katalog meldet der Wizard „Kein Marker-Katalog hochgeladen".

## 5. Kartenpakete

C2 importiert exakt dieselben `*_mappack_v*.zip`-Pakete, die der **ATAKmaps-Importer** erzeugt
(`pipeline/map_packages.py`, Reiter *Export*):

```
mappack.json                       Manifest (optional)
calibration.json                   Ursprung + Skalierung je Achse (Spielmeter <-> Fake-WGS-84-Grad)
mbtiles/sat.mbtiles                Pflicht
mbtiles/terrain.mbtiles            optional - 3D + Höhenanzeige (Terrarium RGB)
mbtiles/grid.mbtiles               optional - Koordinatengitter-Raster
mbtiles/dlc/<layer>_z<N>.mbtiles   optional - hochauflösende Zoomstufen
topo.geojson                       optional - Straßen/Wege
locations.json                     optional - benannte Orte, nach baseType gruppiert
contours.geojson / peaks.geojson   optional - Höhenlinien / Höhenpunkte
buildings|roads|water|powerlines|trees.geojson   optional - Enfusion-TopoExport-Ebenen
```

Alle Rohdaten-Verarbeitung (`.topo` → GeoJSON, `mapLocations` → `locations.json`, Heightmap →
`contours`/`peaks`) passiert **einmalig** im ATAKmaps-Importer beim Paketbau; C2 zeigt nur an. Ältere
Packs mit roher `mapLocations_locations.json` müssen im Importer verarbeitet und neu exportiert werden. Wie
Höhenlinien/Höhenpunkte berechnet werden: [Karte & Werkzeuge](SIDC-C2-Map-Tools).

Import unter *Einstellungen → Karten*: Download-Link, Direkt-Upload oder Repo-Quelle (gleiche Karten-ID ⇒ die
Karte wird ersetzt). *Aktualisieren…* lädt eine neue ZIP. In einem bestehenden Plan lässt sich die Karte nicht wechseln.

## 6. Kartenserver für ATAKmaps (API-Tokens)

C2 kann als **Kartenquelle des ATAKmaps-Clients** dienen (siehe [ATAKmaps](ATAKmaps)): Spieler laden Karten aus C2, statt Pakete zu installieren. Marker, Live-Position und Kalibrierung bleiben auf dem Rechner des Spielers.

1. **Nutzer:** Zahnrad → *API-Tokens* → Name (z. B. Laptop), optional Ablauf in Tagen → *Token erstellen*. Token kopieren — er wird **einmalig** angezeigt, gespeichert wird nur der SHA‑256‑Hash. *Widerrufen* jederzeit (der Client verliert sofort den Zugriff).
2. **ATAKmaps:** Setup-GUI → Reiter *Kartenserver* → Name, Adresse (`https://…`), Token.

Regeln:
- Ein Token hat den Scope `maps:read` und funktioniert **nur auf den lesenden Karten-Routen**: `GET /api/maps`, `/api/maps/{id}/tiles/…`, `style.json`, `topo.geojson`, `locations.json`, `contours.geojson`, `peaks.geojson` (dazu `GET /api/client/ping` als Verbindungstest). Pläne, Marker, Zeichnungen, Nutzerdaten, Kartenverwaltung und Token-Verwaltung sind nur mit Cookie-Session erreichbar. Rate-Limit pro Token.
- Alle angemeldeten Nutzer dürfen alle Karten laden; es gibt keine Kartenrechte pro Nutzer.
- Der Client bekommt die Qualität, die C2 vorhält (importierte Basiskarte, optional DLC). Lokal installierte Karten/DLC haben Vorrang und können höhere Zoomstufen ergänzen.
- Ein Re-Import einer Karte ändert deren `imported_at`; Clients verwerfen dann ihren Kachel-Cache dieser Karte.

## 7. Entwicklung

```bash
# Backend
cd backend && python -m venv .venv && .venv/bin/pip install -r requirements.txt
.venv/bin/uvicorn app.main:app --reload --port 8080
# Frontend (zweites Terminal)
cd frontend && npm install && npm run dev
# Tests
cd backend && pip install -r requirements-dev.txt && pytest -q
```

Enthält der lokale Repo-Pfad ein `&` (wie „SIDC - C2 - Command & Control"), brechen npm-Scripts unter Windows — dann `node ./node_modules/vite/bin/vite.js build` statt `npm run build`. Der Docker-Build ist nicht betroffen.
