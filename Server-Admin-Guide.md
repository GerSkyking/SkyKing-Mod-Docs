# Server & Admin Guide

How to run and configure the SkyKing mods on a dedicated server.

## SkyMap-X

### Config file

`$profile:SMX_ConfigV1.Json` — **loaded only by the server**, then replicated to every client (`SMX_ConfigData`). Players cannot change these. If the file does not exist it is created from the mod defaults on first start. The `V1` in the filename is a schema version — it is bumped on breaking format changes.

| Key | Default | Effect |
|-----|---------|--------|
| `isTabMiniAllow` | `true` | Mini display (`Shift+Q`) allowed on the tablet |
| `isTabBigAllow` | `true` | Big display (`Shift+E`) allowed on the tablet |
| `isTabNormalPaperMapAllow` | `true` | Full‑screen paper map (`M`) allowed on the tablet |
| `isK23MiniAllow` | `true` | Mini display allowed on the K23, unmounted (loosely carried) |
| `isK23NormalPaperMapAllow` | `true` | Paper map allowed on the K23, unmounted |
| `isK23MountedMiniAllow` | `true` | Mini display allowed on the K23, mounted on a vest |
| `isK23MountedNormalPaperMapAllow` | `true` | Paper map allowed on the K23, mounted on a vest |
| `isK23BigAllow` | `true` | Big display allowed on the K23, unmounted |
| `isK23MountedBigAllow` | `true` | Big display allowed on the K23, mounted on a vest |
| `bftShowDeviceID` | `true` | BFT marker shows the device ID |
| `bftShowDeviceInfo` | `false` | BFT marker shows the device info |
| `bftShowPlayerName` | `false` | BFT marker shows the real player name |
| `bftShowAllSquadLeaderMarkers` | `false` | Keep the vanilla squad‑leader map markers visible on the SMX map too, instead of suppressing them when a matching SMX tracker marker exists |

### Prefab / mission setup

- The SMX gadget (`SMX_MainComponentX`) is on the tablet / K23 prefabs shipped with the mod.
- Camera / tracker items (`SMX_ItemDeviceComponent`, roles Cam / Tracker / HeadCam) are configured per‑item via prefab attributes — not through the server config file. The item itself can only be placed/attached in a mission via **Zeus** or the **World/Mission Editor**, same as any other item.
- Per‑player settings live in `$profile:SMX_settings_{playerID}.bin` (mission‑persistent) — see the [Client Settings Guide](Client-Settings-Guide).

### Known server‑relevant issue

BFT tracker markers are spawned without a power/carry check — loose or powered‑off tablets get a BFT marker from mission start, and a normal power‑off does not unregister it. Deferred fix.

## SpeedUI

**No server configuration.** Purely client‑side HUD. The only requirement: the `TAG_SpeedHUD` component must be present on the player prefab your mission uses (shipped on `DefaultPlayerController.et`).

## SIDC-Framework

Channels, physical channels, the Quick‑Marker menu tree and phase‑line colors/widths are all defined in `.conf` resources (`[BaseContainerProps]`), not hard‑coded — see the full list in the [Glossary → Config files](Glossary). That makes the whole marker/channel structure heavily customizable by editing configs and rebuilding, without touching the mod's script.

### How settings load

1. Mod PBO ships **start values** in `Configs/SIDC_ServerDefaultSettings.conf` (and `Configs/SIDC_ServerDefaultValues.conf`). Like the [Quick Marker Config](SIDC-Framework) (`Configs/SIDC_QuickMarkerConfig.conf`), both are GUID‑referenced `.conf` resources — a mission can override any of them by shipping a mission‑local resource at the same path/GUID.
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

### ATAKmaps companion tool

`serverExportMarkers` = `true` makes the server write `SIDC_PlacedMarkers.json` into `$profile:SIDC_Framework/LocalMapData/` for the [ATAKmaps](ATAKmaps) web map. Note: the export is **unfiltered** (all factions / channels / authors) — there is no server‑side faction filter in the mod, so faction‑specific web maps must filter on `ownerFaction` in the tool.

## Throw Back Grenade

**No dedicated config file.** `Configs/Editor/TBG_DebugMode.conf` (`TBG_DebugConfig`) only gates debug logging. Behaviour is tuned via script constants (retry counts / delays, landing‑delta timing) — see [Throw Back Grenade §4](Throw-Back-Grenade). Server‑side requirements:

- The grenade prefabs must carry `TBG_ThrowBackComponent` + `TBG_PickUpGrenadeAction` (shipped on M67, RGD5, ANM8HC, RDG2, M18).
- `TBG_CarryingComponent` must be on the character prefab (`Prefabs/Characters/Core/Character_Base.et`).
- MP model: server is a pure RPC relay; all gameplay logic runs on the owning client. Replication lag is handled with bounded retry loops.

---

# Server- & Admin-Handbuch (Deutsch)

Wie die SkyKing-Mods auf einem dedizierten Server betrieben und konfiguriert werden.

## SkyMap-X

### Config-Datei

`$profile:SMX_ConfigV1.Json` — wird **nur vom Server geladen** und dann an jeden Client repliziert (`SMX_ConfigData`). Spieler können das nicht ändern. Existiert die Datei nicht, wird sie beim ersten Start aus den Mod-Defaults erzeugt. Das `V1` im Dateinamen ist eine Schema-Version — sie wird bei breaking Format-Änderungen hochgezählt.

