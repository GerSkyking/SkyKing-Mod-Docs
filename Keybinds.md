# Keybinds — all SkyKing mods

All keys are **default bindings** and are rebindable in *Settings → Keybindings* under the category named below. Names in parentheses are the raw config input constants (`KC_*` = US‑QWERTY scancode positions, **not** layout‑aware).

## SkyMap-X — category "SkyMap-X"

| Key (default) | Action | Usable in | Note |
|---|---|---|---|
| `Ctrl + F5` | Take tablet into hand / stow | ONtab ↔ HANDtab | With the K23 mount it only flips a flag, no physical stow |
| `Ctrl + F1` | Power the tablet on | OFFtab | Starts the start‑up timer (`m_fStartUpTime`, default 2000 ms) |
| `Alt + Ctrl` | Toggle mouse cursor | HANDtab, MINITab | Auto‑on in BIGtab, no manual toggle needed |
| `Left mouse` | Click in cursor mode | HANDtab, MINITab, BIGtab | Repeats while held (every 250 ms) |
| `Ctrl + 1` | Mode: GPS | HAND/MINI/BIG tab | |
| `Ctrl + 2` | Mode: Chat | HAND/MINI/BIG tab | |
| `Ctrl + 3` | Mode: Settings | HAND/MINI/BIG tab | |
| `Ctrl + 4` | Mode: Feeds | HAND/MINI/BIG tab | |
| `Shift + T` | Fold/unfold K23 mount | K23 only (on vest) | Own context, independent of the tablet gadget |
| `Shift + Q` | Toggle Mini mode | ONtab, HANDtab, BIGtab | Server‑disableable (`m_bIsTabMiniAllow`) |
| `Shift + E` | Toggle Big mode (85 % screen) | ONtab, HANDtab, MINITab | Server‑disableable (`m_bIsTabBigAllow`) |
| `Page Down` | Zoom map/feed in | GPS map or Feeds camera | |
| `Page Up` | Zoom map/feed out | GPS map or Feeds camera | |
| `Ctrl + Page Up` | Brightness up | anywhere (in Feeds: camera image) | |
| `Ctrl + Page Down` | Brightness down | anywhere (in Feeds: camera image) | |
| `Right‑Ctrl + Arrow Up/Right/Down/Left` | Pan map/feed N/E/S/W | GPS or Feeds | Repeats while held |
| `Ctrl + F4` | Set quick marker | anywhere | **WIP** — currently only hides the UI, sets no marker |
| `Ctrl + F3` | Lock map to player position (lock‑on) | anywhere | Flagged "WIP"; works but marked as needing rework |
| `Ctrl + F2` | Center map on player position | anywhere | Same status as lock‑on; currently behaves identically to it, kept as a separate action on purpose |

**Feeds mode (keyboard):**
- `Right‑Ctrl + Arrow keys` swivel the selected camera (5° per step by default, one step every 100 ms while held). Ignored while the camera is locked. Synced to other players like the on‑screen buttons.
- `Page Down` / `Page Up` zoom the camera one step per key press (no repeat while held — same as on the map). Works in Hand, Mini and Big mode, also with the mouse cursor active.

**Not in the "SkyMap-X" category but relevant:**
- `M` (game "GadgetMap" key) opens the PAPERMap tab mode if the paper map is allowed (`m_bIsTabNormalPaperMapAllowOnPrefab`).
- `Esc` ("MenuBackKeybind") fully exits cursor mode in Big mode (closes Big mode); otherwise it only disables the cursor.

> **Update note (Sept 2026):** the internal action names of the SkyMap-X keybinds were renamed (typo fix `Devive` → `Device`, `Curser` → `Cursor`). Reforger stores custom bindings by action name, so **customised SkyMap-X keybinds are reset to their defaults once** — re‑bind them under *Settings → Keybindings → SkyMap-X*. Default keys are unchanged.

## SpeedUI

SpeedUI has **no keybinds**. The HUD is shown/hidden through *Settings → Speed UI*. See [SpeedUI](SpeedUI).

## SIDC-Framework — category "SIDC"

Active on the **open map** (`MapContext`) and in the SIDC sub‑menus.

