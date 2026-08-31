# SkyMap-X (SMX)

> Source: `SkyMap-X-DEV` · Project ID `SkyMapX` (GUID `6718FE4BBDA71C7C`) · ships to the Workshop as **SkyMap-X**.
> Verified against `Scripts/Game/Components/SMX_MainComponentX.c`, `Scripts/Game/UserAction/*.c`, `Configs/System/keyBindingMenu.conf`, `Configs/System/chimeraInputCommon.conf` (state: 2026‑08).

## 1. What it is

SkyMap-X is a **tablet / wrist‑watch gadget** for Arma Reforger. The tablet has four content modes:

| Mode | Key | Purpose |
|------|-----|---------|
| **GPS** | `Ctrl+1` | Tactical map: own position, bearing, MGRS grid, zoom/pan, markers of other visible devices (BFT). Can lock‑on to follow the player or be panned freely. |
| **Chat** | `Ctrl+2` | Text messaging over configurable channels (see [Client Settings](Client-Settings-Guide)). |
| **Settings** | `Ctrl+3` | Device ID / info, communication channels, zoom & brightness presets. |
| **Feeds** | `Ctrl+4` | Live image from camera devices (body‑cams, UGL‑cams, CCTV, trackers) with its own pan/zoom/brightness/lock. |

Two internal states also exist: **OFF** (tablet powered down) and **ON** (powered, no mode picked yet — a short transition state).

### Display variants (how/where the tablet is shown)

| Variant | Toggle | Description |
|---------|--------|-------------|
| **Hand (HANDtab)** | `Ctrl+F5` or taking it out of the inventory | Tablet physically held, medium size, rendered in 3D in front of the character. |
| **Big (BIGtab)** | `Shift+E` | Large 2D screen (~85 % of the display). Mouse cursor auto‑enabled, no manual toggle. Server can disable. |
| **Mini (MINITab)** | `Shift+Q` | Small corner display, Arma‑3‑CTab style. Server can disable. |
| **PaperMap (PAPERMap)** | game map key `M` | The normal full‑screen map, if allowed on the prefab and by the server. |
| **GameMaster (GMmode)** | automatic | Detected when the GM editor is open; dedicated GM presentation. |

Switching Big ↔ Mini goes through a short intermediate step (ON state + 100 ms delay). This is intentional and prevents an engine‑internal map‑loading fault.

### Blue Force Tracking (BFT)

SMX is part of a BFT system: visible devices of other players appear as markers on the map. Visibility is channel‑based — a remote device is shown when its **sending channels** intersect your tablet's **receiver channels**. Re‑evaluated roughly once per second.

## 2. How to use it

1. Get an SMX tablet (or K23 wrist mount) into your inventory / onto your vest.
2. **Take it into your hand:** `Ctrl+F5`. If it is off, **power it on** with `Ctrl+F1` (2 s start‑up timer).
3. Pick a mode with `Ctrl+1..4`.
4. In GPS / Feeds: zoom with `PageUp` / `PageDown`, pan with `Right‑Ctrl + Arrow keys`, adjust brightness with `Ctrl+PageUp` / `Ctrl+PageDown`.
5. Toggle the mouse cursor with `Alt+Ctrl` in Hand/Mini mode (Big mode has it always). Left‑click acts in cursor mode.
6. Switch to a bigger/smaller view with `Shift+E` (Big) / `Shift+Q` (Mini).
7. Open **Settings mode** (`Ctrl+3`) to set your Device ID, name, and communication channels; press **Save**.

### K23 wrist mount

The K23 is worn on the vest and has its own context, independent of the tablet gadget. `Shift+T` folds/unfolds the K23 mount. The server can allow/deny Mini/Big/PaperMap **separately** for "K23 loosely carried" vs. "K23 mounted on vest".

## 3. Keybinds

See the full table on the [Keybinds](Keybinds) page (category **SkyMap-X** in *Settings → Keybindings*).

## 4. Client settings

