# SIDC – C2 – Command & Control (companion tool)

**What is this page?** The entry point to the C2 documentation: what the tool is, the three user groups in one minute, and where to find everything else. **For whom?** Everyone — players, mission builders, Zeus and server admins.

> Source: repo `github.com/GerSkyking/SIDC---C2---Command-Control` (AGPL-3.0) · image `ghcr.io/gerskyking/sidc---c2---command-control`. Verified against `README.md`, `docs/HANDBUCH.md`, `docs/FUNKTIONEN.md`, `CHANGELOG.md`, `deploy/.env.example` and the frontend/backend code (UI labels from `frontend/src/i18n.ts`). **Version 1.0.0** (2026-09-07), features as of 2026-11-12.

## 1. What it is

**SIDC – C2** is a browser‑based, **multi‑user operational planning tool**. It reuses the map and
military‑symbol logic of **[ATAKmaps](ATAKmaps)** / the **[SIDC-Framework](SIDC-Framework)** mod,
but has **no connection to a running game** — nothing is read from or written to Reforger. Users
log in, pick a map, and place SIDC markers and drawings **together in real time**.

Think of it as: the ATAKmaps web map, turned into a collaborative "sand table" that runs on a
server, with accounts, permissions, plans, folders, phases and version history — plus a
**mission‑building** area (projects, files, tasks, whiteboard, feedback), an **ORBAT** library and a **chat**.

| | ATAKmaps | SIDC – C2 |
|---|---|---|
| Data source | live game files (`$profile:.../LocalMapData/`) | its own database, no game link |
| Users | one local player | many, with accounts & roles |
| Runs | on the player's / server's machine | as a Docker stack on a Linux server |
| Persistence | the game | PostgreSQL (plans, markers, versions, projects) |
| Realtime | SSE from a file watcher | WebSocket, authoritative server |
| Shared with ATAKmaps | map packages, milsymbol rendering, SIDC catalogs, wizard/quick‑menu, grid | |

C2 can also act as the **map server** for the local ATAKmaps client (see [Admin & install → map server](SIDC-C2-Admin-Install)).

## 2. Three groups in one minute

| Group | Does | Sees |
|---|---|---|
| **Players** (default) | plan freely on the map: markers, lines, text boxes, phases | everything for players + what mission building releases to them |
| **Mission builder** | sets the guidelines, plans long‑term (projects, files, tasks, builder phases), hands work to individuals | **everything** |
| **Zeus** | runs the mission in game | player content (read‑only) + the parts of mission building released to Zeus (default **72 h**) |

The admin sets the groups (checkboxes **Mission builder** / **Zeus** on a user or group). Details, plan permissions and
"who sees what": **[Roles & visibility](SIDC-C2-Roles-Visibility)**.

## 3. Key terms

| Term | Meaning |
|---|---|
| **Plan** | one map with markers, lines, text boxes, images — edited together in real time |
| **Phase** | a time slice on the timeline. Every element belongs to a phase. **Player phases** (upper lane) and **builder phases** (lower lane, mission builders only) |
| **Layer** | a visibility group you can hide/dim: automatic by identity (BLUFOR/REDFOR/…), custom layers per plan and — in project plans — **Builder notes & stars** |
| **Map** | the base map itself (satellite, grid, contour lines, places …) — menu *Map* |
| **Project** | the bracket around one mission in *Mission building*: files, plans, tasks, whiteboard, feedback |
| **Star** | a marked spot in a plan that points to something in the project (task, sticky note, file …). **Yellow** = everyone, **pink** = mission builders only |

## 4. Pages

| Page | What you find there |
|---|---|
| [Getting started](SIDC-C2-Getting-Started) | first steps for players, mission builders and admins |
| [Roles & visibility](SIDC-C2-Roles-Visibility) | groups, plan & project permissions, who sees what, releases, public links |
| [Workflows (step by step)](SIDC-C2-Workflows) | plan & share, lines and named areas, phases → briefing/presentation, build a mission with stars, release to Zeus, hand work to individuals, search, versions |
| [Map & tools](SIDC-C2-Map-Tools) | toolbar, keys, top bar, timeline, marker wizard, layers, contour lines, search, settings |
| [Projects (mission building)](SIDC-C2-Projects) | directory, files & templates, timeline, tasks, milestones, feedback, whiteboard, link codes |
| [ORBAT](SIDC-C2-ORBAT) | force library, strength & losses, releasing enemy forces to players, briefing page |
| [Chat & notifications](SIDC-C2-Chat-Notifications) | chat, System feed, Discord DM / ntfy / Matrix / Web push / e‑mail, Discord bridge |
| [Admin & install](SIDC-C2-Admin-Install) | Docker deploy, `.env`, map packages, catalog files, map server / API tokens, admin area, architecture |