| Key (default) | Action | Context(s) |
|---|---|---|
| `Ctrl + T` (`KC_LCONTROL` + `KC_T`) | Open Quick‑Marker menu (`SIDC_Add_Marker_UI`) | MapContext, debug‑marker menu |
| `Ctrl + D` (`KC_LCONTROL` + `KC_D`) | Toggle line‑drawing mode (`SIDC_Framework_Drawing`) | MapContext, drawing menu |
| `Left mouse button` (`mouse:button0`) | Place a line point (`SIDC_Framework_Drawing_Draw`) — 2 points = segment, more = chain | drawing menu |
| `Shift` (`KC_LSHIFT`) | Bridge / chain marker (`SIDC_Bridge_Marker`) | MapContext, drawing & debug menu |
| `I` (`KC_I`) | Inspect marker under cursor (`SIDC_Inspect_Marker`) | MapContext, inspection menu |
| `Ctrl + S` (`KC_LCONTROL` + `KC_S`) | Save marker set (`SIDC_Framework_Save_Marker`) | MapContext, save menu — server‑gated |
| `Ctrl + L` (`KC_LCONTROL` + `KC_L`) | Load marker set (`SIDC_Framework_Load_Marker`) | MapContext, load menu — server‑gated |
| `Ctrl + Z` — bound as `KC_LCONTROL` + `KC_Y` * | Undo (`SIDC_Framework_Undo`) | MapContext, drawing menu |
| `Ctrl + Y` — bound as `KC_LCONTROL` + `KC_Z` * | Redo (`SIDC_Framework_Redo`) | MapContext, drawing menu |
| `X` (`KC_X`) | Clipboard copy (`SIDC_Framework_X`) | MapContext, debug menu |
| `C` (`KC_C`) | Clipboard paste (`SIDC_Framework_C`) | MapContext, debug menu |
| `V` (`KC_V`) | Clipboard action (`SIDC_Framework_V`) | MapContext, debug menu |
| `Shift + T` (`KC_LSHIFT` + `KC_T`) | Debug: change player side (`SIDC_Debug_Change_Side_Player`) | MapContext |