There are **two** places to configure SkyMap-X — both per‑player, both saved to `$profile:SMX_settings_{playerID}.bin`:

- **A. The tablet's Settings mode** (`Ctrl+3` on the tablet) — communication channels, device ID/info, zoom, brightness. Saved with the **Save** button.
- **B. The game's Settings menu → "SkyMap-X" tab** (`SMX_UISettingsSubMenu`) — behaviour and appearance options. Saved automatically on each change.

### A. Tablet Settings mode (`Ctrl+3`)

| Setting | Effect |
|---------|--------|
| Device ID | Free‑text ID of your tablet; used for the BFT marker. |
| Device info | Free‑text display name; default = your player name. |
| Sub‑channel 1–4 (send) | Four freely named text channels for sending; each toggled on/off. |
| Sub‑channel 1–4 (receive) | The same four channels for receiving; each toggled on/off. |
| CIV send / receive | Communication on the civilian channel on/off. |
| Faction send / receive | Communication on the faction/team channel (on by default; auto‑filled with the group name on joining a group). |
| Zoom (slider) | Current / default map zoom. |
| Brightness (slider) | Tablet screen brightness in fixed steps. |
| "Randomly reset channel" button | **WIP** — placeholder, currently no function. |

### B. Game Settings → "SkyMap-X" tab

| Setting | Effect |
|---------|--------|
| Auto start | Tablet powers on automatically at mission start. |
| Auto mode | Tablet opens a mode automatically instead of staying on ON. |
| Default mode | Which content mode the tablet opens in (GPS / Chat / Settings / Feeds). |
| Use prefix + Main TAG | Show a personal prefix/tag (free text) on your BFT marker; re‑registers the device live when changed. |
| Notification‑tone duration | Length of the chat/notification sound. |
| Mini‑mode offset up / right / size scale | Position and size of the Mini corner display. |
| Squad‑Mate‑Tracker: scale / show name / show icon / transparency | Appearance of squad‑mate markers on the map. |
| Trace line: R / G / B / alpha / width | Color, opacity and width of the distance/trace line on the map. |
| Tablet color (ATAK) | Tablet skin color, from `SMX_ColorTextureConfig` ("Default" + configured entries). |
| K23 mount color | Skin color of the worn K23 mount, same color list. |

## 5. Server settings

Managed server‑side (`$profile:SMX_ConfigV1.Json`, loaded only by the server and replicated to all clients). See [Server & Admin Guide](Server-Admin-Guide).

| Flag | Effect |
|------|--------|
| Allow Mini mode | Enables/disables `Shift+Q` / Mini globally. |
| Allow Big mode | Enables/disables `Shift+E` / Big globally. |
| Allow paper map | Enables/disables the `M` full‑screen map via the tablet. |
| K23 variants of the three flags above | Separate for "K23 loosely carried" vs. "K23 mounted on vest". |
| BFT: show Device ID | Whether the BFT marker shows the device ID (default: on). |
| BFT: show Device info | Whether the BFT marker shows the device info (default: off). |
| BFT: show player name | Whether the BFT marker shows the real player name (default: off). |
| `m_bBFT_ShowAllSquadLeaderMarkers` | Show all squad‑leader markers on the SMX minimap instead of suppressing them. |

## 6. Known WIP / limitations

- **Quick‑marker** (`Ctrl+F4`): code stub only, currently sets no marker (just hides the UI).
- **Center / Lock‑On** (`Ctrl+F2` / `Ctrl+F3`): marked "WIP" in the keybind menu; the functions work (center + lock on player position) but a code comment flags them as needing rework.
- **Hold‑to‑repeat for the on‑screen Feed buttons** (pan/zoom/brightness via mouse click‑and‑hold in Feed mode) is currently broken — only single clicks work. Keyboard control is unaffected.
- **BFT tracker ignores power state** (open TODO): loose or powered‑off tablets still get a BFT marker from mission start; normal power‑off does not unregister the BFT marker.

