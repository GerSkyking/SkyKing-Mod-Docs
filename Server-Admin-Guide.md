# Server & Admin Guide

How to run and configure the SkyKing mods on a dedicated server. All three are Enfusion mods with **no CLI build** — configuration is done via config files / profile files, not console commands (a future admin UI / console command is planned for SIDC).

## SkyMap-X

### Config file

`$profile:SMX_ConfigV1.Json` — **loaded only by the server**, then replicated to every client (`SMX_ConfigData`). Players cannot change these. If the file does not exist it is created from the mod defaults on first start.

| Flag | Default | Effect |
|------|---------|--------|
| Allow Mini mode (`m_bIsTabMiniAllow`) | on | `Shift+Q` / Mini display globally |
| Allow Big mode (`m_bIsTabBigAllow`) | on | `Shift+E` / Big display globally |
| Allow paper map (`m_bIsTabNormalPaperMapAllow…`) | — | `M` full‑screen map via the tablet |
| K23 variants of the three above | — | Separate for "K23 loosely carried" vs. "K23 mounted on vest" |
| BFT: show Device ID | on | BFT marker shows the device ID |
| BFT: show Device info | off | BFT marker shows the device info |
| BFT: show player name | off | BFT marker shows the real player name |
| `m_bBFT_ShowAllSquadLeaderMarkers` | off | Show all squad‑leader markers on the SMX minimap instead of suppressing them |

### Prefab / mission setup

- The SMX gadget (`SMX_MainComponentX`) is on the tablet / K23 prefabs shipped with the mod.
- Camera / tracker items use `SMX_ItemDeviceComponent` (independent of the gadget).
- Map config: SMX uses a **GameMode map config** rather than a hard‑coded `.conf` at the vanilla path, for mod compatibility. A mod `.conf` placed at the vanilla path+GUID **replaces** rather than extends — always inherit from the vanilla resource for additive behaviour.
- Per‑player settings live in `$profile:SMX_settings_{playerID}.bin` (mission‑persistent).

### Known server‑relevant issue

BFT tracker markers are spawned without a power/carry check — loose or powered‑off tablets get a BFT marker from mission start, and a normal power‑off does not unregister it. Deferred fix.

## SpeedUI

**No server configuration.** Purely client‑side HUD. The only requirement: the `TAG_SpeedHUD` component must be present on the player prefab your mission uses (shipped on `DefaultPlayerController.et`).

## SIDC-Framework

### How settings load

1. Mod PBO ships **start values** in `Configs/SIDC_ServerDefaultSettings.conf` (and `Configs/SIDC_ServerDefaultValues.conf`).
2. On the **very first** server start (no `Settings.json` yet), those values are written once to
   `$profile:SIDC_Framework/ServerSettings/Settings.json`.
3. From then on **only `Settings.json` is read.** Editing the `.conf` after that has no effect. Admin edits in `Settings.json` win, even after a mod update that changed the `.conf` default.
4. If `Settings.json` is older than the mod's `settingsVersion`, missing new keys are auto‑filled with the mod default and the file is rewritten (self‑healing; not a hard migration).
5. `settingsVersion` is written as the **first** JSON key so it's visible when you open the file.

`Settings.json` is server‑side only. Pure client instances never read/write it.

### `Settings.json` keys