| Schlüssel | Standard | Wirkung |
|-----------|----------|---------|
| `isTabMiniAllow` | `true` | Mini-Anzeige (`Shift+Q`) am Tablet erlaubt |
| `isTabBigAllow` | `true` | Big-Anzeige (`Shift+E`) am Tablet erlaubt |
| `isTabNormalPaperMapAllow` | `true` | Vollbild-Papierkarte (`M`) am Tablet erlaubt |
| `isK23MiniAllow` | `true` | Mini-Anzeige am K23 erlaubt, ungemountet (lose getragen) |
| `isK23NormalPaperMapAllow` | `true` | Papierkarte am K23 erlaubt, ungemountet |
| `isK23MountedMiniAllow` | `true` | Mini-Anzeige am K23 erlaubt, an Weste montiert |
| `isK23MountedNormalPaperMapAllow` | `true` | Papierkarte am K23 erlaubt, an Weste montiert |
| `isK23BigAllow` | `true` | Big-Anzeige am K23 erlaubt, ungemountet |
| `isK23MountedBigAllow` | `true` | Big-Anzeige am K23 erlaubt, an Weste montiert |
| `bftShowDeviceID` | `true` | BFT-Marker zeigt die Geräte-ID |
| `bftShowDeviceInfo` | `false` | BFT-Marker zeigt die Geräte-Info |
| `bftShowPlayerName` | `false` | BFT-Marker zeigt den echten Spielernamen |
| `bftShowAllSquadLeaderMarkers` | `false` | Vanilla-Squad-Leader-Marker bleiben zusätzlich auf der SMX-Karte sichtbar, statt unterdrückt zu werden wenn ein passender SMX-Tracker-Marker existiert |

### Prefab-/Missions-Setup

- Das SMX-Gadget (`SMX_MainComponentX`) sitzt auf den mitgelieferten Tablet-/K23-Prefabs.
- Kamera-/Tracker-Items (`SMX_ItemDeviceComponent`, Rollen Cam / Tracker / HeadCam) werden pro Item über Prefab-Attribute konfiguriert — nicht über die Server-Config-Datei. Platziert/angebracht werden kann das Item in einer Mission nur über **Zeus** oder den **World-/Mission-Editor**, wie jedes andere Item auch.
- Spieler-Settings liegen in `$profile:SMX_settings_{playerID}.bin` (missionspersistent) — siehe das [Client-Einstellungen-Handbuch](Client-Settings-Guide).

### Bekanntes serverrelevantes Problem

BFT-Tracker-Marker werden ohne Power-/Trage-Prüfung erzeugt — lose oder ausgeschaltete Tablets bekommen ab Missionsstart einen BFT-Marker, und normales Ausschalten meldet ihn nicht ab. Fix zurückgestellt.

## SpeedUI

**Keine Server-Konfiguration.** Reines Client-HUD. Einzige Voraussetzung: die Komponente `TAG_SpeedHUD` muss am von der Mission genutzten Player-Prefab vorhanden sein (mitgeliefert an `DefaultPlayerController.et`).

## SIDC-Framework

Kanäle, physische Kanäle, der Quick-Marker-Menübaum sowie Phase-Line-Farben/-Breiten sind alle in `.conf`-Ressourcen (`[BaseContainerProps]`) definiert, nicht hartkodiert — vollständige Liste im [Glossar → Konfigurationsdateien](Glossary). Dadurch ist die gesamte Marker-/Kanalstruktur durch Bearbeiten der Configs und Neubau des Mods stark individualisierbar, ohne den Mod-Code anzufassen.

### Wie Einstellungen geladen werden

1. Das Mod-PBO liefert **Startwerte** in `Configs/SIDC_ServerDefaultSettings.conf` (und `Configs/SIDC_ServerDefaultValues.conf`). Wie die [Quick-Marker-Config](SIDC-Framework) (`Configs/SIDC_QuickMarkerConfig.conf`) sind beides GUID-referenzierte `.conf`-Ressourcen — eine Mission kann sie überschreiben, indem sie eine missionseigene Ressource unter demselben Pfad/derselben GUID mitliefert.
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

### ATAKmaps-Begleit-Tool

`serverExportMarkers` = `true` lässt den Server `SIDC_PlacedMarkers.json` nach `$profile:SIDC_Framework/LocalMapData/` schreiben — für die [ATAKmaps](ATAKmaps)-Web-Karte. Hinweis: Der Export ist **ungefiltert** (alle Fraktionen / Kanäle / Ersteller) — es gibt keinen serverseitigen Fraktionsfilter im Mod, fraktionsspezifische Web-Karten müssen also im Tool auf `ownerFaction` filtern.

## Throw Back Grenade

**Keine eigene Config-Datei.** `Configs/Editor/TBG_DebugMode.conf` (`TBG_DebugConfig`) steuert nur das Debug-Logging. Das Verhalten wird über Script-Konstanten eingestellt (Retry-Anzahl/-Verzögerungen, Landungs-Delta-Timing) — siehe [Throw Back Grenade §4](Throw-Back-Grenade). Serverseitige Voraussetzungen:

- Die Granaten-Prefabs müssen `TBG_ThrowBackComponent` + `TBG_PickUpGrenadeAction` tragen (mitgeliefert bei M67, RGD5, ANM8HC, RDG2, M18).
- `TBG_CarryingComponent` muss am Charakter-Prefab sein (`Prefabs/Characters/Core/Character_Base.et`).
- MP-Modell: Server ist reiner RPC-Relay; die gesamte Gameplay-Logik läuft auf dem Owner-Client. Replikations-Latenz wird über begrenzte Retry-Schleifen behandelt.