## 7. Architecture (short)

`SCR_GadgetComponent` → `SMX_MainComponentX` (main hub: widget state, mode/tab state machine `ESMX_Modes` / `ESMX_TabModes`, settings) with singleton managers: `SMX_DeviceManager`, `SMX_ReplicationManager`, `SMX_MenuManager`, `SMX_TabletMarkerManager`. Hand/Mini/Big all render as plain workspace widgets (no `ChimeraMenuBase`) and share the mode‑switch path; Big mode routes clicks through a custom hit‑test. Full detail: the repo `CLAUDE.md`.

---

# SkyMap-X (SMX) — Deutsch

> Quelle: `SkyMap-X-DEV` · Projekt-ID `SkyMapX` (GUID `6718FE4BBDA71C7C`) · erscheint im Workshop als **SkyMap-X**.
> Verifiziert gegen `Scripts/Game/Components/SMX_MainComponentX.c`, `Scripts/Game/UserAction/*.c`, `Configs/System/keyBindingMenu.conf`, `Configs/System/chimeraInputCommon.conf` (Stand: 2026‑08).

## 1. Kurzbeschreibung

SkyMap-X ist ein **Tablet-/Armbanduhr-Gadget** für Arma Reforger. Das Tablet hat vier Inhalts-Modi:

| Modus | Taste | Zweck |
|-------|-------|-------|
| **GPS** | `Strg+1` | Taktische Karte: eigene Position, Peilrichtung, MGRS-Gitter, Zoom/Pan, Marker anderer sichtbarer Geräte (BFT). Lock-On folgt automatisch der Spielerposition oder freies Verschieben. |
| **Chat** | `Strg+2` | Nachrichtenaustausch über konfigurierbare Kanäle (siehe [Client-Einstellungen](Client-Settings-Guide)). |
| **Settings** | `Strg+3` | Geräte-ID/-Info, Kommunikationskanäle, Zoom- und Helligkeits-Voreinstellung. |
| **Feeds** | `Strg+4` | Live-Bild von Kamera-Geräten (Body-Cams, UGL-Cams, CCTV, Tracker) mit eigenem Pan/Zoom/Helligkeit/Lock. |

Zwei interne Zustände zusätzlich: **OFF** (Tablet aus) und **ON** (an, aber kein Modus gewählt — kurzer Übergang).

### Anzeige-Varianten (wie/wo das Tablet erscheint)

| Variante | Umschalten | Beschreibung |
|----------|-----------|--------------|
| **Hand (HANDtab)** | `Strg+F5` oder Herausnehmen aus dem Inventar | Tablet physisch in der Hand, mittelgroß, 3D vor dem Charakter. |
| **Big (BIGtab)** | `Shift+E` | Großer 2D-Vollbildschirm (~85 % des Bildschirms). Maus-Cursor automatisch aktiv. Serverseitig abschaltbar. |
| **Mini (MINITab)** | `Shift+Q` | Kleine Ecken-Anzeige im Arma-3-CTab-Stil. Serverseitig abschaltbar. |
| **PaperMap (PAPERMap)** | Spiel-Kartentaste `M` | Normale Vollbildkarte, sofern am Prefab und serverseitig erlaubt. |
| **GameMaster (GMmode)** | automatisch | Wird erkannt, wenn der GM-Editor offen ist; eigene GM-Darstellung. |

Der Wechsel Big ↔ Mini läuft über einen kurzen Zwischenschritt (ON-Zustand + 100 ms Verzögerung). Das ist gewollt und verhindert einen Engine-internen Kartenladefehler.

### Blue Force Tracking (BFT)

SMX ist Teil eines BFT-Systems: sichtbare Geräte anderer Spieler erscheinen als Marker auf der Karte. Sichtbarkeit ist kanalbasiert — ein fremdes Gerät wird angezeigt, wenn seine **Sendekanäle** mit den **Empfangskanälen** deines Tabletts überschneiden. Neu geprüft etwa einmal pro Sekunde.