## 5. Related

- [ATAKmaps](ATAKmaps) — the game‑connected companion tool C2 is derived from; produces the map
  packages and catalogs.
- [SIDC-Framework](SIDC-Framework) — the mod that defines the SIDC symbols, catalogs and quick menu.
- [SIDC-Framework-Channels](SIDC-Framework-Channels) — the in‑game channel model (C2 itself replaced marker channels by layers in 2026‑10).
- Full user manual in the repo: `docs/HANDBUCH.md` (German). Changelog: `CHANGELOG.md`.

---

# SIDC – C2 – Command & Control (Begleit-Tool) — Deutsch

**Was ist diese Seite?** Der Einstieg in die C2-Doku: was das Tool ist, die drei Nutzergruppen in einer Minute und wo alles Weitere steht. **Für wen?** Für alle — Spieler, Missionsbau, Zeus und Server-Admins.

> Quelle: Repo `github.com/GerSkyking/SIDC---C2---Command-Control` (AGPL-3.0) · Image `ghcr.io/gerskyking/sidc---c2---command-control`. Verifiziert gegen `README.md`, `docs/HANDBUCH.md`, `docs/FUNKTIONEN.md`, `CHANGELOG.md`, `deploy/.env.example` und den Frontend-/Backend-Code (UI-Beschriftungen aus `frontend/src/i18n.ts`). **Version 1.0.0** (2026-09-07), Funktionsstand 2026-11-12.

## 1. Kurzbeschreibung

**SIDC – C2** ist ein browser­basiertes, **mehrbenutzerfähiges Einsatz-Planungstool**. Es nutzt die
Karten- und Militärsymbol-Logik von **[ATAKmaps](ATAKmaps)** bzw. dem
**[SIDC-Framework](SIDC-Framework)**-Mod, hat aber **keine Verbindung zu einem laufenden Spiel** —
es wird nichts aus Reforger gelesen oder geschrieben. Nutzer melden sich an, wählen eine Karte und
setzen **gemeinsam in Echtzeit** SIDC-Marker und Zeichnungen.

Kurz: die ATAKmaps-Web-Karte, umgebaut zu einem kollaborativen „Sandkasten" auf einem Server — mit
Konten, Rechten, Plänen, Ordnern, Phasen und Versionsverlauf — dazu ein Bereich **Missionsbau**
(Projekte, Dateien, Aufgaben, Whiteboard, Feedback), eine **ORBAT**-Bibliothek und ein **Chat**.

| | ATAKmaps | SIDC – C2 |
|---|---|---|
| Datenquelle | Live-Spieldateien (`$profile:.../LocalMapData/`) | eigene Datenbank, keine Spielanbindung |
| Nutzer | ein lokaler Spieler | viele, mit Konten & Rollen |
| Läuft | auf der Maschine des Spielers/Servers | als Docker-Stack auf einem Linux-Server |
| Persistenz | das Spiel | PostgreSQL (Pläne, Marker, Versionen, Projekte) |
| Echtzeit | SSE aus einem Datei-Watcher | WebSocket, autoritativer Server |
| Gemeinsam mit ATAKmaps | Kartenpakete, milsymbol-Rendering, SIDC-Kataloge, Wizard/QuickMenü, Grid | |

C2 kann außerdem als **Kartenserver** für den lokalen ATAKmaps-Client dienen (siehe [Admin & Installation → Kartenserver](SIDC-C2-Admin-Install)).

## 2. Drei Gruppen in einer Minute