| Key | `.conf` default | Effect | Replicated to clients? |
|-----|-----------------|--------|------------------------|
| `settingsVersion` | `9` | Schema version | no |
| `maxMarkersPerPlayer` | `50` | Max simultaneously placed SIDC markers per player. `0` = unlimited. Enforced in `SIDC_MarkerAuthorityComponent.ProcessCreateMarker()`. | no (server‑enforced) |
| `maxDrawingPointsPerPlayer` | `10000` | Separate limit for line‑drawing points (each point of a segment / chain is its own marker with a placeholder SIDC). Does **not** count against `maxMarkersPerPlayer` and vice‑versa. `0` = unlimited. | no (server‑enforced + local pre‑check) |
| `allowAllPhysicalChannel` | `1` | Players may set physical channel to "All" and see markers from every physical channel. Off = "All" entry hidden everywhere + physical gate always enforced. | **yes** (`RplProp`) |
| `allPhysicalChannelIsDefault` | `0` | Only if `allowAllPhysicalChannel` is on: fresh/first‑time profiles start on "All" instead of the config default. Ignored while `allowAllPhysicalChannel` is off. | **yes** |
| `isPublicLoadSaveAllow` | `0` | On = every player may use the Save/Load menu (`Ctrl+S`/`Ctrl+L`). | **yes** (checked client‑side for the menu keybind) |
| `isAdminZeusLoadSaveAllow` | `1` | Only while `isPublicLoadSaveAllow` is off: server host / declared admin / current Zeus/GM may save/load; nobody else. Both off = feature fully disabled. Fallback default if config missing = `true` (safer than fully open or fully closed). | **yes** |
| `serverExportMarkers` | `0` | Server periodically writes `SIDC_PlacedMarkers.json` (via `SIDC_MarkerExporter`), independent of the client‑side export toggle in `SIDC_PlayerProfile`. | no |
| `serverMarkerSyncIntervalMs` | `1000` | Interval (ms) between server marker‑export write attempts. | no |

### `SIDC_ServerDefaultValues.conf` (client behaviour, NOT in `Settings.json`)

These are **not** enforced server‑side and **not** replicated — putting them in `Settings.json` would be a non‑functional dummy. Edit the `.conf` and rebuild the mod to change them.

| Key | Default | Effect |
|-----|---------|--------|
| `m_fDrawingPointSpacingMeters` | `5` | Minimum spacing between line‑drawing points (`SIDC_DrawingSession`). |
| `m_iMaxUndoHistorySize` | `10` | Client‑side undo history depth (`SIDC_UndoManager`). |
| `m_iMaxRedoHistorySize` | `10` | Client‑side redo history depth (`SIDC_UndoManager`). |

### Editing `Settings.json` by hand

Stop the server (or edit before first start of a session), open
`$profile:SIDC_Framework/ServerSettings/Settings.json`, change the values, save, start. Keep `settingsVersion` as‑is — if you bump it below the mod's version the file self‑heals on next load anyway.

### Channel visibility matrix

Not in `Settings.json`. Edit `Configs/SIDC_ChannelConfig.conf` in the mod and rebuild. See [Channel System](SIDC-Framework-Channels).

## Throw Back Grenade

**No dedicated config file.** `Configs/Editor/TBG_DebugMode.conf` (`TBG_DebugConfig`) only gates debug logging. Behaviour is tuned via script constants (retry counts / delays, landing‑delta timing) — see [Throw Back Grenade §4](Throw-Back-Grenade). Server‑side requirements:

- The grenade prefabs must carry `TBG_ThrowBackComponent` + `TBG_PickUpGrenadeAction` (shipped on M67, RGD5, ANM8HC, RDG2, M18).
- `TBG_CarryingComponent` must be on the character prefab (`Prefabs/Characters/Core/Character_Base.et`).
- MP model: server is a pure RPC relay; all gameplay logic runs on the owning client. Replication lag is handled with bounded retry loops.

---

# Server- & Admin-Handbuch (Deutsch)

Wie die SkyKing-Mods auf einem dedizierten Server betrieben und konfiguriert werden. Alle drei sind Enfusion-Mods **ohne CLI-Build** — Konfiguration über Config-/Profildateien, nicht über Konsolenbefehle (eine spätere Admin-UI / ein Konsolenbefehl ist für SIDC geplant).

## SkyMap-X

### Config-Datei

`$profile:SMX_ConfigV1.Json` — wird **nur vom Server geladen** und dann an jeden Client repliziert (`SMX_ConfigData`). Spieler können das nicht ändern. Existiert die Datei nicht, wird sie beim ersten Start aus den Mod-Defaults erzeugt.

