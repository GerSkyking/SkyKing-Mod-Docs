# Client Settings Guide

Per‑player settings for the SkyKing mods, stored in your **player profile** and not controlled by the server.

## SkyMap-X

Two surfaces, both per‑player, both saved to `$profile:SMX_settings_{playerID}.bin` (binary, mission‑persistent). Full detail: [SkyMap-X §4](SkyMap-X).

**A. Tablet Settings mode** (`Ctrl+3` on the tablet, press **Save**):

| Setting | What it does |
|---------|--------------|
| Device ID | Free‑text ID of your tablet; used on the BFT marker (if the server shows it). |
| Device info | Free‑text display name; default = your player name. |
| Sub‑channel 1–4 (send) | Four freely named text channels for **sending**; each on/off. |
| Sub‑channel 1–4 (receive) | Same four channels for **receiving**; each on/off. Determines which other devices' BFT markers / feeds you see. |
| CIV send / receive | Civilian channel comms on/off. |
| Faction send / receive | Faction/team channel comms on/off (on by default; auto‑set to the group name when you join a group). |
| Zoom slider | Default map zoom. |
| Brightness slider | Tablet screen brightness (fixed steps). |
| "Randomly reset channel" button | **WIP** — no function yet. |

**B. Game Settings → "SkyMap-X" tab** (saved automatically on each change):

| Setting | What it does |
|---------|--------------|
| Auto start / Auto mode / Default mode | Tablet powers on / opens a mode automatically at mission start; which mode it opens in. |
| Use prefix + Main TAG | Personal prefix prepended to your Device ID (e.g. `"Alpha-1 " + Device ID`). Shows up wherever your Device ID is listed to others: the Chat contact list, the Feeds device list, and the BFT marker (only if the server has `bftShowDeviceID` on). |
| Notification‑tone duration | Length of the chat/notification sound. |
| Mini‑mode offset up / right / size scale | Mini corner display position and size. |
| Squad‑Mate‑Tracker: scale / show name / show icon / transparency | Squad‑mate marker appearance. |
| Trace line R / G / B / alpha / width | Distance/trace line color, opacity, width. |
| Tablet color (ATAK) / K23 mount color | Skin colors from `SMX_ColorTextureConfig`. |

## SpeedUI

In‑game settings submenu **"Speed UI"** (6 spinboxes + transparency). Saved immediately on every change to `$profile:SpeedUI_Settings.txt` (one value per line).

| Setting | Values |
|---------|--------|
| Show bar (on foot) | on/off |
| Show units (on foot) | on/off |
| Units (on foot) | km/h · m/s · kn · mph |
| Show bar (vehicle) | on/off |
| Show units (vehicle) | on/off |
| Units (vehicle) | km/h · m/s · kn · mph |
| HUD transparency | step value |

There is no toggle keybind — hide the HUD by disabling both "show bar" and "show units".

## SIDC-Framework

Client settings live in the player profile (`SIDC_PlayerProfile`) and the settings menu:

| Setting | What it does |
|---------|--------------|
| Communication channel | Your active channel (Side / Group / Command / Air / Artillery / Infantry / BFT). Changed via the map channel UI. Default = **Side**. |
| Physical channel (`SpinBox_PhysicalMarkerChannel`) | `Map` (default) / `ATAK` / `All`. "All" only offered if the server allows it (`allowAllPhysicalChannel`). |
| Export markers (`SpinBox_ExportMarker`) | Client‑side periodic write of your placed markers to JSON. Independent of the server export. |
| Marker sync interval (`m_iMarkerSyncIntervalMs`) | Interval for the client‑side player‑data / marker export. |
| Personal marker role | Your SIDC "Personnel" role icon (EOD, FO, JFO, MP, observer, sniper, SOF, marksman, medic, signaler, recon, infantry, close protection, riot control, SWAT, demolition, commander, 2IC, rifle, auto‑rifle, HMG, heavy GL, mortar, AT launcher). |
| Favorites | Favorite quick markers (`SIDC_FavoriteMarkerStore` / `SIDC_ListBoxFavoriteStore`). |
| Drawing color / width / opacity | Line & phase‑line style (`SIDC_PhaseLineColorConfig`, `SIDC_PhaseLineWidthConfig`). |

Undo/redo history depth and drawing point spacing are read from `SIDC_ServerDefaultValues.conf` (`m_iMaxUndoHistorySize` = 10, `m_iMaxRedoHistorySize` = 10, `m_fDrawingPointSpacingMeters` = 5) — see [Server & Admin Guide](Server-Admin-Guide).

## Throw Back Grenade

**No client settings** and no in‑game menu. See [Throw Back Grenade](Throw-Back-Grenade).

---

# Client-Einstellungen (Deutsch)

Pro-Spieler-Einstellungen der SkyKing-Mods, im **Spielerprofil** gespeichert und nicht vom Server gesteuert.

## SkyMap-X

Zwei Stellen, beide pro Spieler, beide in `$profile:SMX_settings_{playerID}.bin` gespeichert (binär, missionspersistent). Details: [SkyMap-X §4](SkyMap-X).

**A. Tablet-Settings-Modus** (`Strg+3` am Tablet, **Save** drücken):

