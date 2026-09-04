# Glossary

| Term | Meaning |
|------|---------|
| **SMX** | SkyMap-X — the tablet / wrist‑gadget mod. |
| **SUI** | SpeedUI — the speed HUD mod. |
| **SIDC** | SIDC-Framework — the military‑symbol marker system. Also: *Symbol Identification Code*, the APP‑6 / MIL‑STD‑2525D 20‑digit code identifying a symbol. |
| **TBG** | Throw Back Grenade — pick up a live thrown grenade and rethrow it. |
| **Dummy grenade** | TBG: an inventory copy of the picked‑up grenade, spawned into the throwable slot so the player can throw with normal input. |
| **Flight proxy** | TBG: a hidden entity the carried grenade is parented to during flight so it follows the dummy's arc; kept after landing as a position anchor. |
| **P1 snap** | TBG: the inventory system snapping an item back to the character's position (P1) — what the flight proxy prevents. |
| **`ETBGGrenadeState`** | TBG grenade state: IDLE / THROWN / CARRIED / ACTIVATED. |
| **ATAKmaps** | External web-map tool (local server + browser) that shows live player position and SIDC markers by reading the SIDC-Framework mod's file exports. Not a mod. |
| **LocalMapData** | The folder `$profile:SIDC_Framework/LocalMapData/` where the SIDC-Framework mod writes its JSON exports and reads request files — the interface ATAKmaps uses. |
| **`ownerFaction`** | Field in `SIDC_PlacedMarkers.json`: the marker author's real game faction (`SCR_FactionManager` key), separate from the SIDC `channel`. |
| **APP‑6 / MIL‑STD‑2525D** | NATO / US standards for military map symbology. |
| **BFT** | Blue Force Tracking — showing friendly (or channel‑visible) devices/units as map markers. |
| **Affiliation / Identity** | SIDC digit 3: Unknown / Assumed friend / Friend / Neutral / Suspect / Hostile. |
| **Echelon** | Unit size (team … theatre), SIDC digits 8–9 for land units. |
| **Mobility** | Movement type for equipment, SIDC digits 8–9 for `Land_Equipment` (values ≥ 30). |
| **Amplifier** | Umbrella term for the SIDC digit 8–9 field (echelon or mobility). |
| **HQTFD** | Headquarters / Task Force / Dummy indicator, SIDC digit 7. |
| **Communication channel** | SIDC: the channel a marker is published on (Side, Group, Command, Air, Artillery, Infantry, BFT). |
| **Physical channel** | SIDC: hard visibility gate — Map / ATAK / All. |
| **Visibility percent** | SIDC: chance a marker on one channel is shown to a viewer on another channel. |
| **Quick‑Marker menu** | SIDC's nested `Ctrl+T` menu for picking and placing a symbol. |
| **K23** | SMX wrist mount worn on the vest, with its own input context. |
| **TabMode** | SMX: display variant — OFFtab / ONtab / HANDtab / BIGtab / MINITab / PAPERMap / GMmode. |
| **Mode** | SMX: content mode — OFF / ON / GPS / CHAT / SETTINGS / FEEDS. |
| **ATAK** | SMX tablet/K23 prefab variant **without** the built‑in full‑screen paper map (`m_bIsTabNormalPaperMapAllowOnPrefab` = off on the prefab). |
| **ATAK + Map** (`_MAP` prefab suffix) | SMX tablet/K23 prefab variant that already **includes** the full‑screen paper map (`m_bIsTabNormalPaperMapAllowOnPrefab` = on). Don't add a separate map‑capable item/prefab alongside it — two paper‑map‑enabled devices on the same loadout is a duplicate‑map error, not additive. |
| **`$profile:`** | Path prefix for the player (or dedicated‑server) profile directory. |
| **`GetInstance()`** | The singleton access pattern used throughout all three mods. |
| **`Settings.json`** | SIDC: the admin‑editable server settings file in `$profile:SIDC_Framework/ServerSettings/`. |
| **`SMX_ConfigV1.Json`** | SMX: the server‑loaded, client‑replicated config file. |
| **QWERTZ note** | `KC_*` key constants are US‑QWERTY scancode positions; on a German keyboard Y/Z are physically swapped. |

## Config files

Every `.conf`/`.json`/`.bin`/`.txt` file the SkyKing mods read or write, and what it's for.