| Gruppe | Macht | Sieht |
|---|---|---|
| **Spieler** (Standard) | planen frei auf der Karte: Marker, Linien, Textfelder, Phasen | alles für Spieler + was der Missionsbau ihnen freigibt |
| **Missionsbau** | macht die Vorgaben, plant langfristig (Projekte, Dateien, Aufgaben, Missionsbau-Phasen), lagert Arbeit an Einzelne aus | **alles** |
| **Zeus** | leitet die Mission im Spiel | Spieler-Inhalt (nur ansehen) + für Zeus freigegebene Teile des Missionsbaus (Standard **72 h**) |

Die Gruppen setzt der Admin (Häkchen **Missionsbau** / **Zeus** an Nutzer oder Gruppe). Details, Plan-Rechte und
„wer sieht was": **[Rollen & Sichtbarkeit](SIDC-C2-Roles-Visibility)**.

## 3. Begriffe

| Begriff | Bedeutung |
|---|---|
| **Plan** | eine Karte mit Markern, Linien, Textfeldern, Bildern — gemeinsam in Echtzeit bearbeitet |
| **Phase** | ein Zeitabschnitt im Zeitstrahl. Jedes Element gehört zu einer Phase. **Spieler-Phasen** (obere Spur) und **Missionsbau-Phasen** (untere Spur, nur Missionsbau) |
| **Layer** | eine Sichtbarkeits-Gruppe zum Aus-/Abblenden: automatisch nach Identität (BLUFOR/REDFOR/…), eigene Layer pro Plan und — im Projekt-Plan — **Missionsbau-Notizen & Sterne** |
| **Karte** | die Grundkarte selbst (Satellit, Gitter, Höhenlinien, Orte …) — Menü *Karte* |
| **Projekt** | die Klammer um eine Mission im Bereich *Missionsbau*: Dateien, Pläne, Aufgaben, Whiteboard, Feedback |
| **Stern** | ein markierter Ort im Plan, der auf etwas im Projekt zeigt (Aufgabe, Haftnotiz, Datei …). **Gelb** = für alle, **pink** = nur Missionsbau |

## 4. Seiten

| Seite | Was dort steht |
|---|---|
| [Erste Schritte](SIDC-C2-Getting-Started) | Einstieg für Spieler, Missionsbau und Admins |
| [Rollen & Sichtbarkeit](SIDC-C2-Roles-Visibility) | Gruppen, Plan- & Projektrechte, wer sieht was, Freigaben, öffentliche Links |
| [Abläufe (Schritt für Schritt)](SIDC-C2-Workflows) | Plan anlegen & teilen, Linien und benannte Flächen, Phasen → Briefing/Präsentation, Mission mit Sternen bauen, an Zeus freigeben, Arbeit an Einzelne auslagern, Suche, Versionen |
| [Karte & Werkzeuge](SIDC-C2-Map-Tools) | Werkzeugleiste, Tasten, Topbar, Zeitstrahl, Marker-Wizard, Layer, Höhenlinien, Suche, Einstellungen |
| [Projekte (Missionsbau)](SIDC-C2-Projects) | Verzeichnis, Dateien & Vorlagen, Zeitleiste, Aufgaben, Meilensteine, Feedback, Whiteboard, Verknüpfungs-Codes |
| [ORBAT](SIDC-C2-ORBAT) | Kräfte-Bibliothek, Stärke & Verluste, Feindkräfte für Spieler freigeben, Briefing-Seite |
| [Chat & Benachrichtigungen](SIDC-C2-Chat-Notifications) | Chat, System-Feed, Discord-DM / ntfy / Matrix / Web-Push / E-Mail, Discord-Brücke |
| [Admin & Installation](SIDC-C2-Admin-Install) | Docker-Deploy, `.env`, Kartenpakete, Katalogdateien, Kartenserver / API-Tokens, Admin-Bereich, Architektur |

## 5. Verwandtes

- [ATAKmaps](ATAKmaps) — das spielverbundene Begleit-Tool, von dem C2 abgeleitet ist; erzeugt die
  Kartenpakete und Kataloge.
- [SIDC-Framework](SIDC-Framework) — der Mod, der die SIDC-Symbole, Kataloge und das QuickMenü
  definiert.
- [SIDC-Framework-Channels](SIDC-Framework-Channels) — das Kanalmodell im Spiel (C2 selbst hat die Marker-Channels 2026-10 durch Layer ersetzt).
- Vollständiges Handbuch im Repo: `docs/HANDBUCH.md` (deutsch). Änderungen: `CHANGELOG.md`.
