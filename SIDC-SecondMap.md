# SIDC-SecondMap (SIDC DLC)

> Source: `C:\Users\Sky\Documents\GitHub\SIDC-SecondMap` · prefix `SIDC_SecondMap_` · Enfusion / Enforce Script. Verified against `Scripts/Game/SIDC_SecondMap/*.c`, `Configs/System/chimeraInputCommon.conf`, `Configs/Map/SIDC_SecondMap_Test.conf`, `UI/Layouts/SecondMap/*.layout`. **Status: work in progress / prototype**, not yet released. No dedicated repo yet — lives as its own addon (`addon.gproj`, ID `SIDCSecondMap`) alongside the SIDC-Framework family.

## 1. What it is

SIDC-SecondMap adds **one or more extra, persistent map views** that can stay on screen at the same time as the normal (vanilla) map — e.g. a small overview map that never closes, next to the full-screen map opened with `M`. It ships two different implementations of such a view:

- **`SIDC_SecondMapNativeView`** — reuses the engine's own native map renderer (full vanilla look: real building footprints, forest, contour lines from the `.topo`). Looks identical to the vanilla map, at the cost of a hard limitation (see §7).
- **`SIDC_SecondMapView`** — an independent view built purely from UI widgets (`ImageWidget` + `CanvasWidget`), with its own from-scratch renderer for roads, contour lines, buildings, trees, power lines and place-name/landmark labels. Fully independent of the vanilla map's internal state, and any number of instances can be placed and shown at once.

Both are `GenericEntity` prefabs placed directly in the world (category `SIDC/SecondMap` in the entity browser) and both plug into a shared **focus/input hub**, `SIDC_SecondMap_ViewManager`, so keybinds work the same regardless of which implementation is used.

## 2. How to use it (player)

- The normal map still opens/closes with **`M`** as always.
- **`,` / `.`** — cycle focus between the vanilla map slot ("BASE") and each placed extra view, in registration order.
- **`Ctrl+N`** — toggle the *currently focused* extra view open/closed **with vanilla map input** (`MapContext`). Has no effect while BASE is focused (use `M` for that).
- **`Ctrl+B`** — same, but **passive**: the view opens without activating `MapContext` (no player input), e.g. as a pure display.
- **Vanilla map controls** (focused view with input): right mouse hold = pan, WASD = pan, mouse wheel / `MapZoomIn/Out` = zoom, right click = radial menu, left click/drag = markers (see §8). The old arrow/PageUp/PageDown actions are legacy only.

"Focus" only controls which view receives these keys — it has nothing to do with which views are currently open. Any number of extra views may be open on screen simultaneously; only one of them (or BASE) is focused at a time.

## 3. Keybinds

Category **SIDC_SecondMap_Global** (`Configs/System/chimeraInputCommon.conf`, priority 62).

| Action | Default | Note |
|--------|---------|------|
| `SIDC_SecondMap_ToggleView` | `Ctrl + N` | Open/close the focused extra view |
| `SIDC_SecondMap_CycleNext` | `.` | Focus next view (BASE → view 1 → view 2 → … → BASE) |
| `SIDC_SecondMap_CyclePrevious` | `,` | Focus previous view |
| `SIDC_SecondMap_ToggleViewPassive` | `Ctrl + B` | Open/close the focused view without `MapContext` (passive) |
| `SIDC_SecondMap_DebugSwitchView` | `Ctrl + J` | Open the session debug menu (see §9) |
| `SIDC_SecondMap_ToggleCoControl` | `Ctrl + F5` | Release/lock the focused session for additional controllers (see §9). **Not** a dev keybind - always active (own context `SIDC_SecondMap_Core`) |
| `SIDC_SecondMap_PanUp/Down/Left/Right`, `ZoomIn/Out` | Arrows / PgUp / PgDn | Legacy, no longer needed (vanilla `MapContext` actions are used) |

**Dev keybinds:** all keys above except `Ctrl + F5` are *development helpers* (later everything is meant to be driven by script). They are switched off as a whole by the checkbox `ActivateDevKeybinds` (`m_bActivateDevKeybinds`, category "SIDC - Keybinds") in `Configs/System/SIDC_SecondMap_DebugMode.conf` - default **off**, and a missing `.conf` also means off. Changes take effect after a restart / Workbench play, not live. The map's own mouse interaction (pan, zoom, context menu, markers) is not affected.

The input context is only activated while at least one extra view is registered in the world, and is kept alive by a repeating timer (`ActivateContext` self-expires after ~5 s, refreshed every 500 ms).

## 4. Entities & attributes

| Entity | Attributes | Purpose |
|--------|------------|---------|
| `SIDC_SecondMapNativeViewClass` | `m_sLayout` (layout with a `MapWidget`), `m_sMapConfig` (`SCR_MapConfig`, used only if the vanilla map was never opened yet), `m_fInitialZoomPPU` | Vanilla-look view, one native renderer for the whole engine |
| `SIDC_SecondMapViewClass` | `m_sImageOverride` (fallback satellite image), `m_sLayout` (layout with `MapImage`/`Overlay` widgets), `m_fInitialZoom`, `m_sMapConfig` (`SCR_MapConfig`, e.g. `MapFullscreen.conf`), `m_iForceTreeIndividualVisibility` (default `100`; forces individual tree dots on regardless of what the map config says, `-1` = use the config value unchanged) | Independent custom-rendered view, any number placeable |

