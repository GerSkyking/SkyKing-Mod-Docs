# SkyMap-X — Camera functions (Fnc1 / Fnc2, night vision)

> Source: `SkyMap-X-DEV` → `SMX_ItemDeviceComponent.c`, `SMX_ItemDeviceCharComponent.c`, `SMX_ModeHelper_Feed.c`, `FeedDynamic.layout`.

## 1. What it is

In the **Feed** menu, every camera has two function buttons:

| Button | Function | Switched on by |
|---|---|---|
| **Fnc1** | **Tracking**: after a click, locking onto a vehicle or infantry follows that target | checkbox `m_bHasTracking` (default: on) |
| **Fnc2** | **NV**: night vision on the camera feed | checkbox `m_bHasNVG` (default: off) |

A function is only shown if the selected camera has it: the frame (`FrameLOText1` = Fnc1, `FrameROText1` = Fnc2) is hidden otherwise. Below each label a status text shows `activated` / `not active`.

## 2. Set up a camera (prefab)

All settings sit on `SMX_ItemDeviceComponent` of the camera prefab.

**Category "SCX - Camera"**

| Attribute | Meaning |
|---|---|
| `m_bHasTracking` | Camera has tracking (Fnc1) |
| `m_fCamZoomMin` / `m_fCamZoomMax` | Range of the zoom slider |
| `m_fCamBrightnessMin` / `Max` / `Default` | Range and start value of the brightness slider (EV) |

**Category "SCX - Camera NVG"**

| Attribute | Meaning |
|---|---|
| `m_bHasNVG` | Camera has night vision (Fnc2) |
| `m_rNVGHDRMaterial` | HDR material for NV (empty = default HDR stays) |
| `m_rNVGGrainMaterial` | Film grain material for NV (empty = default grain stays) |
| `m_fNVGBrightnessMin` / `Max` | Allowed brightness range **while NV is on** (default -1 … 1) |
| `m_fNVGBrightnessDefault` | Slider value when NV is switched on (default 0) |
| `m_fNVGBaseEV` | Fixed EV offset added to the slider while NV is on. The slider range is relative to it (default 0) |

### Night vision materials

NV uses two post-process materials, set on the camera of the feed (HDR priority 16, film grain priority 19, same as vanilla night vision sights):

- `Common/Postprocess/NightVision/NV_HDR_Cam.emat` — copy of the vanilla green phosphor look with **much lower gain** (`FilmicExposureBias`, `BloomIntensity`, `Contrast` reduced). The original `NV_HDR_GreenPhosphor.emat` burns out on a feed camera.
- `Common/Postprocess/NightVision/NV_FilmGrain.emat` — film grain.

The sight-component values of vanilla night vision (DOF, vignette, blur, objective FOV) are not used: only the two materials make the look.

### Brightness while NV is on

- The slider range switches to the NV range, the slider snaps back if dragged outside.
- On switching **on**, the current EV is remembered and the NV start value is set. On switching **off**, the old EV comes back.
- The effective camera EV is **slider value + `m_fNVGBaseEV`**.

## 3. Helmet camera (HeadCam)

For helmet cameras the device in the feed list is the **character** (`SMX_ItemDeviceCharComponent`), not the helmet item. When the helmet is worn, the item hands its settings to the character, replicated to all clients:

- **From the helmet item:** Tracking/NV checkboxes, NV materials and EV values, rotate yes/no, PIP camera prefab, yaw/pitch limits, rotate step, zoom (step/min/max), base FOV, brightness (step/min/max/default).
- **Stays on the character (position only):** head bone, invert bone pitch, `m_vCamStorageOffset`, start yaw, start pitch.

When the helmet is taken off, Fnc1 goes back to on and Fnc2 to off. Camera values stay until the next helmet is worn.

## 4. Sync (multiplayer)

The NV state travels with the normal camera state (yaw, pitch, zoom, brightness, lock, tracking) to all viewers. The NV materials are applied locally on each viewer's feed camera, also after the camera is re-created.

## 5. Known limits / planned

- Quality levels for NV (several material pairs, Fnc2 cycles through them) are planned, not built yet.
- The Fnc1/Fnc2 widgets exist in `FeedDynamic.layout` only.

---

# SkyMap-X — Kamera-Funktionen (Fnc1 / Fnc2, Nachtsicht)

> Quelle: `SkyMap-X-DEV` → `SMX_ItemDeviceComponent.c`, `SMX_ItemDeviceCharComponent.c`, `SMX_ModeHelper_Feed.c`, `FeedDynamic.layout`.

## 1. Kurzbeschreibung

Im **Feed**-Menü hat jede Kamera zwei Funktions-Buttons:

| Button | Funktion | Freigeschaltet durch |
|---|---|---|
| **Fnc1** | **Tracking**: nach einem Klick folgt ein Lock auf Fahrzeug oder Infanterie dem Ziel | Checkbox `m_bHasTracking` (Default: an) |
| **Fnc2** | **NV**: Nachtsicht auf dem Kamerabild | Checkbox `m_bHasNVG` (Default: aus) |

