# SpeedUI (SUI)

> Source: `SpeedUI` · Enfusion / Enforce Script. Verified against `scripts/Game/TAG_SpeedHUD.c`, `scripts/Game/SUI_SpeedUISettings.c`.

## 1. What it is

A lightweight HUD that shows the current **player or vehicle speed** as a numeric readout plus a progress bar (speed relative to a max speed). Works in every game mode and for multiple players at once.

- Numeric readout, e.g. `45.3 km/h`
- Progress bar (`SpeedPlayer` on foot / `SpeedCar` in a vehicle)
- Units: km/h, m/s, kn (knots), mph
- Show/hide is controlled entirely through the **Speed UI settings submenu** (see §3) — there is no toggle keybind.

## 2. How to use it

1. The `TAG_SpeedHUD` component must be on the player prefab. On this repo it is attached to `DefaultPlayerController.et`. To add it to another player prefab: open the prefab in Workbench → *Inspector → Components → + Add → `TAG_SpeedHUD`* → save.
2. Enable/disable the bar and the numeric readout (separately for on‑foot and vehicle) in *Settings → Speed UI*.
3. On entering a vehicle the HUD switches to the vehicle bar/units automatically (vehicle entry is detected via `SCR_CompartmentAccessComponent.GetVehicle()`).

Technical: on‑foot speed from `CharacterControllerComponent.GetVelocity()`, vehicle speed from `vehicleEntity.GetPhysics().GetVelocity()`, converted m/s → km/h (× 3.6). Update loop runs every 50 ms via `CallLater` (not `EOnFrame`, which is unreliable in a `ScriptComponent`).

## 3. Client settings

In‑game settings submenu **"Speed UI"** (`SUI_SpeedUISettings.layout`, 6 spinboxes + transparency). Persisted immediately on every change to `$profile:SpeedUI_Settings.txt` (one value per line, in this order):

| Line | Setting | Values |
|------|---------|--------|
| 0 | `SpeedUI_Enable_Bar` | 0/1 — show progress bar (on foot) |
| 1 | `SpeedUI_Enable_Units` | 0/1 — show numeric readout (on foot) |
| 2 | `SpeedUI_Units` | `0`=km/h, `1`=m/s, `2`=kn, `3`=mph (on foot) |
| 3 | `SpeedUI_Enable_Bar_v` | 0/1 — show progress bar (vehicle) |
| 4 | `SpeedUI_Enable_Units_v` | 0/1 — show numeric readout (vehicle) |
| 5 | `SpeedUI_Units_v` | unit index (vehicle) |
| 6 | `SpeedUI_HUD_Transparency` | HUD transparency step |

### Max speed (bar scale)

Set in `TAG_SpeedHUD.c`: `private float m_MaxSpeed = 50.0;` — or at runtime `speedHUD.SetMaxSpeed(100.0);`.

## 4. Server settings

SpeedUI has **no server‑side configuration** — it is a purely client‑side HUD. The only server‑side requirement is that the `TAG_SpeedHUD` component exists on the player prefab used by the mission.

## 5. Troubleshooting

| Symptom | Check |
|---------|-------|
| HUD not showing | Is the game mode assigned correctly? Is `TAG_SpeedHUD` on the player prefab? Does `TAG_SpeedHUD.c` reference the correct layout GUID? Are the bar/units enabled in *Settings → Speed UI*? Check script logs. |

## 6. Ideas / possible extensions

Average speed, per‑vehicle max‑speed scaling, sound feedback, configurable HUD position/size.

---

# SpeedUI (SUI) — Deutsch

> Quelle: `SpeedUI` · Enfusion / Enforce Script. Verifiziert gegen `scripts/Game/TAG_SpeedHUD.c`, `scripts/Game/SUI_SpeedUISettings.c`.

## 1. Kurzbeschreibung

Ein leichtes HUD, das die aktuelle **Spieler- oder Fahrzeuggeschwindigkeit** als Zahlenwert plus Fortschrittsbalken anzeigt (Geschwindigkeit relativ zu einer Maximalgeschwindigkeit). Funktioniert in jedem Spielmodus und für mehrere Spieler gleichzeitig.