`m_sDisplayName` (on the shared base `SIDC_SecondMap_ViewBase`) is the name shown in focus-change log lines. Further base attributes: `m_bShowMarkers`, `m_bShowChannelInfo`, `m_sPhysicalChannel` (see §8).

## 5. Architecture (short)

```
SIDC_SecondMap_ViewManager (singleton)
├── tracks focus index (0 = BASE/vanilla, 1..N = registered views, in registration order)
├── registers/removes the 6 input actions above (Ctrl+N, ,/. , arrows, PgUp/PgDn)
└── routes keys to GetFocusedView() only

SIDC_SecondMap_ViewBase (GenericEntity)          — shared registration + display name
├── SIDC_SecondMapNativeView                     — drives the native SCR_MapEntity renderer directly
└── SIDC_SecondMapView                           — independent widget-based renderer
      ├── SIDC_SecondMap_TopoData  (static, per-world)  — parses the map's .topo file (roads w/ type, building footprints, tree point cloud)
      ├── SIDC_SecondMap_WorldScan (static, per-world)  — fallback source: scans Building/PowerlineEntity from the live world (used when .topo data is missing). For power lines it only gets each PowerlineEntity's own position (no real pole-connection data exists), then reconstructs the line network by greedily matching the globally shortest pole-to-pole pairs first, capped at 2 connections per pole (junction poles with 3 real connections lose one edge - visually cheaper than a wrong "return" line)
      ├── SIDC_SecondMap_Contours  (static, per-world)  — computes contour lines from terrain height via marching squares, tiled + spread over frames
      └── SIDC_SecondMap_MapStyle  (per-view instance)  — reads a vanilla SCR_MapConfig (layer zoom bounds, road/building/tree/grid styling, descriptor visibility) so the custom renderer matches the normal map's look
```

`SIDC_SecondMap_TopoData`, `_WorldScan` and `_Contours` are all **static and shared across every open `SIDC_SecondMapView`** — parsed/scanned once per world, not once per view.

### Key files

| File | Purpose |
|------|---------|
| `SIDC_SecondMap_ViewManager.c` | Focus tracking + the 6 input actions, shared by all view types |
| `SIDC_SecondMap_ViewBase.c` | Common base entity: display name, register/unregister with the manager |
| `SIDC_SecondMapNativeView.c` | Vanilla-renderer view: pan/zoom against `SCR_MapEntity`, hides itself while the real map is open and restores its own state afterward |
| `SIDC_SecondMapView.c` | Custom widget-based view: image + overlay drawing, road/building/tree/landmark/label rendering (~1300 lines) |
| `SIDC_SecondMap_TopoData.c` | Binary `.topo` chunk parser (`ROAD`, `AREA`, `BULD` chunks) |
| `SIDC_SecondMap_WorldScan.c` | Live-world fallback scan for buildings/power lines, tiled across frames |
| `SIDC_SecondMap_Contours.c` | Marching-squares contour line generator from `BaseWorld.GetSurfaceY`, tiled across frames |
| `SIDC_SecondMap_MapStyle.c` | Reads a vanilla `SCR_MapConfig` (layers, road/prop styling, descriptor visibility) for the custom renderer |

### Naming conventions

`SIDC_SecondMap_` classes/files (entities drop the trailing underscore: `SIDC_SecondMapView`, `SIDC_SecondMapNativeView`) · members: `m_a` array, `m_s` string, `m_f` float, `m_i` int, `m_b` bool · `protected` for internal state and helpers, only the base-class contract (`IsOpen`/`ToggleOpen`/`ZoomStep`/`GetDisplayName`) and the manager's public API are exposed.

## 6. Known behaviour / limitations

- **`SIDC_SecondMapNativeView` cannot be shown at the same time as the vanilla map.** The engine's native map renderer (`SCR_MapEntity`/`MapWidget`) is global — a second view sharing it would fight the vanilla map over pan/zoom/visible layer. Verified by test (2026-09-26): a second `SCR_MapEntity` instance moves/empties the normal map. The workaround implemented here: the native view remembers its own pan/zoom, hides the moment the vanilla map opens (`SCR_MapEntity.GetOnMapInit`), hands its pan/zoom back to the vanilla map on open (`GetOnMapOpen`), and resumes with its own state after the vanilla map closes (`GetOnMapClose`).
- **`SIDC_SecondMapView` exists specifically to work around that limitation** — it never touches the native renderer, so any number of instances can be open simultaneously, together with the vanilla map. The cost is a from-scratch renderer that has to re-implement road/building/tree/label drawing and styling from the map config itself.
- The `.topo` file has no readable contour-line data — contour lines are always computed at runtime from terrain height (`SIDC_SecondMap_Contours`), not read from the file.
- Landmark icons only cover a curated allow-list of descriptor types (`IsLandmarkType`) — trees, bushes, individual rocks, fences etc. are intentionally excluded (they would blanket forested terrain).
- If a map's satellite background image can't be read automatically from the map entity's prefab data, `m_sImageOverride` must be set manually (a warning is logged when this happens).
- Only one `SIDC_SecondMapNativeView` makes sense per level (it drives the one global native renderer); any number of `SIDC_SecondMapView` instances can coexist.
- Power lines are drawn as an approximation, not real topology: the engine only exposes each `PowerlineEntity`'s own placement, not which poles it actually connects. A pole with 3 real connections (a junction) will visibly lose one line, since every pole is capped at 2.
- Individual tree dots are forced on via `m_iForceTreeIndividualVisibility` (default `100`) because the vanilla `MapFullscreen.conf` has that particular sub-layer set to 0 (off) - vanilla only shows forest as a filled area, never as individual points.
- `SIDC_SecondMapView` now also shows: a grid on/off + a scale-bar legend (both read from `SCR_MapConfig.m_bEnableGrid`/`m_bEnableLegendScale`), and every map marker read-only from `SCR_MapMarkerManagerComponent`. Static markers reuse the real marker layout + `SCR_MapMarkerWidgetComponent.InitClientSettings` (same code the vanilla map uses), so they look identical and work with any mod's custom marker types; only the on-screen position is computed by this view itself. Marker placing/editing and drawing were added later — see §8. Ruler, watch and the tools menu are still not implemented.

