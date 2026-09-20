# ATAKmaps (companion tool)

> Source: `D:\Mods\ATAKmaps` (dev repo) · pre‑built client `D:\Mods\ATAKmapsClient\{Win,Linux,Server}`. Verified against `README.md`, `Doku/SIDC-Data-Interface.md`, `start.py`, `build_client.py`, `server/app.py`.

## 1. What it is

ATAKmaps is **not a Reforger mod** — it is an external **web map** (local web server + browser frontend) for Arma Reforger. It shows the **live player position** and the **SIDC markers** placed in game, by reading the file‑based data that the **[SIDC-Framework](SIDC-Framework)** mod writes to disk. It can also send marker/channel requests *back* to the mod.

- Local server (FastAPI / Python, `http://localhost:8765`) serving satellite / terrain / grid map tiles (MapLibre GL frontend).
- Reads 4 JSON files from `$profile:SIDC_Framework/LocalMapData/`, writes request files into `LocalMapData/Request/`.
- Purely file‑based — **no network protocol on the mod side**. The tool just needs read/write access to the `$profile:` folder of the machine the mod runs on (a player's client, a listen server, or a dedicated server).
- Map / DLC data (mbtiles, several GB) is distributed as `.zip` packages via **GitHub Releases**, imported through the Setup GUI — not shipped in the repo.

### Distributions

| Variant | For whom | Contents |
|---------|----------|----------|
| **Repo (dev)** — `D:\Mods\ATAKmaps` | Map creation / development | Server, web frontend, import pipeline (satellite / heightmap / grid), export GUI |
| **Client** — `D:\Mods\ATAKmapsClient\Win` (also `Linux`, `Server`) | Players | Server + Setup GUI as EXEs — **no Python needed** |

## 2. Requirements

- The **SIDC-Framework** mod installed and active in Reforger — it writes the JSON files to `$profile:SIDC_Framework/LocalMapData/`.
- For markers: either a client using the manual *Export* toggle in the SIDC settings menu, **or** a server with `serverExportMarkers` enabled (see [Server & Admin Guide](Server-Admin-Guide)).
- At least one map package imported.

## 3. Using the pre‑built Windows client

Folder `D:\Mods\ATAKmapsClient\Win`:

| Step | Launcher | Purpose |
|------|----------|---------|
| 1st run | `3 - Setup starten.bat` (`ATAKmaps-Setup.exe`) | Tab **"SIDC-Profile"**: pick the `SIDC_Framework` folder of your Reforger install (the one containing `LocalMapData`). Tab **"Karten & DLC"**: import at least one map package `.zip` (from GitHub Releases). |
| every session | `1 - Web Server starten.bat` (`ATAKmaps-Server.exe`) | Start the server — leave the window open. |
| every session | `2 - Browser starten.bat` | Opens `http://localhost:8765`. |

The client mode is chosen by the EXE name (`start_client._mode()`); the same PyInstaller spec (`client/atakmaps_client.spec`) builds the Linux variant.

### Map server (optional — load maps from SIDC – C2)

Instead of importing map packages yourself, the client can load maps from a **[SIDC – C2 – Command & Control](SIDC-C2-Command-Control)** server. Setup GUI → tab **"Kartenserver"**: tick *use map server*, enter a name, the address (`https://…`) and an **API token** (created in C2 under *Settings → API tokens*), then *Speichern & verbinden*.

- The token grants **read access to map data only**. C2 plans are not reachable, and ATAKmaps only ever sends `GET` requests — SIDC markers, live position and calibration stay local.
- Maps that exist only on the server show a **☁** in the map selector.
- **Local wins.** If a map is installed locally *and* offered by the server, the server only fills in what is missing locally — e.g. server = base map (up to zoom 18), local DLC packages = zoom 19+. With the map server off, everything runs purely local.
- Tiles and extras (topo, locations, contours, peaks) are fetched on demand and cached in `data/remote/`; cached data keeps working offline. A re-import on the server invalidates that map's cache. The server's calibration is used only while you have none of your own.
- The token is stored in the Windows credential store (fallback: `data/remote.json`). Plain `http://` is only accepted for local addresses.

## 4. Dev setup (repo)

```
python -m venv .venv && .venv\Scripts\activate
pip install -r requirements.txt
cd web && npm install && npm run build && cd ..

python start.py            # server  -> http://localhost:8765
python start.py setup      # Setup GUI (SIDC profile / maps & DLC)
python start.py gui        # Import GUI (satellite / heightmap / grid + Export tab)
python start.py sat "<image folder>"   # satellite import (CLI)
python start.py hm  "<csv path>"       # heightmap import (CLI)
```

Data directory: `<repo>/data` or the `ATAKMAPS_DATA` env var.

### Map packages

- **Base map**: Import GUI → *Export (DEV)* tab → *export map package* → `data/packages/<id>_mappack_v<n>.zip`
- **DLC** (high‑res zoom z19+): same tab, one `<id>_<layer>_z<N>.zip` per level. The server only uses DLC zoom levels **contiguously** from z19 — the GUI warns about gaps.
- Import / delete: Setup GUI → *Karten & DLC* tab.
- Package format: `pipeline/map_packages.py`.

### Build the client

```
pip install -r requirements.txt          # includes pyinstaller
cd web && npm run build && cd ..
python build_client.py                   # -> D:\Mods\ATAKmapsClient\Win\
```

## 5. Server API (localhost:8765)

| Route | Purpose |
|-------|---------|
| `GET /tiles/{map_id}/sat|terrain|grid/{z}/{x}/{y}` | Map tiles |
| `GET /api/maps` · `POST /api/maps` · `POST /api/maps/active` | List / add / switch active map (list includes `source`: `local` \| `remote` \| `both`) |
| `GET /api/remote` · `POST /api/remote/settings` · `/sync` · `/disconnect` · `/clear-cache` | Map server (SIDC – C2) settings / sync / reset — see *Map server* |
| `GET /api/meta` · `POST /api/config` | Map metadata / config |
| `GET /api/sidc/profile` | Active SIDC profile (folder) |
| `GET /api/sidc/catalog` | Marker catalog (`SIDC_AllMarkersCatalog.json`) |
| `GET /api/sidc/stream` | SSE live stream (markers + player position) |
| `POST /api/sidc/marker/create` · `/move` · `/delete` | Writes `CreateMarker-*.json` / `MoveMarker-*.json` / `DelMarker-*.json` request files |
| `POST /api/sidc/channel` | Writes `SetChannel-*.json` (`{channel?, physicalChannel?}`, at least one) |
| `POST /api/sidc/marker/modify` | Writes `ModifyMarker-*.json` — **not implemented mod‑side**, silently ignored |

`server/sidc_watcher.py` polls `LocalMapData/` (~200 ms default) and pushes changes over the SSE stream. `data/sidc_profiles.json` maps profile names → folders.

## 6. Data interface (mod ↔ tool)

Full spec: `D:\Mods\ATAKmaps\Doku\SIDC-Data-Interface.md`. Folder `$profile:SIDC_Framework/LocalMapData/`:

| File | Direction | Written by | Update trigger |
|------|-----------|-----------|----------------|
| `SIDC_PlacedMarkers.json` | mod → tool | mod (client manual export **or** server periodic) | dirty‑check signature; written only on change |
| `SIDC_PlayerData.json` | mod → tool | mod, **client‑side only** (`SIDC_FrameworkBase.TickPlayerData()`) | every tick (`playerSyncIntervalMs`, default 50 ms); empty on a dedicated server |
| `SIDC_ChannelSettings.json` | mod → tool | mod, on every `SIDC_PlayerProfile.Save()` | channel / override change |
| `SIDC_AllMarkersCatalog.json` | mod → tool | mod, once per GameMode start | never at runtime |
| `Request/*.json` | tool → mod | this tool | on demand; the mod deletes the file after reading — no return channel |

Key facts:

- `SIDC_PlacedMarkers.json` is **unfiltered** — every faction, every channel, every author. There is **no server‑side faction/channel filter** in the mod. Faction‑specific deployments must filter on `ownerFaction` in `sidc_watcher.py` or the web frontend.
- Markers use **2D** `worldX`/`worldY`; player data uses **real 3D** (`worldY` = height). Don't mix them up.
- `channel` (SIDC visibility category, e.g. "Alpha", "Air", "Command") is **not** the real game faction — that's the separate `ownerFaction` field.
- Both `channel` and `physicalChannel` in `SIDC_PlayerData.json` are never empty since 2026‑08‑30 (config‑default fallback). `physicalChannel` filters markers binary (visible/hidden); the `All` physical channel sees everything.
- Requests carry no `playerId` — the caller is always the local player who owns that `$profile:` folder.

## 7. Related

- [SIDC-Framework](SIDC-Framework) — the mod that produces the data.
- Mod‑side exporters: `SIDC_MarkerExporter.c`, `SIDC_MarkerRequestProcessor.c`, `SIDC_PlayerDataExporter.c`, `SIDC_ChannelSettingsExporter.c`.
- TypeScript field mirror: `web/src/sidc-marker/types.ts`.

---

# ATAKmaps (Begleit-Tool) — Deutsch

> Quelle: `D:\Mods\ATAKmaps` (Dev-Repo) · vorgebauter Client `D:\Mods\ATAKmapsClient\{Win,Linux,Server}`. Verifiziert gegen `README.md`, `Doku/SIDC-Data-Interface.md`, `start.py`, `build_client.py`, `server/app.py`.

## 1. Kurzbeschreibung

ATAKmaps ist **kein Reforger-Mod** — es ist eine externe **Web-Karte** (lokaler Webserver + Browser-Frontend) für Arma Reforger. Sie zeigt die **Live-Spielerposition** und die im Spiel gesetzten **SIDC-Marker**, indem sie die dateibasierten Daten liest, die der **[SIDC-Framework](SIDC-Framework)**-Mod auf die Platte schreibt. Sie kann außerdem Marker-/Kanal-Requests *zurück* an den Mod schicken.

- Lokaler Server (FastAPI / Python, `http://localhost:8765`), liefert Satelliten-/Höhen-/Grid-Kartenkacheln (MapLibre-GL-Frontend).
- Liest 4 JSON-Dateien aus `$profile:SIDC_Framework/LocalMapData/`, schreibt Request-Dateien nach `LocalMapData/Request/`.
- Rein dateibasiert — **kein Netzwerkprotokoll auf Mod-Seite**. Das Tool braucht nur Lese-/Schreibzugriff auf den `$profile:`-Ordner der Maschine, auf der der Mod läuft (Client, Listen-Server oder dedizierter Server).
- Karten-/DLC-Daten (mbtiles, mehrere GB) werden als `.zip`-Pakete über **GitHub-Releases** verteilt und über die Setup-GUI importiert — nicht im Repo.

### Varianten

| Variante | Für wen | Enthält |
|----------|---------|---------|
| **Repo (dev)** — `D:\Mods\ATAKmaps` | Kartenerstellung / Entwicklung | Server, Web-Frontend, Import-Pipeline (Sat / Heightmap / Grid), Export-GUI |
| **Client** — `D:\Mods\ATAKmapsClient\Win` (auch `Linux`, `Server`) | Spieler | Server + Setup-GUI als EXE — **kein Python nötig** |

## 2. Voraussetzungen

- Der **SIDC-Framework**-Mod in Reforger installiert und aktiv — er schreibt die JSON-Dateien nach `$profile:SIDC_Framework/LocalMapData/`.
- Für Marker: entweder ein Client mit dem manuellen *Export*-Toggle im SIDC-Einstellungsmenü, **oder** ein Server mit aktivem `serverExportMarkers` (siehe [Server- & Admin-Handbuch](Server-Admin-Guide)).
- Mindestens ein Kartenpaket importiert.

## 3. Vorgebauter Windows-Client

Ordner `D:\Mods\ATAKmapsClient\Win`:

| Schritt | Launcher | Zweck |
|---------|----------|-------|
| Erststart | `3 - Setup starten.bat` (`ATAKmaps-Setup.exe`) | Reiter **„SIDC-Profile"**: den `SIDC_Framework`-Ordner der Reforger-Installation wählen (enthält `LocalMapData`). Reiter **„Karten & DLC"**: mindestens ein Kartenpaket `.zip` importieren (aus GitHub-Releases). |
| jede Session | `1 - Web Server starten.bat` (`ATAKmaps-Server.exe`) | Server starten — Fenster offen lassen. |
| jede Session | `2 - Browser starten.bat` | Öffnet `http://localhost:8765`. |

Der Client-Modus wird über den EXE-Namen gewählt (`start_client._mode()`); dieselbe PyInstaller-Spec (`client/atakmaps_client.spec`) baut die Linux-Variante.

### Kartenserver (optional — Karten von SIDC – C2 laden)

Statt Kartenpakete selbst zu importieren, kann der Client Karten von einem **[SIDC – C2 – Command & Control](SIDC-C2-Command-Control)**-Server laden. Setup-GUI → Reiter **„Kartenserver"**: „Kartenserver nutzen" anhaken, Name, Adresse (`https://…`) und **API-Token** (in C2 unter *Einstellungen → API-Tokens* erstellt) eintragen, dann *Speichern & verbinden*.

- Der Token gibt **nur Lesezugriff auf Kartendaten**. Pläne aus C2 sind nicht erreichbar, und ATAKmaps sendet ausschließlich `GET`-Anfragen — SIDC-Marker, Live-Position und Kalibrierung bleiben lokal.
- Karten, die es nur auf dem Server gibt, zeigen in der Kartenauswahl ein **☁**.
- **Lokal hat Vorrang.** Ist eine Karte lokal installiert *und* auf dem Server, füllt der Server nur auf, was lokal fehlt — z. B. Server = Basiskarte (bis Zoom 18), lokale DLC-Pakete = Zoom 19+. Mit ausgeschaltetem Kartenserver läuft alles rein lokal.
- Kacheln und Zusatzdaten (topo, locations, contours, peaks) werden bei Bedarf geladen und in `data/remote/` gecacht; Gecachtes funktioniert offline weiter. Ein Re-Import auf dem Server verwirft den Cache dieser Karte. Die Kalibrierung des Servers gilt nur, solange man keine eigene hat.
- Der Token liegt im Windows-Anmeldeinformationsspeicher (Fallback: `data/remote.json`). Unverschlüsseltes `http://` wird nur für lokale Adressen akzeptiert.

## 4. Dev-Setup (Repo)

```
python -m venv .venv && .venv\Scripts\activate
pip install -r requirements.txt
cd web && npm install && npm run build && cd ..

python start.py            # Server  -> http://localhost:8765
python start.py setup      # Setup-GUI (SIDC-Profil / Karten & DLC)
python start.py gui        # Import-GUI (Sat / Heightmap / Grid + Export-Reiter)
python start.py sat "<Bilder-Ordner>"   # Sat-Import (CLI)
python start.py hm  "<CSV-Pfad>"        # Heightmap-Import (CLI)
```

Daten-Verzeichnis: `<repo>/data` bzw. Umgebungsvariable `ATAKMAPS_DATA`.

### Kartenpakete

- **Basiskarte**: Import-GUI → Reiter *Export (DEV)* → *Kartenpaket exportieren* → `data/packages/<id>_mappack_v<n>.zip`
- **DLC** (hochauflösende Zoomstufen z19+): gleicher Reiter, je Stufe ein `<id>_<layer>_z<N>.zip`. Der Server nutzt DLC-Zoomstufen nur **lückenlos** ab z19 — die GUI warnt bei Lücken.
- Import / Löschen: Setup-GUI → Reiter *Karten & DLC*.
- Paketformat: `pipeline/map_packages.py`.

### Client bauen

```
pip install -r requirements.txt          # enthält pyinstaller
cd web && npm run build && cd ..
python build_client.py                   # -> D:\Mods\ATAKmapsClient\Win\
```

## 5. Server-API (localhost:8765)

| Route | Zweck |
|-------|-------|
| `GET /tiles/{map_id}/sat|terrain|grid/{z}/{x}/{y}` | Kartenkacheln |
| `GET /api/maps` · `POST /api/maps` · `POST /api/maps/active` | Karten auflisten / hinzufügen / aktive wechseln (Liste enthält `source`: `local` \| `remote` \| `both`) |
| `GET /api/remote` · `POST /api/remote/settings` · `/sync` · `/disconnect` · `/clear-cache` | Kartenserver (SIDC – C2): Einstellungen / Abgleich / Zurücksetzen — siehe *Kartenserver* |
| `GET /api/meta` · `POST /api/config` | Karten-Metadaten / Config |
| `GET /api/sidc/profile` | Aktives SIDC-Profil (Ordner) |
| `GET /api/sidc/catalog` | Marker-Katalog (`SIDC_AllMarkersCatalog.json`) |
| `GET /api/sidc/stream` | SSE-Live-Stream (Marker + Spielerposition) |
| `POST /api/sidc/marker/create` · `/move` · `/delete` | Schreibt `CreateMarker-*.json` / `MoveMarker-*.json` / `DelMarker-*.json` |
| `POST /api/sidc/channel` | Schreibt `SetChannel-*.json` (`{channel?, physicalChannel?}`, mind. eines) |
| `POST /api/sidc/marker/modify` | Schreibt `ModifyMarker-*.json` — **mod-seitig nicht implementiert**, wird still ignoriert |

`server/sidc_watcher.py` pollt `LocalMapData/` (~200 ms Default) und schickt Änderungen über den SSE-Stream. `data/sidc_profiles.json` bildet Profilnamen → Ordner ab.

## 6. Datenschnittstelle (Mod ↔ Tool)

Vollständige Spezifikation: `D:\Mods\ATAKmaps\Doku\SIDC-Data-Interface.md`. Ordner `$profile:SIDC_Framework/LocalMapData/`:

| Datei | Richtung | Geschrieben von | Update-Trigger |
|-------|----------|-----------------|----------------|
| `SIDC_PlacedMarkers.json` | Mod → Tool | Mod (client-manuell **oder** server-periodisch) | Dirty-Check-Signatur; nur bei Änderung |
| `SIDC_PlayerData.json` | Mod → Tool | Mod, **nur clientseitig** (`SIDC_FrameworkBase.TickPlayerData()`) | jeden Tick (`playerSyncIntervalMs`, Default 50 ms); auf dediziertem Server leer |
| `SIDC_ChannelSettings.json` | Mod → Tool | Mod, bei jedem `SIDC_PlayerProfile.Save()` | Kanal-/Override-Änderung |
| `SIDC_AllMarkersCatalog.json` | Mod → Tool | Mod, einmalig pro GameMode-Start | nie zur Laufzeit |
| `Request/*.json` | Tool → Mod | dieses Tool | on-demand; der Mod löscht die Datei nach dem Lesen — kein Rückkanal |

Wichtige Fakten:

- `SIDC_PlacedMarkers.json` ist **ungefiltert** — jede Fraktion, jeder Kanal, jeder Ersteller. Es gibt **keinen serverseitigen Fraktions-/Kanal-Filter** im Mod. Fraktionsspezifische Deployments müssen auf `ownerFaction` in `sidc_watcher.py` oder im Web-Frontend filtern.
- Marker nutzen **2D** `worldX`/`worldY`; Spielerdaten nutzen **echtes 3D** (`worldY` = Höhe). Nicht verwechseln.
- `channel` (SIDC-Sichtbarkeitskategorie, z. B. „Alpha", „Air", „Command") ist **nicht** die echte Spiel-Fraktion — das ist das separate Feld `ownerFaction`.
- `channel` und `physicalChannel` in `SIDC_PlayerData.json` sind seit 2026-08-30 nie leer (Config-Default-Fallback). `physicalChannel` filtert Marker binär (sichtbar/unsichtbar); der physische Kanal `All` sieht alles.
- Requests haben kein `playerId` — der Aufrufer ist immer der lokale Spieler, dem der `$profile:`-Ordner gehört.

## 7. Verwandtes

- [SIDC-Framework](SIDC-Framework) — der Mod, der die Daten erzeugt.
- Mod-seitige Exporter: `SIDC_MarkerExporter.c`, `SIDC_MarkerRequestProcessor.c`, `SIDC_PlayerDataExporter.c`, `SIDC_ChannelSettingsExporter.c`.
- TypeScript-Feldspiegel: `web/src/sidc-marker/types.ts`.