- Zahlenanzeige, z. B. `45.3 km/h`
- Fortschrittsbalken (`SpeedPlayer` zu Fuß / `SpeedCar` im Fahrzeug)
- Einheiten: km/h, m/s, kn (Knoten), mph
- Ein-/Ausblenden läuft komplett über das **Speed-UI-Einstellungsuntermenü** (siehe §3) — es gibt keine Umschalt-Taste.

## 2. Bedienung

1. Die Komponente `TAG_SpeedHUD` muss am Player-Prefab hängen. In diesem Repo ist sie an `DefaultPlayerController.et`. Für ein anderes Player-Prefab: Prefab in der Workbench öffnen → *Inspector → Components → + Add → `TAG_SpeedHUD`* → speichern.
2. Balken und Zahlenanzeige (getrennt für zu Fuß und Fahrzeug) unter *Einstellungen → Speed UI* aktivieren/deaktivieren.
3. Beim Einsteigen in ein Fahrzeug wechselt das HUD automatisch auf Fahrzeug-Balken/-Einheiten (Fahrzeugerkennung über `SCR_CompartmentAccessComponent.GetVehicle()`).

Technisch: Geschwindigkeit zu Fuß aus `CharacterControllerComponent.GetVelocity()`, im Fahrzeug aus `vehicleEntity.GetPhysics().GetVelocity()`, Umrechnung m/s → km/h (× 3,6). Update-Schleife alle 50 ms über `CallLater` (nicht `EOnFrame`, das in einer `ScriptComponent` unzuverlässig ist).

## 3. Client-Einstellungen

In-Game-Untermenü **„Speed UI"** (`SUI_SpeedUISettings.layout`, 6 Spinboxen + Transparenz). Bei jeder Änderung sofort in `$profile:SpeedUI_Settings.txt` gespeichert (ein Wert pro Zeile, in dieser Reihenfolge):

| Zeile | Einstellung | Werte |
|-------|-------------|-------|
| 0 | `SpeedUI_Enable_Bar` | 0/1 — Fortschrittsbalken (zu Fuß) |
| 1 | `SpeedUI_Enable_Units` | 0/1 — Zahlenanzeige (zu Fuß) |
| 2 | `SpeedUI_Units` | `0`=km/h, `1`=m/s, `2`=kn, `3`=mph (zu Fuß) |
| 3 | `SpeedUI_Enable_Bar_v` | 0/1 — Fortschrittsbalken (Fahrzeug) |
| 4 | `SpeedUI_Enable_Units_v` | 0/1 — Zahlenanzeige (Fahrzeug) |
| 5 | `SpeedUI_Units_v` | Einheiten-Index (Fahrzeug) |
| 6 | `SpeedUI_HUD_Transparency` | HUD-Transparenzstufe |

### Maximalgeschwindigkeit (Balken-Skala)

In `TAG_SpeedHUD.c`: `private float m_MaxSpeed = 50.0;` — oder zur Laufzeit `speedHUD.SetMaxSpeed(100.0);`.

## 4. Server-Einstellungen

SpeedUI hat **keine serverseitige Konfiguration** — es ist ein reines Client-HUD. Serverseitig ist nur nötig, dass die Komponente `TAG_SpeedHUD` am von der Mission genutzten Player-Prefab vorhanden ist.

## 5. Fehlerbehebung

| Symptom | Prüfen |
|---------|--------|
| HUD wird nicht angezeigt | GameMode korrekt zugewiesen? `TAG_SpeedHUD` am Player-Prefab? Verweist `TAG_SpeedHUD.c` auf die richtige Layout-GUID? Balken/Einheiten unter *Einstellungen → Speed UI* aktiviert? Script-Logs prüfen. |

## 6. Ideen / mögliche Erweiterungen

Durchschnittsgeschwindigkeit, fahrzeugabhängige Max-Speed-Skalierung, Sound-Feedback, anpassbare HUD-Position/-Größe.