| File | Mod | Edited by | Purpose |
|------|-----|-----------|---------|
| `$profile:SMX_ConfigV1.Json` | SMX | Server admin | Server‑wide feature‑flag toggles (Mini/Big/paper‑map allow, BFT display) — see [Server & Admin Guide](Server-Admin-Guide). |
| `$profile:SMX_settings_{playerID}.bin` | SMX | Player (in‑game) | Per‑player tablet + HUD settings — see [Client Settings Guide](Client-Settings-Guide). |
| `Configs/Faction Config/SMX_Faction_*.conf` (`SMX_FactionConfig`) | SMX | Mod dev (rebuild) | Per‑item: which real game factions (`SCR_FactionManager` keys) count as that item's own faction for the SMX faction channel. |
| `Configs/SMX/SMX_Color_ATAK.conf` / `SMX_Color_MOUNT.conf` (`SMX_ColorTextureConfig`) | SMX | Mod dev (rebuild) | Selectable tablet / K23‑mount skin colors offered in the client "Tablet color" / "K23 mount color" setting. |
| `$profile:SIDC_Framework/ServerSettings/Settings.json` | SIDC | Server admin | The live, admin‑editable server settings — see [Server & Admin Guide](Server-Admin-Guide). |
| `Configs/SIDC_ServerDefaultSettings.conf` / `SIDC_ServerDefaultValues.conf` | SIDC | Mod dev (rebuild), overridable per‑mission | Mod‑shipped default values: the former seeds `Settings.json` on first server start, the latter holds client‑only behaviour never written to `Settings.json` (undo/redo depth, drawing‑point spacing). |
| `Configs/SIDC_QuickMarkerConfig.conf` | SIDC | Mod dev (rebuild), overridable per‑mission | Defines the whole `Ctrl+T` Quick‑Marker menu tree (categories → sub‑categories → rows → buttons). |
| `Configs/SIDC_ChannelConfig.conf` | SIDC | Mod dev (rebuild) | The communication‑channel visibility matrix (which channels can see which). |
| `Configs/SIDC_PhysicalChannelConfig.conf` | SIDC | Mod dev (rebuild) | Defines the physical channels (Map / ATAK / All) and their default. |
| `Configs/SIDC_PhaseLineColorConfig.conf` / `SIDC_PhaseLineWidthConfig.conf` | SIDC | Mod dev (rebuild) | Selectable colors / line widths offered for phase‑line drawing. |
| `$profile:SIDC_Framework/LocalMapData/SIDC_PlacedMarkers.json` | SIDC | Written by server/client (export) | Marker export consumed by [ATAKmaps](ATAKmaps); unfiltered by faction. |
| `Configs/Editor/TBG_DebugMode.conf` (`TBG_DebugConfig`) | TBG | Mod dev / mission | Debug logging on/off only — no gameplay settings. |
| `$profile:SpeedUI_Settings.txt` | SpeedUI | Player (in‑game) | Per‑player HUD settings (bar/units/transparency) — see [Client Settings Guide](Client-Settings-Guide). |
| `Configs/GameplaySettings.conf` (`SUI_SpeedUISettings` module) | SpeedUI | Mission author (scenario Gameplay Settings) | Mission‑level defaults for the SpeedUI bar/units toggle, same mechanism as vanilla scenario Gameplay Settings modules. |

---

# Glossar (Deutsch)

