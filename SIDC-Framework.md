# SIDC-Framework (SIDC)

> Source: `SIDC-Framework` · `#define SIDC_FRAMEWORK`. Verified against `Scripts/Game/SIDC/**`, `Configs/**`, `Configs/System/*.conf`.

## 1. What it is

SIDC-Framework is a **military‑symbol marker system** for the Arma Reforger map, based on APP‑6 / MIL‑STD‑2525D symbology (SIDC = *Symbol Identification Code*). It replaces the plain vanilla marker flow with:

- A **Quick‑Marker menu** (`Ctrl+T` on the map, or via the map's **right‑click radial menu** — no keybind needed) — a nested, searchable tree of thousands of APP‑6 symbols organised by faction (Red/OPFOR, Blue/BLUFOR, …), category and sub‑category, with amplifier (echelon) and direction pickers.
- **Channels** — markers are published on a communication channel (Side, Group, Command, Air, Artillery, Infantry, BFT, …) and are visible to others based on configurable **visibility percentages** between channels. Plus a **physical channel** layer (Map / ATAK / All).
- **Phase lines / line drawing** — `Ctrl+D` drawing mode. There is **no freehand drawing**: lines are placed point‑to‑point — click two points for a single segment, or chain several points for a polyline. Configurable color, width, opacity; per‑player **Undo** (`Ctrl+Z`) and **Redo** (`Ctrl+Y`) — key labels as on a German keyboard.
- **Save / Load marker sets** — `Ctrl+S` / `Ctrl+L` (gated by server flags; see below).
- **Inspect** (`I`) — read the SIDC / author / channel of a marker under the cursor.
- **Bridge marker** (`Shift`) and **clipboard** (`X` copy / `C` paste / `V`) operations.
- A **personal marker** for the player, using the SIDC "Personnel" symbol set (role icons: EOD, FO/JFO, MP, sniper, medic, signaler, recon, infantry, commander, 2IC, …).
- **Marker export** — `SIDC_PlacedMarkers.json` (client‑ or server‑side, configurable), plus player‑data / channel‑settings exporters.

The framework only handles its **own** markers (configured via `SIDC_MarkerListEntry` / `SIDC_AllMarkersConfig` with finished SIDC strings). Full SIDC⇄symbol round‑tripping and vanilla/foreign‑mod marker parsing was removed in 2026‑08; what remains of `SIDC_Service` is the digit helpers (affiliation / echelon / mobility) used by the marker factory.

> **No keybinds needed:** the map's vanilla right‑click radial menu also carries "Create Marker", a "SIDC" sub‑category with "SIDC Channel" (change channel) / "Save" / "Load", and "Drawing" — the main SIDC actions are reachable without touching a single keybind.

Almost the entire framework — the channel model, the physical channels, the whole Quick‑Marker menu tree, phase‑line colors/widths, the server settings defaults — is driven by `[BaseContainerProps]` `.conf` resources rather than hard‑coded script, so it's heavily customizable by editing/replacing configs (see [Glossary → Config files](Glossary) and [Server & Admin Guide](Server-Admin-Guide)) without touching the mod's code.

### SIDC digit reference (0‑based index)

| Index | Field | Notes |
|-------|-------|-------|
| 3 | Standard Identity / Affiliation | `1`=Unknown, `2`=Assumed friend, `3`=Friend/BLUFOR, `4`=Neutral/INDFOR/Civilian, `5`=Suspect, `6`=Hostile/OPFOR |
| 6 | Status / Operational Condition | Fully capable / damaged / destroyed / full‑to‑capacity |
| 7 | HQ / Task Force / Dummy (HQTFD) | |
| 8–9 | Amplifier / Descriptor | Echelon for land units (`00`, `11`–`23`); **Mobility** for `Land_Equipment` (values ≥ `30`) |
| 16–17 | Sector 1 modifier | |
| 18–19 | Sector 2 modifier | |

Echelon codes: `11` Team · `12` Squad · `13` Section · `14` Platoon · `15` Company · `16` Battalion · `17` Regiment · `18` Brigade · `19` Division · `20` Corps · `21` Army · `22` Army Group · `23` Theatre.

## 2. How to use it (player)

Everything happens **on the open map**.

1. Open the map (`M` or the SMX GPS mode).
2. **Place a marker:** `Ctrl+T` → opens the Quick‑Marker menu. Navigate the faction row → category → sub‑category. Some entries need an **Amplifier** (echelon) and/or a **Direction** — you'll be prompted. Entries with `m_bPlaceOnClick 1` place immediately at the cursor; others open a preview.
3. **Search:** sub‑menus with a search box let you type a marker name across the configured sub‑categories.
4. **Draw a line:** `Ctrl+D` → drawing mode. Left‑click to set points: two points = one segment, more points = a chained polyline. `Ctrl+Z` = undo, `Ctrl+Y` = redo.
5. **Inspect:** hover a marker, press `I` → shows SIDC, author, channel.
6. **Copy / paste:** `X` copy the marker under the cursor, `C` / `V` paste.
7. **Save / load a marker set:** `Ctrl+S` / `Ctrl+L` (only if the server allows it for you — see §4).
8. **Change your channel:** via the map channel UI (`SIDC_Map_Channel_UI`), and the physical channel spinbox in settings.
9. **Radial menu shortcut:** right‑click the map to open the vanilla radial menu — "Create Marker", the "SIDC" sub‑menu ("SIDC Channel" / "Save" / "Load"), and "Drawing" are all in there too, so steps 2, 4 and 7–8 don't strictly need their keybinds.

## 3. Keybinds

Category **SIDC** in *Settings → Keybindings*. Full table on the [Keybinds](Keybinds) page. Summary:

| Action | Default | Where |
|--------|---------|-------|
| `SIDC_Add_Marker_UI` | `Ctrl + T` | MapContext + debug‑marker menu |
| `SIDC_Framework_Drawing` | `Ctrl + D` | Map / drawing menu |
| `SIDC_Framework_Drawing_Draw` | Left mouse button | Drawing menu |
| `SIDC_Bridge_Marker` | `Shift` | Map / drawing / debug menu |
| `SIDC_Inspect_Marker` | `I` | Map / inspection menu |
| `SIDC_Framework_Save_Marker` | `Ctrl + S` | Map / save menu |
| `SIDC_Framework_Load_Marker` | `Ctrl + L` | Map / load menu |
| `SIDC_Framework_Undo` | `Ctrl + Z` * | Map / drawing menu |
| `SIDC_Framework_Redo` | `Ctrl + Y` * | Map / drawing menu |
| `SIDC_Framework_X` / `_C` / `_V` | `X` / `C` / `V` | Map / debug menu (clipboard) |
| `SIDC_Debug_Change_Side_Player` | `Shift + T` | Map (debug) |

\* Undo binds scancode `KC_Y`, Redo binds scancode `KC_Z` — deliberately swapped so they land on the **"Z"‑ and "Y"‑labelled keys of a German keyboard**. `KC_*` constants are US‑QWERTY scancode positions, not layout‑aware — intentional for the German target audience.

## 4. Server settings

Startup defaults live in the mod PBO (`Configs/SIDC_ServerDefaultSettings.conf`, `Configs/SIDC_ServerDefaultValues.conf`). On the **first** server start they are copied once into `$profile:SIDC_Framework/ServerSettings/Settings.json`. After that, **only that JSON file is read** — admin edits there win, even across mod updates. See [Server & Admin Guide](Server-Admin-Guide) for the full table.

| Key (`Settings.json`) | Default | Effect |
|-----------------------|---------|--------|
| `settingsVersion` | `9` (conf) / self‑healing | Schema version; missing new keys are auto‑filled with the mod default and the file rewritten. |
| `maxMarkersPerPlayer` | `50` | Max simultaneously placed SIDC markers per player. `0` = unlimited. |
| `maxDrawingPointsPerPlayer` | `10000` | Separate limit for line‑drawing points (each point in a segment / chain); does **not** count against `maxMarkersPerPlayer`. `0` = unlimited. |
| `allowAllPhysicalChannel` | `1` (conf) / `false` (fallback) | Lets players set physical channel to "All" and see markers from every physical channel. Replicated to clients. |
| `allPhysicalChannelIsDefault` | `0` / `false` | Only if the above is on: fresh player profiles start on "All" instead of the config default. |
| `isPublicLoadSaveAllow` | `0` / `false` | If on: **every** player may use the Save/Load menu (`Ctrl+S`/`Ctrl+L`). |
| `isAdminZeusLoadSaveAllow` | `1` / `true` | Only relevant while the above is off: server host / declared admin / current Zeus/GM may save/load; everyone else cannot. Off + off = feature fully disabled. |
| `serverExportMarkers` | `0` / `false` | Server periodically writes `SIDC_PlacedMarkers.json` (independent of the client export toggle). |
| `serverMarkerSyncIntervalMs` | `1000` | Interval (ms) between server marker‑export writes. |

Client‑only values in `SIDC_ServerDefaultValues.conf` (**not** enforced server‑side, no replication — pure client behaviour):

| Key | Default | Effect |
|-----|---------|--------|
| `m_fDrawingPointSpacingMeters` | `5` | Minimum spacing between line‑drawing points. |
| `m_iMaxUndoHistorySize` | `10` | Client‑side undo history depth. |
| `m_iMaxRedoHistorySize` | `10` | Client‑side redo history depth. |

## 5. Channel system

The channel model (Side, Group, Command, Air, Artillery, Infantry, BFT + physical Map/ATAK/All) and the cross‑channel visibility‑percentage rules are documented on the dedicated [Channel System](SIDC-Framework-Channels) page.

## 6. Quick‑Marker config

`Configs/SIDC_QuickMarkerConfig.conf` defines the whole menu tree. Top level = **category rows** → **categories** (Red (OPFOR) / Blue (BLUFOR), each with `m_bSetsIdentity 1` + `m_eIdentity`) → **sub‑categories** → **rows** → **buttons** (with optional `m_aNestedRows`). Per‑button fields:

| Field | Meaning |
|-------|---------|
| `m_sButtonLanguageKey` | Button label (literal or `#…` language key) |
| `m_sMarkerSubCategory` | Which marker sub‑category the SIDC comes from (e.g. `Land_Unit`, `Air`, `Dismounted_Individuals`, `Lines`, `Targets`, `Area`, `Flag Markers`, `Controlmeasure`, `Tactics`) |
| `m_sMarkerDescription` | The specific marker name within that sub‑category |
| `m_bNeedsAmp` | Prompt for an amplifier / echelon |
| `m_bNeedsDirection` | Prompt for a facing direction |
| `m_bPlaceOnClick` | `1` place immediately at cursor · `0` open preview first |
| `m_bShowSearch` / `m_aSearchSubCategories` | Show a search box scoped to these sub‑categories |
| `m_bSetsIdentity` / `m_eIdentity` (category) | Category sets the affiliation for everything under it |

Marker definitions themselves: `Configs/AllMarkers/AllMarkers.conf` + `Configs/AllMarkers/Modifications/*.conf` (Air, Land_Unit, Land_Equipment, Sea_Surface, Control_Measure, Mine_Warfare, …).

## 7. Architecture (short)

- `Scripts/Game/SIDC/Core/` — `SIDC_Service` (digit helpers), `SIDC_Symbol`, `SIDC_JsonUtil`, `SIDC_EPlayerRole`.
- `Scripts/Game/SIDC/FrameworkMod/` — controllers & menus: marker menu, change channel, drawing, inspect, load/save, undo, clipboard; `SIDC_MarkerAuthorityComponent` (server authority: enforces `maxMarkersPerPlayer`, load/save gating, replicates the physical‑channel & load/save flags), `SIDC_PlayerIdentityComponent`, `SIDC_PlayerControllerBridge`.
- `Scripts/Game/SIDC/Marker/` — `SIDC_MarkerFactory`, `SIDC_MarkerRequestProcessor`, `SIDC_MarkerExporter`, favorites store, `SIDC_SavedMarkerFile`.
- `Scripts/Game/SIDC/Map/` — menus (dimension/direction selection, favorites, last‑marker list, preview), rendering (`SIDC_PhaseLineRenderer`, `SIDC_MarkerVisibilityRegistry`), session objects (line drawing / phase‑line / marker‑build / undo‑redo).
- `Scripts/Game/SIDC/MarkerConfig/` — all `[BaseContainerProps]` config classes (channels, physical channels, phase‑line color/width, quick‑marker, debug).
- `Scripts/Game/SIDC/Server/` — `SIDC_ServerSettings` (JSON in `$profile:`), `SIDC_ServerDefaultSettingsConfig`, `SIDC_ServerDefaultValues`.
- `Scripts/Game/SIDC/VanillaModded/` — `modded` `SCR_MapMarkerBase`, `SCR_MapMarkerManagerComponent`, `SCR_MapMarkersUI`, `SCR_MapMarkerSyncComponent`, `SCR_MapMarkerWidgetComponent`, plus `SIDC_MarkerHoverIconComponent`.
- New menu presets: `modded enum ChimeraMenuPreset` → `SIDC_FrameworkDebugMarkerChimeraMenu`, `SIDC_Framework_Inspection_ChimeraMenu`, `SIDC_Framework_SaveMarker_ChimeraMenu`, `SIDC_Framework_LoadMarker_ChimeraMenu`, `SIDC_Framework_Drawing_ChimeraMenu`.

## 8. Companion tool

[ATAKmaps](ATAKmaps) — an external web map that reads this mod's file exports (`$profile:SIDC_Framework/LocalMapData/`: `SIDC_PlacedMarkers.json`, `SIDC_PlayerData.json`, `SIDC_ChannelSettings.json`, `SIDC_AllMarkersCatalog.json`) and can write back marker / channel requests (`LocalMapData/Request/`). The mod‑side exporters are `SIDC_MarkerExporter.c`, `SIDC_MarkerRequestProcessor.c`, `SIDC_PlayerDataExporter.c`, `SIDC_ChannelSettingsExporter.c`; the full spec is `ATAKmaps/Doku/SIDC-Data-Interface.md`.

---

# SIDC-Framework (SIDC) — Deutsch

> Quelle: `SIDC-Framework` · `#define SIDC_FRAMEWORK`. Verifiziert gegen `Scripts/Game/SIDC/**`, `Configs/**`, `Configs/System/*.conf`.

## 1. Kurzbeschreibung

Das SIDC-Framework ist ein **Militärsymbol-Markersystem** für die Arma-Reforger-Karte auf Basis von APP-6 / MIL-STD-2525D (SIDC = *Symbol Identification Code*). Es ersetzt den einfachen Vanilla-Marker-Ablauf durch:

- Ein **Quick-Marker-Menü** (`Strg+T` auf der Karte, oder über das **Rechtsklick-Radialmenü** der Karte — kein Keybind nötig) — ein verschachtelter, durchsuchbarer Baum aus tausenden APP-6-Symbolen, geordnet nach Fraktion (Rot/OPFOR, Blau/BLUFOR, …), Kategorie und Unterkategorie, mit Auswahl für Amplifier (Echelon) und Richtung.
- **Kanäle** — Marker werden auf einem Kommunikationskanal veröffentlicht (Side, Group, Command, Air, Artillery, Infantry, BFT, …) und sind für andere anhand konfigurierbarer **Sichtbarkeits-Prozentwerte** zwischen Kanälen sichtbar. Plus eine **physische Kanal-Ebene** (Map / ATAK / All).
- **Phase Lines / Linienzeichnen** — Zeichnen-Modus mit `Strg+D`. **Kein Freihandzeichnen**: Linien werden Punkt für Punkt gesetzt — zwei Punkte = ein Segment, mehrere Punkte = eine verkettete Linie (Chain). Konfigurierbare Farbe, Breite, Deckkraft; **Undo** (`Strg+Z`) und **Redo** (`Strg+Y`) pro Spieler — Tastenbeschriftung wie auf einer deutschen Tastatur.
- **Marker-Sätze speichern/laden** — `Strg+S` / `Strg+L` (durch Server-Flags gesteuert, siehe unten).
- **Inspizieren** (`I`) — SIDC / Autor / Kanal eines Markers unter dem Cursor auslesen.
- **Bridge-Marker** (`Shift`) und **Zwischenablage** (`X` Kopieren / `C` Einfügen / `V`).
- Ein **persönlicher Marker** für den Spieler mit dem SIDC-„Personnel"-Symbolsatz (Rollen-Icons: EOD, FO/JFO, MP, Scharfschütze, Sanitäter, Funker, Aufklärer, Infanterie, Kommandeur, 2IC, …).
- **Marker-Export** — `SIDC_PlacedMarkers.json` (client- oder serverseitig, konfigurierbar), plus Spielerdaten-/Kanaleinstellungs-Exporter.

Das Framework verarbeitet nur seine **eigenen** Marker (per `SIDC_MarkerListEntry` / `SIDC_AllMarkersConfig` mit fertigen SIDC-Strings). Voller SIDC⇄Symbol-Roundtrip und Vanilla-/Fremd-Mod-Marker-Parsing wurden 2026-08 entfernt; von `SIDC_Service` bleiben die Ziffern-Helfer (Affiliation / Echelon / Mobility) für die Marker-Factory.

> **Ohne Keybinds nutzbar:** Das vanilla Rechtsklick-Radialmenü der Karte trägt ebenfalls „Create Marker", eine „SIDC"-Unterkategorie mit „SIDC Channel" (Kanal wechseln) / „Save" / „Load", sowie „Drawing" — die wichtigsten SIDC-Funktionen sind also auch ganz ohne Keybind erreichbar.

Fast das gesamte Framework — das Kanalmodell, die physischen Kanäle, der gesamte Quick-Marker-Menübaum, Phase-Line-Farben/-Breiten, die Server-Settings-Defaults — wird über `[BaseContainerProps]`-`.conf`-Ressourcen statt hartkodiertem Skript gesteuert und ist dadurch stark individualisierbar, indem man Configs bearbeitet/ersetzt (siehe [Glossar → Konfigurationsdateien](Glossary) und [Server- & Admin-Handbuch](Server-Admin-Guide)), ohne den Mod-Code anzufassen.

### SIDC-Ziffern-Referenz (0-basierter Index)

| Index | Feld | Hinweise |
|-------|------|----------|
| 3 | Standard Identity / Affiliation | `1`=Unbekannt, `2`=Vermutet freund, `3`=Freund/BLUFOR, `4`=Neutral/INDFOR/Zivil, `5`=Verdächtig, `6`=Feindlich/OPFOR |
| 6 | Status / Operational Condition | Voll einsatzfähig / beschädigt / zerstört / voll ausgelastet |
| 7 | HQ / Task Force / Dummy (HQTFD) | |
| 8–9 | Amplifier / Descriptor | Echelon bei Land-Units (`00`, `11`–`23`); **Mobility** bei `Land_Equipment` (Werte ≥ `30`) |
| 16–17 | Sektor-1-Modifikator | |
| 18–19 | Sektor-2-Modifikator | |

Echelon-Codes: `11` Team · `12` Squad · `13` Section · `14` Zug · `15` Kompanie · `16` Bataillon · `17` Regiment · `18` Brigade · `19` Division · `20` Korps · `21` Armee · `22` Heeresgruppe · `23` Theatre.

## 2. Bedienung (Spieler)

Alles passiert auf der **geöffneten Karte**.

1. Karte öffnen (`M` oder SMX-GPS-Modus).
2. **Marker setzen:** `Strg+T` → öffnet das Quick-Marker-Menü. Fraktions-Reihe → Kategorie → Unterkategorie navigieren. Einige Einträge brauchen einen **Amplifier** (Echelon) und/oder eine **Richtung** — wird abgefragt. Einträge mit `m_bPlaceOnClick 1` setzen sofort am Cursor; andere öffnen eine Vorschau.
3. **Suchen:** Untermenüs mit Suchfeld erlauben Tippen eines Markernamens über die konfigurierten Unterkategorien.
4. **Linie zeichnen:** `Strg+D` → Zeichnen-Modus. Linksklick setzt Punkte: zwei Punkte = ein Segment, mehr Punkte = eine verkettete Linie. `Strg+Z` = rückgängig, `Strg+Y` = wiederholen.
5. **Inspizieren:** Marker anvisieren, `I` drücken → zeigt SIDC, Autor, Kanal.
6. **Kopieren / Einfügen:** `X` kopiert den Marker unter dem Cursor, `C` / `V` fügt ein.
7. **Marker-Satz speichern/laden:** `Strg+S` / `Strg+L` (nur wenn der Server es dir erlaubt — siehe §4).
8. **Kanal wechseln:** über die Karten-Kanal-UI (`SIDC_Map_Channel_UI`) und die Physische-Kanal-Spinbox in den Einstellungen.
9. **Radialmenü-Abkürzung:** Rechtsklick auf die Karte öffnet das vanilla Radialmenü — „Create Marker", das „SIDC"-Untermenü („SIDC Channel" / „Save" / „Load") und „Drawing" sind auch dort drin, Schritte 2, 4 und 7–8 brauchen also nicht zwingend ihren Keybind.

## 3. Tastenbelegung

Kategorie **SIDC** unter *Einstellungen → Tastenbelegung*. Vollständige Tabelle auf der Seite [Tastenbelegung](Keybinds). Zusammenfassung:

| Aktion | Standard | Wo |
|--------|----------|-----|
| `SIDC_Add_Marker_UI` | `Strg + T` | MapContext + Debug-Marker-Menü |
| `SIDC_Framework_Drawing` | `Strg + D` | Karte / Zeichnen-Menü |
| `SIDC_Framework_Drawing_Draw` | Linke Maustaste | Zeichnen-Menü |
| `SIDC_Bridge_Marker` | `Shift` | Karte / Zeichnen / Debug-Menü |
| `SIDC_Inspect_Marker` | `I` | Karte / Inspektions-Menü |
| `SIDC_Framework_Save_Marker` | `Strg + S` | Karte / Speichern-Menü |
| `SIDC_Framework_Load_Marker` | `Strg + L` | Karte / Laden-Menü |
| `SIDC_Framework_Undo` | `Strg + Z` * | Karte / Zeichnen-Menü |
| `SIDC_Framework_Redo` | `Strg + Y` * | Karte / Zeichnen-Menü |
| `SIDC_Framework_X` / `_C` / `_V` | `X` / `C` / `V` | Karte / Debug-Menü (Zwischenablage) |
| `SIDC_Debug_Change_Side_Player` | `Shift + T` | Karte (Debug) |

\* Undo bindet Scancode `KC_Y`, Redo bindet Scancode `KC_Z` — bewusst getauscht, damit sie auf der **mit „Z" bzw. „Y" beschrifteten Taste einer deutschen Tastatur** liegen. `KC_*`-Konstanten sind US-QWERTY-Scancode-Positionen, nicht layoutbewusst — bewusst für die deutsche Zielgruppe.

## 4. Server-Einstellungen

Startwerte liegen im Mod-PBO (`Configs/SIDC_ServerDefaultSettings.conf`, `Configs/SIDC_ServerDefaultValues.conf`). Beim **ersten** Serverstart werden sie einmalig nach `$profile:SIDC_Framework/ServerSettings/Settings.json` kopiert. Danach wird **nur noch diese JSON gelesen** — Admin-Änderungen dort haben Vorrang, auch über Mod-Updates hinweg. Vollständige Tabelle im [Server- & Admin-Handbuch](Server-Admin-Guide).

| Schlüssel (`Settings.json`) | Standard | Wirkung |
|----------------------------|----------|---------|
| `settingsVersion` | `9` (conf) / selbstheilend | Schema-Version; fehlende neue Schlüssel werden mit dem Mod-Default aufgefüllt und die Datei neu geschrieben. |
| `maxMarkersPerPlayer` | `50` | Max. gleichzeitig platzierte SIDC-Marker pro Spieler. `0` = kein Limit. |
| `maxDrawingPointsPerPlayer` | `10000` | Eigenes Limit für Linien-Zeichenpunkte (jeder Punkt eines Segments / einer Chain); zählt **nicht** gegen `maxMarkersPerPlayer`. `0` = kein Limit. |
| `allowAllPhysicalChannel` | `1` (conf) / `false` (Fallback) | Erlaubt Spielern physischen Kanal „All" und dadurch Marker aus jedem physischen Kanal. An Clients repliziert. |
| `allPhysicalChannelIsDefault` | `0` / `false` | Nur wenn obiges an ist: frische Spielerprofile starten auf „All" statt Config-Default. |
| `isPublicLoadSaveAllow` | `0` / `false` | Wenn an: **jeder** Spieler darf das Speichern/Laden-Menü nutzen (`Strg+S`/`Strg+L`). |
| `isAdminZeusLoadSaveAllow` | `1` / `true` | Nur relevant solange obiges aus ist: Server-Host / deklarierter Admin / aktueller Zeus/GM dürfen speichern/laden; alle anderen nicht. Aus + aus = Feature komplett deaktiviert. |
| `serverExportMarkers` | `0` / `false` | Server schreibt periodisch `SIDC_PlacedMarkers.json` (unabhängig vom Client-Export-Toggle). |
| `serverMarkerSyncIntervalMs` | `1000` | Intervall (ms) zwischen Server-Marker-Export-Schreibversuchen. |

Client-only-Werte in `SIDC_ServerDefaultValues.conf` (**nicht** serverseitig durchgesetzt, keine Replikation — reines Client-Verhalten):

| Schlüssel | Standard | Wirkung |
|-----------|----------|---------|
| `m_fDrawingPointSpacingMeters` | `5` | Mindestabstand zwischen Linien-Zeichenpunkten. |
| `m_iMaxUndoHistorySize` | `10` | Clientseitige Undo-Historie-Tiefe. |
| `m_iMaxRedoHistorySize` | `10` | Clientseitige Redo-Historie-Tiefe. |

## 5. Kanalsystem

Das Kanalmodell (Side, Group, Command, Air, Artillery, Infantry, BFT + physisch Map/ATAK/All) und die kanalübergreifenden Sichtbarkeits-Prozentregeln sind auf der eigenen Seite [Kanalsystem](SIDC-Framework-Channels) dokumentiert.

## 6. Quick-Marker-Konfiguration

`Configs/SIDC_QuickMarkerConfig.conf` definiert den gesamten Menübaum. Oberste Ebene = **Kategorie-Reihen** → **Kategorien** (Rot (OPFOR) / Blau (BLUFOR), je mit `m_bSetsIdentity 1` + `m_eIdentity`) → **Unterkategorien** → **Reihen** → **Buttons** (mit optionalen `m_aNestedRows`). Felder pro Button:

| Feld | Bedeutung |
|------|-----------|
| `m_sButtonLanguageKey` | Button-Beschriftung (literal oder `#…`-Language-Key) |
| `m_sMarkerSubCategory` | Aus welcher Marker-Unterkategorie der SIDC stammt (z. B. `Land_Unit`, `Air`, `Dismounted_Individuals`, `Lines`, `Targets`, `Area`, `Flag Markers`, `Controlmeasure`, `Tactics`) |
| `m_sMarkerDescription` | Der konkrete Markername in dieser Unterkategorie |
| `m_bNeedsAmp` | Amplifier / Echelon abfragen |
| `m_bNeedsDirection` | Blickrichtung abfragen |
| `m_bPlaceOnClick` | `1` sofort am Cursor setzen · `0` erst Vorschau öffnen |
| `m_bShowSearch` / `m_aSearchSubCategories` | Suchfeld über diese Unterkategorien zeigen |
| `m_bSetsIdentity` / `m_eIdentity` (Kategorie) | Kategorie legt die Affiliation für alles darunter fest |

Marker-Definitionen selbst: `Configs/AllMarkers/AllMarkers.conf` + `Configs/AllMarkers/Modifications/*.conf` (Air, Land_Unit, Land_Equipment, Sea_Surface, Control_Measure, Mine_Warfare, …).

## 7. Architektur (kurz)

- `Scripts/Game/SIDC/Core/` — `SIDC_Service` (Ziffern-Helfer), `SIDC_Symbol`, `SIDC_JsonUtil`, `SIDC_EPlayerRole`.
- `Scripts/Game/SIDC/FrameworkMod/` — Controller & Menüs: Marker-Menü, Kanalwechsel, Zeichnen, Inspizieren, Laden/Speichern, Undo, Zwischenablage; `SIDC_MarkerAuthorityComponent` (Server-Autorität: erzwingt `maxMarkersPerPlayer`, Load/Save-Gating, repliziert Physische-Kanal- & Load/Save-Flags), `SIDC_PlayerIdentityComponent`, `SIDC_PlayerControllerBridge`.
- `Scripts/Game/SIDC/Marker/` — `SIDC_MarkerFactory`, `SIDC_MarkerRequestProcessor`, `SIDC_MarkerExporter`, Favoriten-Store, `SIDC_SavedMarkerFile`.
- `Scripts/Game/SIDC/Map/` — Menüs (Dimensions-/Richtungsauswahl, Favoriten, Letzte-Marker-Liste, Vorschau), Rendering (`SIDC_PhaseLineRenderer`, `SIDC_MarkerVisibilityRegistry`), Session-Objekte (Linienzeichnen / Phase-Line / Marker-Build / Undo-Redo).
- `Scripts/Game/SIDC/MarkerConfig/` — alle `[BaseContainerProps]`-Config-Klassen (Kanäle, physische Kanäle, Phase-Line-Farbe/-Breite, Quick-Marker, Debug).
- `Scripts/Game/SIDC/Server/` — `SIDC_ServerSettings` (JSON in `$profile:`), `SIDC_ServerDefaultSettingsConfig`, `SIDC_ServerDefaultValues`.
- `Scripts/Game/SIDC/VanillaModded/` — `modded` `SCR_MapMarkerBase`, `SCR_MapMarkerManagerComponent`, `SCR_MapMarkersUI`, `SCR_MapMarkerSyncComponent`, `SCR_MapMarkerWidgetComponent`, plus `SIDC_MarkerHoverIconComponent`.
- Neue Menü-Presets: `modded enum ChimeraMenuPreset` → `SIDC_FrameworkDebugMarkerChimeraMenu`, `SIDC_Framework_Inspection_ChimeraMenu`, `SIDC_Framework_SaveMarker_ChimeraMenu`, `SIDC_Framework_LoadMarker_ChimeraMenu`, `SIDC_Framework_Drawing_ChimeraMenu`.

## 8. Begleit-Tool

[ATAKmaps](ATAKmaps) — eine externe Web-Karte, die die Dateiexporte dieses Mods liest (`$profile:SIDC_Framework/LocalMapData/`: `SIDC_PlacedMarkers.json`, `SIDC_PlayerData.json`, `SIDC_ChannelSettings.json`, `SIDC_AllMarkersCatalog.json`) und Marker-/Kanal-Requests zurückschreiben kann (`LocalMapData/Request/`). Die mod-seitigen Exporter sind `SIDC_MarkerExporter.c`, `SIDC_MarkerRequestProcessor.c`, `SIDC_PlayerDataExporter.c`, `SIDC_ChannelSettingsExporter.c`; die vollständige Spezifikation ist `ATAKmaps/Doku/SIDC-Data-Interface.md`.