## 2. Bedienung

1. SMX-Tablet (oder K23-Armhalterung) ins Inventar / an die Weste bringen.
2. **In die Hand nehmen:** `Strg+F5`. Wenn aus, mit `Strg+F1` **einschalten** (2 s Hochfahr-Timer).
3. Modus mit `Strg+1..4` wählen.
4. In GPS / Feeds: Zoom mit `Bild-auf` / `Bild-ab`, verschieben mit `Rechts-Strg + Pfeiltasten`, Helligkeit mit `Strg+Bild-auf` / `Strg+Bild-ab`.
5. Maus-Cursor mit `Alt+Strg` in Hand-/Mini-Modus umschalten (in Big immer an). Linksklick wirkt im Cursor-Modus.
6. Größere/kleinere Ansicht mit `Shift+E` (Big) / `Shift+Q` (Mini).
7. **Settings-Modus** (`Strg+3`) öffnen für Geräte-ID, Name und Kommunikationskanäle; **Save** drücken.

### K23-Armhalterung

Die K23 wird an der Weste getragen und hat einen eigenen Kontext, unabhängig vom Tablet-Gadget. `Shift+T` klappt die K23-Halterung ein/aus. Der Server kann Mini/Big/PaperMap **getrennt** für „K23 lose getragen" vs. „K23 an Weste montiert" erlauben/sperren.

## 3. Tastenbelegung

Vollständige Tabelle auf der Seite [Tastenbelegung](Keybinds) (Kategorie **SkyMap-X** unter *Einstellungen → Tastenbelegung*).

## 4. Client-Einstellungen

Es gibt **zwei** Stellen zum Konfigurieren von SkyMap-X — beide pro Spieler, beide in `$profile:SMX_settings_{playerID}.bin` gespeichert:

- **A. Der Settings-Modus des Tabletts** (`Strg+3` am Tablet) — Kommunikationskanäle, Geräte-ID/-Info, Zoom, Helligkeit. Speichern über den **Save**-Button.
- **B. Das normale Spiel-Einstellungsmenü → Reiter „SkyMap-X"** (`SMX_UISettingsSubMenu`) — Verhaltens- und Darstellungsoptionen. Speichert automatisch bei jeder Änderung.

### A. Tablet-Settings-Modus (`Strg+3`)

| Option | Wirkung |
|--------|---------|
| Geräte-ID | Freitext-ID des eigenen Tabletts; für den BFT-Marker genutzt. |
| Geräte-Info | Freitext-Anzeigename; Standard = eigener Spielername. |
| Subkanal 1–4 (Senden) | Vier frei benennbare Textkanäle zum Senden; je Kanal an/aus. |
| Subkanal 1–4 (Empfangen) | Dieselben vier Kanäle zum Empfangen; je Kanal an/aus. |
| CIV Senden/Empfangen | Kommunikation auf dem zivilen Kanal an/aus. |
| Fraktion Senden/Empfangen | Kommunikation auf dem Fraktions-/Team-Kanal (standardmäßig an; beim Gruppenbeitritt automatisch mit dem Gruppennamen befüllt). |
| Zoom (Slider) | Aktueller/Standard-Kartenzoom. |
| Helligkeit (Slider) | Bildschirmhelligkeit in festen Stufen. |
| „Kanal zufällig zurücksetzen"-Button | **WIP** — Platzhalter, aktuell ohne Funktion. |

### B. Spiel-Einstellungen → Reiter „SkyMap-X"