| Begriff | Bedeutung |
|---------|-----------|
| **SMX** | SkyMap-X — der Tablet-/Armband-Gadget-Mod. |
| **SUI** | SpeedUI — der Geschwindigkeits-HUD-Mod. |
| **SIDC** | SIDC-Framework — das Militärsymbol-Markersystem. Auch: *Symbol Identification Code*, der 20-stellige APP-6-/MIL-STD-2525D-Code, der ein Symbol identifiziert. |
| **TBG** | Throw Back Grenade — eine scharfe geworfene Granate aufheben und zurückwerfen. |
| **Dummy-Granate** | TBG: eine Inventarkopie der aufgehobenen Granate, in den Wurfslot gespawnt, damit der Spieler mit normaler Eingabe werfen kann. |
| **Flight-Proxy** | TBG: eine versteckte Entität, an die die getragene Granate während des Flugs gehängt wird, damit sie der Wurfbahn des Dummys folgt; nach der Landung als Positions-Anker behalten. |
| **P1-Snap** | TBG: das Inventarsystem schnappt ein Item zurück auf die Charakterposition (P1) — genau das verhindert der Flight-Proxy. |
| **`ETBGGrenadeState`** | TBG-Granatenzustand: IDLE / THROWN / CARRIED / ACTIVATED. |
| **ATAKmaps** | Externes Web-Karten-Tool (lokaler Server + Browser), das Live-Spielerposition und SIDC-Marker zeigt, indem es die Dateiexporte des SIDC-Framework-Mods liest. Kein Mod. |
| **LocalMapData** | Der Ordner `$profile:SIDC_Framework/LocalMapData/`, in den der SIDC-Framework-Mod seine JSON-Exporte schreibt und Request-Dateien liest — die Schnittstelle, die ATAKmaps nutzt. |
| **`ownerFaction`** | Feld in `SIDC_PlacedMarkers.json`: die echte Spiel-Fraktion des Marker-Erstellers (`SCR_FactionManager`-Key), getrennt vom SIDC-`channel`. |
| **APP-6 / MIL-STD-2525D** | NATO-/US-Standards für militärische Kartensymbolik. |
| **BFT** | Blue Force Tracking — befreundete (oder kanal-sichtbare) Geräte/Einheiten als Kartenmarker. |
| **Affiliation / Identity** | SIDC-Ziffer 3: Unbekannt / Vermutet freund / Freund / Neutral / Verdächtig / Feindlich. |
| **Echelon** | Einheitengröße (Team … Theatre), SIDC-Ziffern 8–9 bei Land-Units. |
| **Mobility** | Bewegungstyp bei Ausrüstung, SIDC-Ziffern 8–9 bei `Land_Equipment` (Werte ≥ 30). |
| **Amplifier** | Oberbegriff für das SIDC-Feld Ziffern 8–9 (Echelon oder Mobility). |
| **HQTFD** | Headquarters/Task-Force/Dummy-Indikator, SIDC-Ziffer 7. |
| **Kommunikationskanal** | SIDC: der Kanal, auf dem ein Marker veröffentlicht wird (Side, Group, Command, Air, Artillery, Infantry, BFT). |
| **Physischer Kanal** | SIDC: hartes Sichtbarkeits-Gate — Map / ATAK / All. |
| **Sichtbarkeits-Prozent** | SIDC: Chance, dass ein Marker eines Kanals einem Betrachter auf einem anderen Kanal angezeigt wird. |
| **Quick-Marker-Menü** | SIDCs verschachteltes `Strg+T`-Menü zum Wählen und Setzen eines Symbols. |
| **K23** | SMX-Armhalterung an der Weste, mit eigenem Eingabekontext. |
| **TabMode** | SMX: Anzeige-Variante — OFFtab / ONtab / HANDtab / BIGtab / MINITab / PAPERMap / GMmode. |
| **Mode** | SMX: Inhalts-Modus — OFF / ON / GPS / CHAT / SETTINGS / FEEDS. |
| **ATAK** | SMX Tablet-/K23-Prefab-Variante **ohne** eingebaute Vollbild-Papierkarte (`m_bIsTabNormalPaperMapAllowOnPrefab` = aus am Prefab). |
| **ATAK + Map** (`_MAP`-Prefab-Suffix) | SMX Tablet-/K23-Prefab-Variante, die die Vollbild-Papierkarte bereits **eingebaut** hat (`m_bIsTabNormalPaperMapAllowOnPrefab` = an). Kein zusätzliches kartenfähiges Item/Prefab dazu ausgeben — zwei kartenfähige Geräte im selben Loadout sind ein Doppelkarte-Fehler, kein additives Feature. |
| **`$profile:`** | Pfad-Präfix für das Spieler- (oder Dedicated-Server-) Profilverzeichnis. |
| **`GetInstance()`** | Das Singleton-Zugriffsmuster in allen drei Mods. |
| **`Settings.json`** | SIDC: die admin-editierbare Server-Settings-Datei in `$profile:SIDC_Framework/ServerSettings/`. |
| **`SMX_ConfigV1.Json`** | SMX: die server-geladene, an Clients replizierte Config-Datei. |
| **QWERTZ-Hinweis** | `KC_*`-Tastenkonstanten sind US-QWERTY-Scancode-Positionen; auf einer deutschen Tastatur sind Y/Z physisch vertauscht. |