## 7. Test setup in this repo

`World/2 MapEntitys Test.ent` + `Configs/Map/SIDC_SecondMap_Test.conf` — a test level with two map entities, used to verify the "native renderer is global" limitation above. `UI/Layouts/SecondMap/SIDC_SecondMap_TestLayout.layout` and the two production layouts (`SIDC_SecondMapView.layout` for the native view's `MapWidget`, `SIDC_SecondMapImageView.layout` for the custom view's `MapImage`/`Overlay`) are the layout contracts each view type expects (see §4).

## 8. SIDC-Framework integration (markers, radial menu, API)

Both views depend on **SIDC-Framework** and render its markers with their own layer (`SIDC_SecondMap_MarkerLayer`, reads `SCR_MapMarkerManagerComponent`, incl. disabled markers, edge culling, sync ≤ 0.1 s while drawing / 0.5 s otherwise).

- **Markers:** SIDC markers (layouts, phase lines, drawings), vanilla static markers (custom/military: show, move, delete, edit, hover shows type icons) and dynamic markers (squad leader/member; any other type from the marker config gets its layout, position updates every frame, text/icon updates only for squad leader/member).
- **Interaction (view with input):** right click = radial menu with all framework entries + category "Marker" (custom/military dialogs, own `MenuBase` menus in `SIDC_SecondMap_VanillaMarkerMenus`, presets in `Configs/System/chimeraMenus.conf`) + "delete marker"; left click on marker = edit, left drag = move, `MapMarkerDelete` = delete, `I` = inspect, hover = marker info, Shift+click chain = drawing, all framework keybinds work in the focused view.
- **Physical channel per view:** script (`SetPhysicalChannel` / `OpenWithChannel`) > attribute `m_sPhysicalChannel` > default entry of `SIDC_PhysicalChannelConfig`. Only markers of that channel are visible and new markers/drawings preselect it. Framework side: `SIDC_MapHost` (`Map/SIDC_MapHost.c`, external cursor/channel source set via `SIDC_MapHost.SetExternal`) and the `viewerPhysicalChannelOverride` parameter of `SIDC_ChannelVisibilityResolver.ResolvePercent`.
- **Script API** (only the two extra views, not the vanilla map; no auto-focus on open): `Open/OpenWithInput(bool)/OpenWithChannel(bool, string)/Close`, `IsReady/IsInteractive/IsMapInputEnabled`, `CenterOnWorld/GetCenterWorld/PanByPixels/ZoomTo/ZoomBy/GetZoom/ZoomStep`, `StartDrag/UpdateDrag/EndDrag`, `WorldToViewPixels/ViewPixelsToWorld/GetCursorViewPixels/GetCursorWorldPosition/GetViewSizePixels`, `GetHoveredMarkerObject/DeleteMarkerObject`, `GetPhysicalChannel/SetPhysicalChannel/IsChannelInfoEnabled`.
- **Not supported:** vanilla radial entries bound to the open vanilla map (commands, transport, recon), hover/click popups for dynamic markers, zoom-layer text hiding.
- **Status:** vanilla marker creation, dynamic markers and the new menus are not yet tested in-game (`chimeraMenus.conf` may need a Workbench restart).
- Repo README: `SIDC-SecondMap/README.md` (German).

## 9. Multiplayer: sessions, controllers and viewers

> Status: implemented, not yet released. The component **`SIDC_SecondMap_ReplicationManager`** must be added to the **GameMode** (prefab/entity) in the Workbench - without it everything runs locally and without sync.

Several players can look at the same view: some **control** it (pan/zoom), others **watch** passively or **co-control**.

**Terms**

| Term | Meaning |
|------|---------|
| Session | Every time a player opens a view *with map input* (`Ctrl+N`), a **new session** with its own ID is created (server counter 1, 2, 3 …), independent of other players. A session ends when its last controller leaves. |
| Controller | Controls the session. Any number per session. The creator is the **owner** and first controller. |
| Viewer | Watches passively (no map input), follows the controllers' pan/zoom. |
| Co-control ("frei") | Controllers released the session for additional controllers (`Ctrl+F5`). The first controller is always free; further ones may only join while released. |
| `isLiveView` | `true` while more than one participant (controllers + viewers) is present. Pan/zoom is only sent then. |