| Flag | Standard | Wirkung |
|------|----------|---------|
| Mini-Modus erlauben (`m_bIsTabMiniAllow`) | an | `Shift+Q` / Mini-Anzeige global |
| Big-Modus erlauben (`m_bIsTabBigAllow`) | an | `Shift+E` / Big-Anzeige global |
| Papierkarte erlauben (`m_bIsTabNormalPaperMapAllow…`) | — | `M`-Vollbildkarte über das Tablet |
| K23-Varianten der drei obigen | — | Getrennt für „K23 lose getragen" vs. „K23 an Weste montiert" |
| BFT: Geräte-ID anzeigen | an | BFT-Marker zeigt die Geräte-ID |
| BFT: Geräte-Info anzeigen | aus | BFT-Marker zeigt die Geräte-Info |
| BFT: Spielername anzeigen | aus | BFT-Marker zeigt den echten Spielernamen |
| `m_bBFT_ShowAllSquadLeaderMarkers` | aus | Alle Squad-Leader-Marker auf der SMX-Minimap zeigen statt unterdrücken |

### Prefab-/Missions-Setup

- Das SMX-Gadget (`SMX_MainComponentX`) sitzt auf den mitgelieferten Tablet-/K23-Prefabs.
- Kamera-/Tracker-Items nutzen `SMX_ItemDeviceComponent` (unabhängig vom Gadget).
- Map-Config: SMX nutzt eine **GameMode-Map-Config** statt einer hartkodierten `.conf` am Vanilla-Pfad (Mod-Kompatibilität). Eine Mod-`.conf` am Vanilla-Pfad+GUID **ersetzt** statt zu erweitern — für additives Verhalten immer von der Vanilla-Ressource erben.
- Spieler-Settings liegen in `$profile:SMX_settings_{playerID}.bin` (missionspersistent).

### Bekanntes serverrelevantes Problem

BFT-Tracker-Marker werden ohne Power-/Trage-Prüfung erzeugt — lose oder ausgeschaltete Tablets bekommen ab Missionsstart einen BFT-Marker, und normales Ausschalten meldet ihn nicht ab. Fix zurückgestellt.

## SpeedUI

**Keine Server-Konfiguration.** Reines Client-HUD. Einzige Voraussetzung: die Komponente `TAG_SpeedHUD` muss am von der Mission genutzten Player-Prefab vorhanden sein (mitgeliefert an `DefaultPlayerController.et`).

## SIDC-Framework

### Wie Einstellungen geladen werden

1. Das Mod-PBO liefert **Startwerte** in `Configs/SIDC_ServerDefaultSettings.conf` (und `Configs/SIDC_ServerDefaultValues.conf`).
2. Beim **allerersten** Serverstart (noch keine `Settings.json`) werden diese Werte einmalig nach
   `$profile:SIDC_Framework/ServerSettings/Settings.json` geschrieben.
3. Danach wird **nur noch `Settings.json` gelesen.** Ein Bearbeiten der `.conf` danach hat keine Wirkung. Admin-Änderungen in `Settings.json` haben Vorrang, auch nach einem Mod-Update mit geändertem `.conf`-Default.
4. Ist `Settings.json` älter als die `settingsVersion` des Mods, werden fehlende neue Schlüssel mit dem Mod-Default aufgefüllt und die Datei neu geschrieben (Selbstheilung; kein hartes Migrationsschema).
5. `settingsVersion` wird als **erster** JSON-Key geschrieben, damit es beim Öffnen sofort sichtbar ist.

`Settings.json` ist serverseitig only. Reine Client-Instanzen lesen/schreiben sie nie.

### `Settings.json`-Schlüssel