## Konfigurationsdateien

Jede `.conf`-/`.json`-/`.bin`-/`.txt`-Datei, die die SkyKing-Mods lesen oder schreiben, und wofür sie da ist.

| Datei | Mod | Bearbeitet von | Zweck |
|-------|-----|-----------------|-------|
| `$profile:SMX_ConfigV1.Json` | SMX | Server-Admin | Serverweite Feature-Flags (Mini/Big/Papierkarte erlauben, BFT-Anzeige) — siehe [Server- & Admin-Handbuch](Server-Admin-Guide). |
| `$profile:SMX_settings_{playerID}.bin` | SMX | Spieler (im Spiel) | Pro-Spieler-Tablet- und HUD-Settings — siehe [Client-Einstellungen](Client-Settings-Guide). |
| `Configs/Faction Config/SMX_Faction_*.conf` (`SMX_FactionConfig`) | SMX | Mod-Dev (Rebuild) | Pro Item: welche echten Spiel-Fraktionen (`SCR_FactionManager`-Keys) als eigene Fraktion für den SMX-Fraktionskanal dieses Items zählen. |
| `Configs/SMX/SMX_Color_ATAK.conf` / `SMX_Color_MOUNT.conf` (`SMX_ColorTextureConfig`) | SMX | Mod-Dev (Rebuild) | Auswählbare Tablet-/K23-Halterungs-Skinfarben in der Client-Einstellung „Tablet-Farbe"/„K23-Halterungs-Farbe". |
| `$profile:SIDC_Framework/ServerSettings/Settings.json` | SIDC | Server-Admin | Die live admin-editierbaren Server-Settings — siehe [Server- & Admin-Handbuch](Server-Admin-Guide). |
| `Configs/SIDC_ServerDefaultSettings.conf` / `SIDC_ServerDefaultValues.conf` | SIDC | Mod-Dev (Rebuild), pro Mission überschreibbar | Mod-mitgelieferte Default-Werte: Ersteres speist `Settings.json` beim allerersten Serverstart, Letzteres enthält reines Client-Verhalten, das nie in `Settings.json` landet (Undo/Redo-Tiefe, Zeichenpunkt-Abstand). |
| `Configs/SIDC_QuickMarkerConfig.conf` | SIDC | Mod-Dev (Rebuild), pro Mission überschreibbar | Definiert den gesamten `Strg+T`-Quick-Marker-Menübaum (Kategorien → Unterkategorien → Reihen → Buttons). |
| `Configs/SIDC_ChannelConfig.conf` | SIDC | Mod-Dev (Rebuild) | Die Sichtbarkeitsmatrix der Kommunikationskanäle (welcher Kanal sieht welchen). |
| `Configs/SIDC_PhysicalChannelConfig.conf` | SIDC | Mod-Dev (Rebuild) | Definiert die physischen Kanäle (Map / ATAK / All) und deren Default. |
| `Configs/SIDC_PhaseLineColorConfig.conf` / `SIDC_PhaseLineWidthConfig.conf` | SIDC | Mod-Dev (Rebuild) | Auswählbare Farben/Linienbreiten für das Phase-Line-Zeichnen. |
| `$profile:SIDC_Framework/LocalMapData/SIDC_PlacedMarkers.json` | SIDC | Server/Client schreibt (Export) | Marker-Export, den [ATAKmaps](ATAKmaps) konsumiert; ungefiltert nach Fraktion. |
| `Configs/Editor/TBG_DebugMode.conf` (`TBG_DebugConfig`) | TBG | Mod-Dev/Mission | Nur Debug-Logging an/aus — keine Gameplay-Settings. |
| `$profile:SpeedUI_Settings.txt` | SpeedUI | Spieler (im Spiel) | Pro-Spieler-HUD-Settings (Balken/Einheiten/Transparenz) — siehe [Client-Einstellungen](Client-Settings-Guide). |
| `Configs/GameplaySettings.conf` (`SUI_SpeedUISettings`-Modul) | SpeedUI | Missionsersteller (Szenario-Gameplay-Settings) | Missionsweite Defaults für den SpeedUI-Balken/Einheiten-Toggle, gleicher Mechanismus wie vanilla Szenario-Gameplay-Settings-Module. |