**Overlay:** `SIDC_FullscreenInfoChannels.layout` shows `ViewID` (session ID of this view, `?` until the server has replicated it) and `ViewControler` (name(s) of the controller(s), comma-separated).

**Debug menu** (`Ctrl+J`, `SIDC_SecondMap_DebugSwitchView.layout`): list of all running sessions as `Name | ID n | Controller(s) | Owner [| frei]`.

| Button | Action |
|--------|--------|
| `btn_checkView` | Watch the selected session passively (opens the local view with the same key, follows the controllers) |
| `btn_checkViewAsController` | Join as an additional controller - only if the session is released (`Ctrl+F5`), otherwise the menu stays open and the reason is logged |
| `btn_back`, `btn_close`, `Esc` | Close the menu |

**Sync rules**
- Transferred: view centre (world x/z) + **visible width in metres** (not pixels per metre), so different resolutions show the same section.
- Only while `isLiveView`, only on change, about 10 times per second. Controller → server → all other participants.
- Simultaneous panning: the server locks a session for 400 ms for the controller who is currently changing it and drops other controllers' states meanwhile (the blocked controller is overwritten by the lock holder's state).
- A receiving client never sends the received state back (no echo). A new co-controller receives the others' current state instead of sending its own.
- Disconnect: the player leaves all sessions; a session without a controller is removed.

**Manual steps in the Workbench:** add the component to the GameMode; create the button `btn_checkViewAsController` in `SIDC_SecondMap_DebugSwitchView.layout`; add the strings `SIDC-UI-btn_back`, `SIDC-UI-btn_close`, `SIDC-UI-btn_checkViewAsController` to the string table.

**Files:** `SIDC_SecondMap_ReplicationManager.c` (GameMode component + modded `SCR_PlayerController` for all client→server RPCs), `SIDC_SecondMap_ViewManager.c` (local sync tick, join/watch API), `SIDC_SecondMap_ViewBase.c` (session ID, role, sync state), `SIDC_SecondMap_DebugSwitchViewMenu.c` (debug menu).

---

# SIDC-SecondMap (SIDC-DLC) — Deutsch

> Quelle: `C:\Users\Sky\Documents\GitHub\SIDC-SecondMap` · Präfix `SIDC_SecondMap_` · Enfusion / Enforce Script. Verifiziert gegen `Scripts/Game/SIDC_SecondMap/*.c`, `Configs/System/chimeraInputCommon.conf`, `Configs/Map/SIDC_SecondMap_Test.conf`, `UI/Layouts/SecondMap/*.layout`. **Status: in Arbeit / Prototyp**, noch nicht veröffentlicht. Noch kein eigenes Repo — läuft als eigenes Addon (`addon.gproj`, ID `SIDCSecondMap`) neben der SIDC-Framework-Familie.

## 1. Kurzbeschreibung

SIDC-SecondMap fügt **eine oder mehrere zusätzliche, dauerhaft offene Kartenansichten** hinzu, die gleichzeitig mit der normalen (Vanilla-)Karte angezeigt werden können — z.B. eine kleine Übersichtskarte, die nie schließt, neben der Vollbild-Karte, die man mit `M` öffnet. Es gibt zwei verschiedene Umsetzungen einer solchen Ansicht:

- **`SIDC_SecondMapNativeView`** — nutzt den nativen Kartenrenderer der Engine mit (vollem Vanilla-Look: echte Gebäude-Grundrisse, Wald, Höhenlinien aus der `.topo`). Sieht identisch zur Vanilla-Karte aus, hat dafür eine harte Einschränkung (siehe §7).
- **`SIDC_SecondMapView`** — eine eigenständige, rein auf UI-Widgets basierende Ansicht (`ImageWidget` + `CanvasWidget`), mit eigenem Renderer für Straßen, Höhenlinien, Gebäude, Bäume, Stromleitungen und Orts-/Landmarken-Beschriftungen. Komplett unabhängig vom internen Zustand der Vanilla-Karte, beliebig viele Instanzen gleichzeitig möglich.

Beide sind `GenericEntity`-Prefabs, die direkt in die Welt platziert werden (Kategorie `SIDC/SecondMap` im Entity-Browser), und beide hängen an einem gemeinsamen **Fokus-/Tasten-Hub**, `SIDC_SecondMap_ViewManager`, damit die Tastenbelegung für beide Umsetzungen gleich funktioniert.

## 2. Bedienung (Spieler)

- Die normale Karte öffnet/schließt weiterhin wie gewohnt mit **`M`**.
- **`,` / `.`** — Fokus zwischen dem Vanilla-Karten-Slot ("BASE") und jeder platzierten Zusatz-Ansicht wechseln (in Registrierungsreihenfolge).
- **`Strg+N`** — die *gerade fokussierte* Zusatz-Ansicht auf-/zuklappen, **mit Vanilla-Map-Input** (`MapContext`). Wirkt sich nicht aus, während BASE fokussiert ist (dafür `M` benutzen).
- **`Strg+B`** — dasselbe, aber **passiv**: die Ansicht öffnet ohne `MapContext` (kein Spieler-Input), z.B. als reine Anzeige.
- **Vanilla-Map-Steuerung** (fokussierte Ansicht mit Input): rechte Maustaste halten = verschieben, WASD = verschieben, Mausrad / `MapZoomIn/Out` = zoomen, Rechtsklick = Radialmenü, Linksklick/Ziehen = Marker (siehe §8). Die alten Pfeil-/Bild-auf/-ab-Aktionen sind nur noch Legacy.