| Option | Wirkung |
|--------|---------|
| Auto-Start | Tablet schaltet sich beim Missionsstart automatisch ein. |
| Auto-Modus | Tablet öffnet automatisch einen Modus statt im ON-Zustand zu bleiben. |
| Standard-Modus | In welchem Inhalts-Modus das Tablet öffnet (GPS / Chat / Settings / Feeds). |
| Präfix nutzen + Main-TAG | Persönliches Präfix/Tag (Freitext) auf dem BFT-Marker anzeigen; registriert das Gerät bei Änderung live neu. |
| Benachrichtigungston-Dauer | Länge des Chat-/Benachrichtigungstons. |
| Mini-Modus Offset hoch / rechts / Größenskala | Position und Größe der Mini-Ecken-Anzeige. |
| Squad-Mate-Tracker: Skala / Name zeigen / Icon zeigen / Transparenz | Darstellung der Squad-Mate-Marker auf der Karte. |
| Trace Line: R / G / B / Alpha / Breite | Farbe, Deckkraft und Breite der Distanz-/Trace-Linie auf der Karte. |
| Tablet-Farbe (ATAK) | Tablet-Skin-Farbe aus `SMX_ColorTextureConfig` („Default" + konfigurierte Einträge). |
| K23-Halterungs-Farbe | Skin-Farbe der getragenen K23-Halterung, gleiche Farbliste. |

## 5. Server-Einstellungen

Serverseitig verwaltet (`$profile:SMX_ConfigV1.Json`, nur vom Server geladen und an alle Clients repliziert). Siehe [Server- & Admin-Handbuch](Server-Admin-Guide).

| Flag | Wirkung |
|------|---------|
| Mini-Modus erlauben | Schaltet `Shift+Q` / Mini global frei/gesperrt. |
| Big-Modus erlauben | Schaltet `Shift+E` / Big global frei/gesperrt. |
| Papierkarte erlauben | Schaltet die `M`-Vollbildkarte über das Tablet frei/gesperrt. |
| K23-Varianten der drei obigen Flags | Getrennt für „K23 lose getragen" vs. „K23 in Weste montiert". |
| BFT: Geräte-ID anzeigen | Ob der BFT-Marker die Geräte-ID zeigt (Standard: an). |
| BFT: Geräte-Info anzeigen | Ob der BFT-Marker die Geräte-Info zeigt (Standard: aus). |
| BFT: Spielername anzeigen | Ob der BFT-Marker den echten Spielernamen zeigt (Standard: aus). |
| `m_bBFT_ShowAllSquadLeaderMarkers` | Alle Squad-Leader-Marker auf der SMX-Minimap zeigen statt unterdrücken. |

## 6. Bekannte Baustellen / Einschränkungen

- **Quick-Marker** (`Strg+F4`): nur Code-Stub, setzt aktuell keinen Marker (blendet nur die UI aus).
- **Center / Lock-On** (`Strg+F2` / `Strg+F3`): im Keybind-Menü als „WIP" eingestuft; die Funktionen arbeiten (Zentrieren + Sperren auf Spielerposition), ein Code-Kommentar markiert sie aber als überarbeitungsbedürftig.
- **Gedrückt-halten-Wiederholung für die On-Screen-Feed-Buttons** (Pan/Zoom/Helligkeit per Maus-Klick-und-Halten im Feed-Modus) ist aktuell defekt — nur Einzelklicks. Tastatursteuerung nicht betroffen.
- **BFT-Tracker ignoriert Power-Status** (offenes TODO): lose oder ausgeschaltete Tablets bekommen ab Missionsstart einen BFT-Marker; normaler Off ruft kein Unregister.

## 7. Architektur (kurz)

`SCR_GadgetComponent` → `SMX_MainComponentX` (Haupt-Hub: Widget-State, Modus-/Tab-Zustandsautomat `ESMX_Modes` / `ESMX_TabModes`, Settings) mit Singleton-Managern: `SMX_DeviceManager`, `SMX_ReplicationManager`, `SMX_MenuManager`, `SMX_TabletMarkerManager`. Hand/Mini/Big rendern als reine Workspace-Widgets (kein `ChimeraMenuBase`) und teilen den Modus-Wechsel-Pfad; Big-Modus routet Klicks über ein eigenes Hit-Test-System. Details: die `CLAUDE.md` im Repo.