| Schlüssel | `.conf`-Standard | Wirkung | An Clients repliziert? |
|-----------|------------------|---------|------------------------|
| `settingsVersion` | `9` | Schema-Version | nein |
| `maxMarkersPerPlayer` | `50` | Max. gleichzeitig platzierte SIDC-Marker pro Spieler. `0` = kein Limit. Erzwungen in `SIDC_MarkerAuthorityComponent.ProcessCreateMarker()`. | nein (server-erzwungen) |
| `maxDrawingPointsPerPlayer` | `10000` | Eigenes Limit für Linien-Zeichenpunkte (jeder Punkt eines Segments / einer Chain ein eigener Marker mit Platzhalter-SIDC). Zählt **nicht** gegen `maxMarkersPerPlayer` und umgekehrt. `0` = kein Limit. | nein (server-erzwungen + lokaler Vorab-Check) |
| `allowAllPhysicalChannel` | `1` | Spieler dürfen physischen Kanal auf „All" stellen und Marker aus jedem physischen Kanal sehen. Aus = „All"-Eintrag überall ausgeblendet + physisches Gate immer erzwungen. | **ja** (`RplProp`) |
| `allPhysicalChannelIsDefault` | `0` | Nur wenn `allowAllPhysicalChannel` an ist: frische/erstmalige Profile starten auf „All" statt Config-Default. Ignoriert solange `allowAllPhysicalChannel` aus ist. | **ja** |
| `isPublicLoadSaveAllow` | `0` | An = jeder Spieler darf das Speichern/Laden-Menü nutzen (`Strg+S`/`Strg+L`). | **ja** (clientseitig für den Menü-Keybind geprüft) |
| `isAdminZeusLoadSaveAllow` | `1` | Nur solange `isPublicLoadSaveAllow` aus ist: Server-Host / deklarierter Admin / aktueller Zeus/GM dürfen speichern/laden; sonst niemand. Beides aus = Feature komplett deaktiviert. Fallback-Default bei fehlender Config = `true` (sicherer als komplett offen oder komplett zu). | **ja** |
| `serverExportMarkers` | `0` | Server schreibt periodisch `SIDC_PlacedMarkers.json` (über `SIDC_MarkerExporter`), unabhängig vom clientseitigen Export-Toggle in `SIDC_PlayerProfile`. | nein |
| `serverMarkerSyncIntervalMs` | `1000` | Intervall (ms) zwischen Server-Marker-Export-Schreibversuchen. | nein |

### `SIDC_ServerDefaultValues.conf` (Client-Verhalten, NICHT in `Settings.json`)

Diese Werte werden **nicht** serverseitig erzwungen und **nicht** repliziert — in `Settings.json` wären sie eine funktionslose Attrappe. Zum Ändern die `.conf` bearbeiten und den Mod neu bauen.

| Schlüssel | Standard | Wirkung |
|-----------|----------|---------|
| `m_fDrawingPointSpacingMeters` | `5` | Mindestabstand zwischen Linien-Zeichenpunkten (`SIDC_DrawingSession`). |
| `m_iMaxUndoHistorySize` | `10` | Clientseitige Undo-Historie-Tiefe (`SIDC_UndoManager`). |
| `m_iMaxRedoHistorySize` | `10` | Clientseitige Redo-Historie-Tiefe (`SIDC_UndoManager`). |

### `Settings.json` von Hand bearbeiten

Server stoppen (oder vor dem ersten Start einer Session bearbeiten),
`$profile:SIDC_Framework/ServerSettings/Settings.json` öffnen, Werte ändern, speichern, starten. `settingsVersion` unverändert lassen — wird sie unter die Mod-Version gesetzt, heilt sich die Datei beim nächsten Laden ohnehin selbst.

### Kanal-Sichtbarkeitsmatrix

Nicht in `Settings.json`. `Configs/SIDC_ChannelConfig.conf` im Mod bearbeiten und neu bauen. Siehe [Kanalsystem](SIDC-Framework-Channels).

## Throw Back Grenade

**Keine eigene Config-Datei.** `Configs/Editor/TBG_DebugMode.conf` (`TBG_DebugConfig`) steuert nur das Debug-Logging. Das Verhalten wird über Script-Konstanten eingestellt (Retry-Anzahl/-Verzögerungen, Landungs-Delta-Timing) — siehe [Throw Back Grenade §4](Throw-Back-Grenade). Serverseitige Voraussetzungen:

- Die Granaten-Prefabs müssen `TBG_ThrowBackComponent` + `TBG_PickUpGrenadeAction` tragen (mitgeliefert bei M67, RGD5, ANM8HC, RDG2, M18).
- `TBG_CarryingComponent` muss am Charakter-Prefab sein (`Prefabs/Characters/Core/Character_Base.et`).
- MP-Modell: Server ist reiner RPC-Relay; die gesamte Gameplay-Logik läuft auf dem Owner-Client. Replikations-Latenz wird über begrenzte Retry-Schleifen behandelt.