"Fokus" bestimmt nur, welche Ansicht diese Tasten bekommt — hat nichts damit zu tun, welche Ansichten offen sind. Beliebig viele Zusatz-Ansichten dürfen gleichzeitig offen sein, immer nur eine (oder BASE) ist fokussiert.

## 3. Tastenbelegung

Kategorie **SIDC_SecondMap_Global** (`Configs/System/chimeraInputCommon.conf`, Priorität 62).

| Aktion | Standard | Anmerkung |
|--------|----------|-----------|
| `SIDC_SecondMap_ToggleView` | `Strg + N` | Fokussierte Zusatz-Ansicht auf-/zuklappen |
| `SIDC_SecondMap_CycleNext` | `.` | Nächste Ansicht fokussieren (BASE → Ansicht 1 → Ansicht 2 → … → BASE) |
| `SIDC_SecondMap_CyclePrevious` | `,` | Vorherige Ansicht fokussieren |
| `SIDC_SecondMap_ToggleViewPassive` | `Strg + B` | Fokussierte Ansicht ohne `MapContext` auf-/zuklappen (passiv) |
| `SIDC_SecondMap_DebugSwitchView` | `Strg + J` | Session-Debug-Menü öffnen (siehe §9) |
| `SIDC_SecondMap_ToggleCoControl` | `Strg + F5` | Fokussierte Session für weitere Controller freigeben/sperren (siehe §9). **Kein** Dev-Keybind - immer aktiv (eigener Context `SIDC_SecondMap_Core`) |
| `SIDC_SecondMap_PanUp/Down/Left/Right`, `ZoomIn/Out` | Pfeile / Bild auf / Bild ab | Legacy, nicht mehr nötig (es gelten die Vanilla-`MapContext`-Aktionen) |

**Dev-Keybinds:** alle Tasten oben außer `Strg + F5` sind *Entwicklungshilfen* (später läuft alles per Script). Sie werden gemeinsam über die Checkbox `ActivateDevKeybinds` (`m_bActivateDevKeybinds`, Kategorie „SIDC - Keybinds“) in `Configs/System/SIDC_SecondMap_DebugMode.conf` abgeschaltet - Default **aus**, fehlt die `.conf`, ist es ebenfalls aus. Änderungen wirken erst nach Neustart / Workbench-Play, nicht live. Die Maus-Interaktion der Karte selbst (Verschieben, Zoom, Kontextmenü, Marker) ist davon nicht betroffen.

Der Input-Context wird nur aktiviert, solange mindestens eine Zusatz-Ansicht in der Welt registriert ist, und durch einen wiederholenden Timer am Leben gehalten (`ActivateContext` läuft nach ~5 s selbst ab, wird alle 500 ms erneuert).

## 4. Entitäten & Attribute

| Entität | Attribute | Zweck |
|---------|-----------|-------|
| `SIDC_SecondMapNativeViewClass` | `m_sLayout` (Layout mit `MapWidget`), `m_sMapConfig` (`SCR_MapConfig`, nur genutzt falls die Vanilla-Karte noch nie offen war), `m_fInitialZoomPPU` | Ansicht im Vanilla-Look, ein nativer Renderer für die ganze Engine |
| `SIDC_SecondMapViewClass` | `m_sImageOverride` (Ersatz-Satellitenbild), `m_sLayout` (Layout mit `MapImage`/`Overlay`-Widgets), `m_fInitialZoom`, `m_sMapConfig` (`SCR_MapConfig`, z.B. `MapFullscreen.conf`), `m_iForceTreeIndividualVisibility` (Default `100`; erzwingt einzelne Baum-Punkte unabhängig vom Wert der Map-Config, `-1` = Config-Wert unverändert übernehmen) | Eigenständige, selbst gerenderte Ansicht, beliebig oft platzierbar |

`m_sDisplayName` (auf der gemeinsamen Basis `SIDC_SecondMap_ViewBase`) ist der Name, der in den Fokus-Wechsel-Log-Zeilen erscheint. Weitere Basis-Attribute: `m_bShowMarkers`, `m_bShowChannelInfo`, `m_sPhysicalChannel` (siehe §8).

## 5. Architektur (kurz)