Eine Funktion wird nur angezeigt, wenn die gewählte Kamera sie hat: Der Frame (`FrameLOText1` = Fnc1, `FrameROText1` = Fnc2) wird sonst ausgeblendet. Unter jeder Beschriftung zeigt ein Status-Text `activated` / `not active`.

## 2. Kamera einrichten (Prefab)

Alle Einstellungen liegen auf der `SMX_ItemDeviceComponent` des Kamera-Prefabs.

**Kategorie „SCX - Camera"**

| Attribut | Bedeutung |
|---|---|
| `m_bHasTracking` | Kamera hat Tracking (Fnc1) |
| `m_fCamZoomMin` / `m_fCamZoomMax` | Bereich des Zoom-Sliders |
| `m_fCamBrightnessMin` / `Max` / `Default` | Bereich und Startwert des Helligkeits-Sliders (EV) |

**Kategorie „SCX - Camera NVG"**

| Attribut | Bedeutung |
|---|---|
| `m_bHasNVG` | Kamera hat Nachtsicht (Fnc2) |
| `m_rNVGHDRMaterial` | HDR-Material für NV (leer = Standard-HDR bleibt) |
| `m_rNVGGrainMaterial` | Film-Grain-Material für NV (leer = Standard-Grain bleibt) |
| `m_fNVGBrightnessMin` / `Max` | Erlaubter Helligkeitsbereich **bei aktivem NV** (Default -1 … 1) |
| `m_fNVGBrightnessDefault` | Slider-Wert beim Einschalten von NV (Default 0) |
| `m_fNVGBaseEV` | Fester EV-Offset, der bei aktivem NV zum Slider addiert wird. Der Slider-Bereich ist relativ dazu (Default 0) |

### Nachtsicht-Materialien

NV nutzt zwei Post-Process-Materialien, gesetzt auf der Kamera des Feeds (HDR-Priorität 16, Film Grain 19, wie bei den Vanilla-Nachtsicht-Visieren):

- `Common/Postprocess/NightVision/NV_HDR_Cam.emat` — Kopie des Vanilla-Grünphosphor-Looks mit **deutlich geringerer Verstärkung** (`FilmicExposureBias`, `BloomIntensity`, `Contrast` gesenkt). Das Original `NV_HDR_GreenPhosphor.emat` brennt auf einer Feed-Kamera aus.
- `Common/Postprocess/NightVision/NV_FilmGrain.emat` — Film Grain.

Die Sight-Werte der Vanilla-Nachtsicht (DOF, Vignette, Blur, Objective FOV) werden nicht benutzt: Den Look machen allein die beiden Materialien.

### Helligkeit bei aktivem NV

- Der Slider-Bereich wechselt auf den NV-Bereich. Zieht man darüber hinaus, springt der Slider zurück.
- Beim **Einschalten** wird der aktuelle EV gemerkt und der NV-Startwert gesetzt. Beim **Ausschalten** kommt der alte EV zurück.
- Der wirksame Kamera-EV ist **Slider-Wert + `m_fNVGBaseEV`**.

## 3. Helmkamera (HeadCam)

Bei Helmkameras ist das Gerät in der Feed-Liste der **Charakter** (`SMX_ItemDeviceCharComponent`), nicht das Helm-Item. Beim Tragen reicht das Item seine Einstellungen an den Charakter weiter, repliziert an alle Clients:

- **Vom Helm-Item:** Tracking-/NV-Checkboxen, NV-Materialien und -EV-Werte, schwenkbar ja/nein, PIP-Kamera-Prefab, Yaw-/Pitch-Limits, Rotations-Schritt, Zoom (Schritt/Min/Max), Basis-FOV, Helligkeit (Schritt/Min/Max/Default).
- **Bleibt am Charakter (nur Position):** Kopf-Bone, Bone-Pitch invertieren, `m_vCamStorageOffset`, Start-Yaw, Start-Pitch.

Beim Ablegen des Helms geht Fnc1 auf an und Fnc2 auf aus. Die Kamera-Werte bleiben bis zum nächsten Helm bestehen.

## 4. Sync (Multiplayer)

Der NV-Zustand läuft mit dem normalen Kamera-Zustand (Yaw, Pitch, Zoom, Helligkeit, Lock, Tracking) zu allen Zuschauern. Die NV-Materialien werden lokal auf der Feed-Kamera jedes Zuschauers gesetzt, auch nach dem Neuerzeugen der Kamera.

## 5. Bekannte Grenzen / geplant

- Qualitätsstufen für NV (mehrere Material-Paare, Fnc2 schaltet durch) sind geplant, noch nicht gebaut.
- Die Fnc1/Fnc2-Widgets gibt es nur in `FeedDynamic.layout`.