\* **QWERTZ note:** `KC_*` constants map to physical **US‑QWERTY** positions. On a German keyboard Y and Z are swapped, so scancode `KC_Y` is the "Z"‑labelled key and `KC_Z` is the "Y"‑labelled key. Undo binds `KC_Y` and Redo binds `KC_Z` — deliberately, so they sit on the German "Z" and "Y" keys (project's target audience), not the US‑QWERTY scancode positions.

### SIDC menu contexts (priority 66, `MenuBack`/`MenuBackKeybind`/`MenuBackHold` = leave menu)

`SIDC_FrameworkDebugMarkerMenu`, `SIDC_Framework_Inspection_Menu`, `SIDC_Framework_LoadMarker_Menu`, `SIDC_Framework_SaveMarker_Menu`, `SIDC_Framework_Drawing_Menu`.

## Throw Back Grenade — category "TBG"

| Key (default) | Action | Context | Note |
|---|---|---|---|
| `Ctrl + X` (`KC_LCONTROL` + `KC_X`) | Throw the carried grenade (`TBG_Throw`, "Throw Granade") | `TBG_Action` (priority 100) | The normal grenade‑throw input also throws it |

Picking up a grenade is a **world action** ("Pick Up and Throw Grenade"), not a keybind — use your normal interact key. See [Throw Back Grenade](Throw-Back-Grenade).

---

# Tastenbelegung — alle SkyKing-Mods (Deutsch)

Alle Tasten sind **Standardbelegungen** und unter *Einstellungen → Tastenbelegung* in der unten genannten Kategorie frei änderbar. Namen in Klammern sind die rohen Config-Konstanten (`KC_*` = US-QWERTY-Scancode-Positionen, **nicht** layoutbewusst).

## SkyMap-X — Kategorie „SkyMap-X"

| Taste (Standard) | Aktion | Nutzbar in | Anmerkung |
|---|---|---|---|
| `Strg + F5` | Tablet in die Hand nehmen / wegstecken | ONtab ↔ HANDtab | Bei K23-Halterung nur ein Flag, kein physisches Verstauen |
| `Strg + F1` | Tablet einschalten | OFFtab | Startet den Hochfahr-Timer (`m_fStartUpTime`, Standard 2000 ms) |
| `Alt + Strg` | Maus-Cursor umschalten | HANDtab, MINITab | In BIGtab automatisch aktiv |
| `Maustaste links` | Klick im Cursor-Modus | HANDtab, MINITab, BIGtab | Feuert wiederholt bei Halten (alle 250 ms) |
| `Strg + 1` | Modus: GPS | HAND/MINI/BIG-Tab | |
| `Strg + 2` | Modus: Chat | HAND/MINI/BIG-Tab | |
| `Strg + 3` | Modus: Settings | HAND/MINI/BIG-Tab | |
| `Strg + 4` | Modus: Feeds | HAND/MINI/BIG-Tab | |
| `Shift + T` | K23-Halterung ein-/ausklappen | nur K23 (an Weste) | Eigener Kontext, unabhängig vom Tablet-Gadget |
| `Shift + Q` | Mini-Modus umschalten | ONtab, HANDtab, BIGtab | Serverseitig abschaltbar (`m_bIsTabMiniAllow`) |
| `Shift + E` | Big-Modus umschalten (85 % Bildschirm) | ONtab, HANDtab, MINITab | Serverseitig abschaltbar (`m_bIsTabBigAllow`) |
| `Bild-ab` | Karte/Feed reinzoomen | GPS-Karte oder Feeds-Kamera | |
| `Bild-auf` | Karte/Feed rauszoomen | GPS-Karte oder Feeds-Kamera | |
| `Strg + Bild-auf` | Helligkeit hoch | überall (in Feeds: Kamerabild) | |
| `Strg + Bild-ab` | Helligkeit runter | überall (in Feeds: Kamerabild) | |
| `Rechts-Strg + Pfeil hoch/rechts/runter/links` | Karte/Feed nach N/O/S/W schieben | GPS oder Feeds | Reagiert auf Halten |
| `Strg + F4` | Quick-Marker setzen | überall | **WIP** — blendet aktuell nur die UI aus, setzt keinen Marker |
| `Strg + F3` | Karte auf Spielerposition sperren (Lock-On) | überall | Als „WIP" eingestuft; funktioniert, gilt als überarbeitungsbedürftig |
| `Strg + F2` | Karte auf Spielerposition zentrieren | überall | Gleiche Einstufung wie Lock-On; verhält sich aktuell identisch, ist aber bewusst eine eigene Aktion |

**Feeds-Modus (Tastatur):**
- `Rechts-Strg + Pfeiltasten` schwenken die ausgewählte Kamera (standardmäßig 5° pro Schritt, ein Schritt alle 100 ms beim Halten). Bei gesperrter Kamera wirkungslos. Wird wie die On-Screen-Buttons an andere Spieler synchronisiert.
- `Bild-ab` / `Bild-auf` zoomen die Kamera einen Schritt pro Tastendruck (keine Wiederholung beim Halten — wie auf der Karte). Funktioniert in Hand-, Mini- und Big-Modus, auch bei aktivem Maus-Cursor.

**Nicht in der Kategorie „SkyMap-X", aber relevant:**
- `M` (Spiel-„GadgetMap"-Taste) öffnet den PAPERMap-Modus, wenn die Papierkarte erlaubt ist (`m_bIsTabNormalPaperMapAllowOnPrefab`).
- `Esc` („MenuBackKeybind") verlässt im Big-Modus den Cursor-Modus vollständig (schließt Big-Modus); sonst deaktiviert es nur den Cursor.

> **Update-Hinweis (Sept 2026):** Die internen Action-Namen der SkyMap-X-Tastenbelegung wurden umbenannt (Tippfehler `Devive` → `Device`, `Curser` → `Cursor`). Reforger speichert eigene Belegungen unter dem Action-Namen, daher werden **angepasste SkyMap-X-Tasten einmalig auf Standard zurückgesetzt** — unter *Einstellungen → Tastenbelegung → SkyMap-X* neu belegen. Die Standardtasten bleiben gleich.

## SpeedUI

SpeedUI hat **keine Tastenbelegung**. Das HUD wird über *Einstellungen → Speed UI* ein-/ausgeblendet. Siehe [SpeedUI](SpeedUI).

## SIDC-Framework — Kategorie „SIDC"

Aktiv auf der **geöffneten Karte** (`MapContext`) und in den SIDC-Untermenüs.

| Taste (Standard) | Aktion | Kontext(e) |
|---|---|---|
| `Strg + T` (`KC_LCONTROL` + `KC_T`) | Quick-Marker-Menü öffnen (`SIDC_Add_Marker_UI`) | MapContext, Debug-Marker-Menü |
| `Strg + D` (`KC_LCONTROL` + `KC_D`) | Linien-Zeichnen-Modus umschalten (`SIDC_Framework_Drawing`) | MapContext, Zeichnen-Menü |
| `Linke Maustaste` (`mouse:button0`) | Linien-Punkt setzen (`SIDC_Framework_Drawing_Draw`) — 2 Punkte = Segment, mehr = Chain | Zeichnen-Menü |
| `Shift` (`KC_LSHIFT`) | Bridge / Chain-Marker (`SIDC_Bridge_Marker`) | MapContext, Zeichnen- & Debug-Menü |
| `I` (`KC_I`) | Marker unter Cursor inspizieren (`SIDC_Inspect_Marker`) | MapContext, Inspektions-Menü |
| `Strg + S` (`KC_LCONTROL` + `KC_S`) | Marker-Satz speichern (`SIDC_Framework_Save_Marker`) | MapContext, Speichern-Menü — server-gesteuert |
| `Strg + L` (`KC_LCONTROL` + `KC_L`) | Marker-Satz laden (`SIDC_Framework_Load_Marker`) | MapContext, Laden-Menü — server-gesteuert |
| `Strg + Z` — gebunden als `KC_LCONTROL` + `KC_Y` * | Rückgängig (`SIDC_Framework_Undo`) | MapContext, Zeichnen-Menü |
| `Strg + Y` — gebunden als `KC_LCONTROL` + `KC_Z` * | Wiederholen / Redo (`SIDC_Framework_Redo`) | MapContext, Zeichnen-Menü |
| `X` (`KC_X`) | Zwischenablage kopieren (`SIDC_Framework_X`) | MapContext, Debug-Menü |
| `C` (`KC_C`) | Zwischenablage einfügen (`SIDC_Framework_C`) | MapContext, Debug-Menü |
| `V` (`KC_V`) | Zwischenablage-Aktion (`SIDC_Framework_V`) | MapContext, Debug-Menü |
| `Shift + T` (`KC_LSHIFT` + `KC_T`) | Debug: Spieler-Seite wechseln (`SIDC_Debug_Change_Side_Player`) | MapContext |

\* **QWERTZ-Hinweis:** `KC_*`-Konstanten zeigen auf physische **US-QWERTY**-Positionen. Auf einer deutschen Tastatur sind Y und Z vertauscht: Scancode `KC_Y` ist die „Z"-Taste, `KC_Z` die „Y"-Taste. Undo bindet `KC_Y`, Redo bindet `KC_Z` — bewusst, damit sie auf der deutschen „Z"- und „Y"-Taste liegen (Zielgruppe des Projekts), nicht auf den US-QWERTY-Scancode-Positionen.

### SIDC-Menükontexte (Priorität 66, `MenuBack`/`MenuBackKeybind`/`MenuBackHold` = Menü verlassen)

`SIDC_FrameworkDebugMarkerMenu`, `SIDC_Framework_Inspection_Menu`, `SIDC_Framework_LoadMarker_Menu`, `SIDC_Framework_SaveMarker_Menu`, `SIDC_Framework_Drawing_Menu`.

## Throw Back Grenade — Kategorie „TBG"

| Taste (Standard) | Aktion | Kontext | Anmerkung |
|---|---|---|---|
| `Strg + X` (`KC_LCONTROL` + `KC_X`) | Getragene Granate werfen (`TBG_Throw`, „Throw Granade") | `TBG_Action` (Priorität 100) | Die normale Granaten-Wurftaste wirft sie ebenfalls |

Eine Granate aufheben ist eine **Welt-Aktion** („Pick Up and Throw Grenade"), keine Taste — normale Interaktionstaste nutzen. Siehe [Throw Back Grenade](Throw-Back-Grenade).