```
SIDC_SecondMap_ViewManager (Singleton)
├── verwaltet den Fokus-Index (0 = BASE/Vanilla, 1..N = registrierte Ansichten, Registrierungsreihenfolge)
├── registriert/entfernt die 6 Input-Actions oben (Strg+N, ,/. , Pfeile, Bild-auf/-ab)
└── leitet Tasten nur an GetFocusedView() weiter

SIDC_SecondMap_ViewBase (GenericEntity)          — gemeinsame Registrierung + Anzeigename
├── SIDC_SecondMapNativeView                     — steuert den nativen SCR_MapEntity-Renderer direkt an
└── SIDC_SecondMapView                           — eigenständiger, widget-basierter Renderer
      ├── SIDC_SecondMap_TopoData  (statisch, pro Welt)  — parst die .topo-Datei der Karte (Straßen mit Typ, Gebäude-Grundrisse, Baum-Punktwolke)
      ├── SIDC_SecondMap_WorldScan (statisch, pro Welt)  — Ersatzquelle: scannt Building/PowerlineEntity aus der laufenden Welt (falls .topo-Daten fehlen). Fuer Stromleitungen gibt es nur die eigene Position jeder PowerlineEntity (keine echte Anschluss-Topologie) - das Leitungsnetz wird per gierigem globalem Matching rekonstruiert (kuerzeste Mast-Paare zuerst, hoechstens 2 Verbindungen pro Mast; Kreuzungsmaste mit 3 echten Verbindungen verlieren dadurch eine Linie - optisch guenstiger als eine falsche "Rueckweg"-Linie)
      ├── SIDC_SecondMap_Contours  (statisch, pro Welt)  — berechnet Höhenlinien aus der Geländehöhe per Marching Squares, gekachelt und über mehrere Frames verteilt
      └── SIDC_SecondMap_MapStyle  (pro Ansichts-Instanz) — liest eine Vanilla-SCR_MapConfig (Layer-Zoomgrenzen, Straßen-/Gebäude-/Baum-/Raster-Stil, Descriptor-Sichtbarkeit), damit der eigene Renderer wie die normale Karte aussieht
```

`SIDC_SecondMap_TopoData`, `_WorldScan` und `_Contours` sind alle **statisch und von jeder offenen `SIDC_SecondMapView` gemeinsam genutzt** — einmal pro Welt geparst/gescannt, nicht einmal pro Ansicht.

### Wichtige Dateien

| Datei | Zweck |
|-------|-------|
| `SIDC_SecondMap_ViewManager.c` | Fokus-Verwaltung + die 6 Input-Actions, von allen Ansichtstypen geteilt |
| `SIDC_SecondMap_ViewBase.c` | Gemeinsame Basis-Entität: Anzeigename, An-/Abmeldung beim Manager |
| `SIDC_SecondMapNativeView.c` | Vanilla-Renderer-Ansicht: Pan/Zoom gegen `SCR_MapEntity`, blendet sich aus solange die echte Karte offen ist und stellt danach ihren eigenen Stand wieder her |
| `SIDC_SecondMapView.c` | Widget-basierte Ansicht: Bild + Overlay-Zeichnung, Straßen-/Gebäude-/Baum-/Landmarken-/Label-Rendering (~1300 Zeilen) |
| `SIDC_SecondMap_TopoData.c` | Binärer `.topo`-Chunk-Parser (`ROAD`-, `AREA`-, `BULD`-Chunks) |
| `SIDC_SecondMap_WorldScan.c` | Live-Welt-Ersatzscan für Gebäude/Stromleitungen, über mehrere Frames gekachelt |
| `SIDC_SecondMap_Contours.c` | Höhenlinien-Generator (Marching Squares) aus `BaseWorld.GetSurfaceY`, über mehrere Frames gekachelt |
| `SIDC_SecondMap_MapStyle.c` | Liest eine Vanilla-`SCR_MapConfig` (Layer, Straßen-/Prop-Stil, Descriptor-Sichtbarkeit) für den eigenen Renderer |

### Namenskonventionen

`SIDC_SecondMap_`-Klassen/-Dateien (Entitäten ohne den letzten Unterstrich: `SIDC_SecondMapView`, `SIDC_SecondMapNativeView`) · Member: `m_a` Array, `m_s` String, `m_f` Float, `m_i` Int, `m_b` Bool · `protected` für internen Zustand/Hilfsfunktionen, nur der Basisklassen-Vertrag (`IsOpen`/`ToggleOpen`/`ZoomStep`/`GetDisplayName`) und die öffentliche API des Managers sind nach außen sichtbar.

## 6. Bekanntes Verhalten / Einschränkungen