| Option | Funktion |
|--------|----------|
| Geräte-ID | Freitext-ID des Tabletts; auf dem BFT-Marker genutzt (falls der Server ihn zeigt). |
| Geräte-Info | Freitext-Anzeigename; Standard = Spielername. |
| Subkanal 1–4 (Senden) | Vier frei benennbare Textkanäle zum **Senden**; je an/aus. |
| Subkanal 1–4 (Empfangen) | Dieselben vier Kanäle zum **Empfangen**; je an/aus. Bestimmt, welche BFT-Marker / Feeds anderer Geräte du siehst. |
| CIV Senden/Empfangen | Ziviler Kanal an/aus. |
| Fraktion Senden/Empfangen | Fraktions-/Team-Kanal an/aus (standardmäßig an; beim Gruppenbeitritt automatisch auf den Gruppennamen gesetzt). |
| Zoom-Slider | Standard-Kartenzoom. |
| Helligkeits-Slider | Bildschirmhelligkeit (feste Stufen). |
| „Kanal zufällig zurücksetzen"-Button | **WIP** — noch ohne Funktion. |

**B. Spiel-Einstellungen → Reiter „SkyMap-X"** (speichert automatisch bei jeder Änderung):

| Option | Funktion |
|--------|----------|
| Auto-Start / Auto-Modus / Standard-Modus | Tablet schaltet sich ein / öffnet automatisch einen Modus beim Missionsstart; in welchem Modus es öffnet. |
| Präfix nutzen + Main-TAG | Persönliches Präfix vor deiner Geräte-ID (z.B. `"Alpha-1 " + Geräte-ID`). Erscheint überall, wo deine Geräte-ID anderen angezeigt wird: Chat-Kontaktliste, Feeds-Geräteliste und BFT-Marker (nur wenn der Server `bftShowDeviceID` aktiviert hat). |
| Benachrichtigungston-Dauer | Länge des Chat-/Benachrichtigungstons. |
| Mini-Modus Offset hoch / rechts / Größenskala | Position und Größe der Mini-Ecken-Anzeige. |
| Squad-Mate-Tracker: Skala / Name / Icon / Transparenz | Darstellung der Squad-Mate-Marker. |
| Trace Line R / G / B / Alpha / Breite | Farbe, Deckkraft, Breite der Distanz-/Trace-Linie. |
| Tablet-Farbe (ATAK) / K23-Halterungs-Farbe | Skin-Farben aus `SMX_ColorTextureConfig`. |

## SpeedUI

In-Game-Untermenü **„Speed UI"** (6 Spinboxen + Transparenz). Bei jeder Änderung sofort in `$profile:SpeedUI_Settings.txt` gespeichert (ein Wert pro Zeile).

| Option | Werte |
|--------|-------|
| Balken zeigen (zu Fuß) | an/aus |
| Einheiten zeigen (zu Fuß) | an/aus |
| Einheit (zu Fuß) | km/h · m/s · kn · mph |
| Balken zeigen (Fahrzeug) | an/aus |
| Einheiten zeigen (Fahrzeug) | an/aus |
| Einheit (Fahrzeug) | km/h · m/s · kn · mph |
| HUD-Transparenz | Stufenwert |

Es gibt keine Umschalt-Taste — das HUD ausblenden, indem „Balken zeigen" und „Einheiten zeigen" beide deaktiviert werden.

## SIDC-Framework

Client-Einstellungen im Spielerprofil (`SIDC_PlayerProfile`) und im Einstellungsmenü:

| Option | Funktion |
|--------|----------|
| Kommunikationskanal | Dein aktiver Kanal (Side / Group / Command / Air / Artillery / Infantry / BFT). Über die Karten-Kanal-UI geändert. Standard = **Side**. |
| Physischer Kanal (`SpinBox_PhysicalMarkerChannel`) | `Map` (Standard) / `ATAK` / `All`. „All" nur angeboten, wenn der Server es erlaubt (`allowAllPhysicalChannel`). |
| Marker exportieren (`SpinBox_ExportMarker`) | Clientseitiges periodisches Schreiben deiner platzierten Marker als JSON. Unabhängig vom Server-Export. |
| Marker-Sync-Intervall (`m_iMarkerSyncIntervalMs`) | Intervall für den clientseitigen Spielerdaten-/Marker-Export. |
| Persönliche Marker-Rolle | Dein SIDC-„Personnel"-Rollen-Icon (EOD, FO, JFO, MP, Beobachter, Scharfschütze, SOF, Schütze, Sanitäter, Funker, Aufklärer, Infanterie, Close Protection, Riot Control, SWAT, Demolition, Kommandeur, 2IC, Gewehr, Automatikgewehr, HMG, schwerer GL, Mörser, PzF). |
| Favoriten | Favorisierte Quick-Marker (`SIDC_FavoriteMarkerStore` / `SIDC_ListBoxFavoriteStore`). |
| Zeichnen-Farbe / -Breite / -Deckkraft | Linien- & Phase-Line-Stil (`SIDC_PhaseLineColorConfig`, `SIDC_PhaseLineWidthConfig`). |

Undo-/Redo-Historie-Tiefe und Zeichenpunkt-Abstand kommen aus `SIDC_ServerDefaultValues.conf` (`m_iMaxUndoHistorySize` = 10, `m_iMaxRedoHistorySize` = 10, `m_fDrawingPointSpacingMeters` = 5) — siehe [Server- & Admin-Handbuch](Server-Admin-Guide).

## Throw Back Grenade

**Keine Client-Einstellungen** und kein In-Game-Menü. Siehe [Throw Back Grenade](Throw-Back-Grenade).