- **`SIDC_SecondMapNativeView` kann nicht gleichzeitig mit der Vanilla-Karte angezeigt werden.** Der native Kartenrenderer der Engine (`SCR_MapEntity`/`MapWidget`) ist global — eine zweite Ansicht, die ihn mitbenutzt, würde mit der Vanilla-Karte um Pan/Zoom/sichtbaren Layer konkurrieren. Per Test verifiziert (2026-09-26): eine zweite `SCR_MapEntity`-Instanz verschiebt/leert die normale Karte. Die hier gebaute Lösung: die native Ansicht merkt sich ihren eigenen Pan/Zoom, blendet sich aus, sobald die Vanilla-Karte öffnet (`SCR_MapEntity.GetOnMapInit`), gibt der Vanilla-Karte beim Öffnen ihren eigenen Pan/Zoom zurück (`GetOnMapOpen`) und kommt nach deren Schließen mit ihrem eigenen Stand wieder (`GetOnMapClose`).
- **`SIDC_SecondMapView` existiert genau als Workaround für diese Einschränkung** — sie fasst den nativen Renderer nie an, daher können beliebig viele Instanzen gleichzeitig mit der Vanilla-Karte offen sein. Der Preis: ein komplett selbst gebauter Renderer, der Straßen-/Gebäude-/Baum-/Label-Zeichnung und -Stil aus der Map-Config selbst nachbilden muss.
- Die `.topo`-Datei enthält keine lesbaren Höhenlinien-Daten — Höhenlinien werden immer zur Laufzeit aus der Geländehöhe berechnet (`SIDC_SecondMap_Contours`), nie aus der Datei gelesen.
- Landmarken-Symbole decken nur eine kuratierte Positivliste von Descriptor-Typen ab (`IsLandmarkType`) — Bäume, Büsche, einzelne Felsen, Zäune usw. sind bewusst ausgenommen (die würden bewaldetes Gelände komplett zudecken).
- Kann das Satellitenbild einer Karte nicht automatisch aus den Prefab-Daten der Map-Entity gelesen werden, muss `m_sImageOverride` manuell gesetzt werden (dabei wird eine Warnung geloggt).
- Nur eine `SIDC_SecondMapNativeView` pro Level ist sinnvoll (sie steuert den einen globalen nativen Renderer); beliebig viele `SIDC_SecondMapView`-Instanzen können koexistieren.
- Stromleitungen sind eine Näherung, keine echte Topologie: die Engine liefert nur die Position jeder `PowerlineEntity`, nicht welche Maste sie tatsächlich verbindet. Ein Mast mit 3 echten Verbindungen (Kreuzung) verliert dadurch sichtbar eine Linie, da jeder Mast auf 2 begrenzt ist.
- Einzelne Baum-Punkte werden über `m_iForceTreeIndividualVisibility` (Default `100`) erzwungen, weil die Vanilla-`MapFullscreen.conf` diesen Sub-Layer auf 0 (aus) stehen hat - Vanilla zeigt Wald nur als gefüllte Fläche, nie als Einzelpunkte.
- `SIDC_SecondMapView` zeigt jetzt zusätzlich: Gitter an/aus + eine Maßstabs-Legende (beides aus `SCR_MapConfig.m_bEnableGrid`/`m_bEnableLegendScale`), sowie alle Kartenmarker rein lesend aus `SCR_MapMarkerManagerComponent`. Statische Marker nutzen dafür das echte Marker-Layout + `SCR_MapMarkerWidgetComponent.InitClientSettings` (dieselbe Logik wie die Vanilla-Karte), sehen also identisch aus und funktionieren mit jedem modeigenen Marker-Typ - nur die Bildschirmposition berechnet diese Ansicht selbst. Marker setzen/bearbeiten und Zeichnen kamen später dazu — siehe §8. Lineal, Uhr und das Werkzeug-Menü sind weiterhin nicht umgesetzt.

## 7. Test-Setup in diesem Repo

`World/2 MapEntitys Test.ent` + `Configs/Map/SIDC_SecondMap_Test.conf` — ein Test-Level mit zwei Map-Entitäten, verwendet um die oben beschriebene "nativer Renderer ist global"-Einschränkung zu verifizieren. `UI/Layouts/SecondMap/SIDC_SecondMap_TestLayout.layout` und die beiden Produktions-Layouts (`SIDC_SecondMapView.layout` für das `MapWidget` der nativen Ansicht, `SIDC_SecondMapImageView.layout` für `MapImage`/`Overlay` der eigenen Ansicht) sind die Layout-Verträge, die jeder Ansichtstyp erwartet (siehe §4).

## 8. SIDC-Framework-Anbindung (Marker, Radialmenü, API)

Beide Ansichten hängen von **SIDC-Framework** ab und zeigen dessen Marker mit einem eigenen Layer (`SIDC_SecondMap_MarkerLayer`, liest `SCR_MapMarkerManagerComponent` inkl. deaktivierter Marker, Culling am Rand, Abgleich ≤ 0,1 s beim Zeichnen / sonst 0,5 s).

- **Marker:** SIDC-Marker (Layouts, Phase Lines, Zeichnungen), statische Vanilla-Marker (Custom/Military: anzeigen, verschieben, löschen, bearbeiten, Hover zeigt Typ-Symbole) und dynamische Marker (Squad Leader/Member; jeder weitere Typ der Marker-Config bekommt sein Layout, Position jeden Frame, Text/Symbol-Updates nur für Squad Leader/Member).
- **Bedienung (Ansicht mit Input):** Rechtsklick = Radialmenü mit allen Framework-Einträgen + Kategorie „Marker" (Custom/Military-Dialoge, eigene `MenuBase`-Menüs in `SIDC_SecondMap_VanillaMarkerMenus`, Presets in `Configs/System/chimeraMenus.conf`) + „Marker löschen"; Linksklick auf Marker = bearbeiten, Linksziehen = verschieben, `MapMarkerDelete` = löschen, `I` = Inspect, Hover = Marker-Info, Shift+Klick-Kette = Zeichnen, alle Framework-Keybinds gehen in der fokussierten Ansicht.
- **Physischer Channel pro Ansicht:** Script (`SetPhysicalChannel` / `OpenWithChannel`) > Attribut `m_sPhysicalChannel` > Default-Eintrag der `SIDC_PhysicalChannelConfig`. Nur Marker dieses Channels sind sichtbar, neue Marker/Zeichnungen wählen ihn vor. Framework-Seite: `SIDC_MapHost` (`Map/SIDC_MapHost.c`, externe Cursor-/Channel-Quelle über `SIDC_MapHost.SetExternal`) und der Parameter `viewerPhysicalChannelOverride` in `SIDC_ChannelVisibilityResolver.ResolvePercent`.
- **Script-API** (nur die beiden Zusatz-Ansichten, nicht die Vanilla-Karte; kein Auto-Fokus beim Öffnen): `Open/OpenWithInput(bool)/OpenWithChannel(bool, string)/Close`, `IsReady/IsInteractive/IsMapInputEnabled`, `CenterOnWorld/GetCenterWorld/PanByPixels/ZoomTo/ZoomBy/GetZoom/ZoomStep`, `StartDrag/UpdateDrag/EndDrag`, `WorldToViewPixels/ViewPixelsToWorld/GetCursorViewPixels/GetCursorWorldPosition/GetViewSizePixels`, `GetHoveredMarkerObject/DeleteMarkerObject`, `GetPhysicalChannel/SetPhysicalChannel/IsChannelInfoEnabled`.
- **Nicht unterstützt:** Vanilla-Radialeinträge, die an der offenen Vanilla-Karte hängen (Kommandos, Transport, Recon), Hover-/Klick-Popups für dynamische Marker, Zoom-Layer-Textausblendung.
- **Status:** Vanilla-Marker-Erstellung, dynamische Marker und die neuen Menüs sind noch nicht im Spiel getestet (`chimeraMenus.conf` braucht ggf. einen Workbench-Neustart).
- Repo-README: `SIDC-SecondMap/README.md`.

## 9. Multiplayer: Sessions, Controller und Zuschauer

> Status: implementiert, noch nicht veröffentlicht. Die Component **`SIDC_SecondMap_ReplicationManager`** muss im Workbench am **GameMode** (Prefab/Entity) hinzugefügt werden - ohne sie läuft alles lokal und ohne Sync.

Mehrere Spieler können dieselbe Ansicht sehen: manche **steuern** sie (Pan/Zoom), andere **schauen passiv zu** oder **steuern mit**.

**Begriffe**

| Begriff | Bedeutung |
|---------|-----------|
| Session | Jedes Mal, wenn ein Spieler eine Ansicht *mit Map-Input* öffnet (`Strg+N`), entsteht eine **neue Session** mit eigener ID (Server-Zähler 1, 2, 3 …), unabhängig von anderen Spielern. Eine Session endet, wenn ihr letzter Controller geht. |
| Controller | Steuert die Session. Beliebig viele pro Session. Der Ersteller ist **Owner** und erster Controller. |
| Viewer | Schaut passiv zu (kein Map-Input), folgt Pan/Zoom der Controller. |
| Mit-Steuern („frei“) | Controller haben die Session für weitere Controller freigegeben (`Strg+F5`). Der erste Controller ist immer frei; weitere dürfen nur beitreten, solange freigegeben ist. |
| `isLiveView` | `true`, solange mehr als ein Teilnehmer (Controller + Viewer) da ist. Nur dann wird Pan/Zoom gesendet. |

**Overlay:** `SIDC_FullscreenInfoChannels.layout` zeigt `ViewID` (Session-ID dieser Ansicht, `?` bis der Server sie repliziert hat) und `ViewControler` (Name(n) der Controller, mit Komma getrennt).

**Debug-Menü** (`Strg+J`, `SIDC_SecondMap_DebugSwitchView.layout`): Liste aller laufenden Sessions als `Name | ID n | Controller | Owner [| frei]`.

| Button | Wirkung |
|--------|---------|
| `btn_checkView` | Gewählte Session passiv zuschauen (öffnet lokal die Ansicht mit demselben Schlüssel, folgt den Controllern) |
| `btn_checkViewAsController` | Als weiterer Controller beitreten - nur wenn die Session freigegeben ist (`Strg+F5`), sonst bleibt das Menü offen und der Grund steht im Log |
| `btn_back`, `btn_close`, `Esc` | Menü schließen |

**Sync-Regeln**
- Übertragen wird die Mitte der Ansicht (Welt x/z) + **sichtbare Breite in Metern** (nicht Pixel pro Meter), damit verschiedene Auflösungen denselben Ausschnitt zeigen.
- Nur bei `isLiveView`, nur bei Änderung, etwa 10-mal pro Sekunde. Controller → Server → alle anderen Teilnehmer.
- Gleichzeitiges Verschieben: der Server sperrt die Session 400 ms lang für den Controller, der gerade etwas ändert, und verwirft in der Zeit Stände anderer Controller (der blockierte Controller wird vom Stand des Sperrhalters überschrieben).
- Ein Empfänger schickt den übernommenen Stand nie zurück (kein Echo). Ein neuer Mit-Controller bekommt den aktuellen Stand der anderen, statt seinen eigenen zu senden.
- Disconnect: der Spieler verlässt alle Sessions; eine Session ohne Controller wird entfernt.

**Manuelle Schritte im Workbench:** Component am GameMode hinzufügen; Button `btn_checkViewAsController` in `SIDC_SecondMap_DebugSwitchView.layout` anlegen; Strings `SIDC-UI-btn_back`, `SIDC-UI-btn_close`, `SIDC-UI-btn_checkViewAsController` in die String-Tabelle eintragen.

**Dateien:** `SIDC_SecondMap_ReplicationManager.c` (GameMode-Component + modded `SCR_PlayerController` für alle Client→Server-RPCs), `SIDC_SecondMap_ViewManager.c` (lokaler Sync-Takt, Beitritts-/Zuschau-API), `SIDC_SecondMap_ViewBase.c` (Session-ID, Rolle, Sync-Zustand), `SIDC_SecondMap_DebugSwitchViewMenu.c` (Debug-Menü).
