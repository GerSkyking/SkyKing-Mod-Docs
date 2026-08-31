# Throw Back Grenade (TBG)

> Source: `…/ArmaReforgerWorkbench/addons/Throw Back Grenade` · prefix `TBG_` · Enfusion / Enforce Script. Verified against `Scripts/Game/TBG_*.c`, `Configs/System/*.conf`, `Prefabs/Weapons/Grenades/*.et`.

## 1. What it is

Throw Back Grenade lets a player **pick up a live, already‑thrown grenade and immediately throw it back** (or somewhere else) before it goes off. It works for frag grenades and smoke grenades.

- A "**Pick Up and Throw Grenade**" world action appears on a live grenade lying on the ground.
- On pickup the grenade is put into the character's hand, a **dummy inventory copy** is spawned into the throwable slot and auto‑equipped, and the player throws it with the normal throw input.
- The carried grenade **follows the dummy's throw arc** via a hidden "flight proxy" so the real (still‑ticking) grenade lands where you aimed, then detonates / smokes on its own timer.
- If the grenade expires while still in your hand (smoke runs out, it explodes), the dummy is removed from your inventory automatically.

### Supported grenades (prefabs shipped with the mod)

| Prefab | Type |
|--------|------|
| `Grenade_M67.et` | Frag (US) |
| `Grenade_RGD5.et` | Frag (USSR) |
| `Smoke_ANM8HC.et` | Smoke (HC white) |
| `Smoke_RDG2.et` | Smoke (USSR) |
| `Smoke_M18_Base.et` | Smoke (M18, colored) |

Any grenade prefab gets support by adding two components (see §5).

## 2. How to use it (player)

1. An enemy (or friendly) grenade lands near you and is still live.
2. Look at it — the **"Pick Up and Throw Grenade"** action shows (only if a normal "Pick Up" action is *not* available for that item and the grenade is not already attached to a character).
3. Perform the action. The grenade goes into your hand and is auto‑selected in the throwable slot.
4. **Throw** it with your normal grenade throw input, or with **`Ctrl+X`** (`TBG_Throw`, default). Aim first — the live grenade follows where the throw lands.
5. If you instead switch to another quick‑slot weapon while holding it, the grenade is **dropped** at your feet (it does not vanish).

⚠️ The grenade keeps counting down the whole time. Picking one up is a gamble.

## 3. Keybinds

Category **TBG** in *Settings → Keybindings*.

| Action | Default | Context | Note |
|--------|---------|---------|------|
| `TBG_Throw` ("Throw Granade") | `Ctrl + X` (`KC_LCONTROL` + `KC_X`) | `TBG_Action` (priority 100) | Throw the currently carried grenade. The normal grenade‑throw input also works. |

The pickup itself is a **world action** ("Pick Up and Throw Grenade"), not a keybind — use your normal interact key.

## 4. Settings

### Client settings

None. There is no in‑game settings menu for TBG.

### Server settings

No dedicated config file. `Configs/Editor/TBG_DebugMode.conf` (`TBG_DebugConfig`) only gates debug logging (`TBG_DebugConfig.Log(...)`). Tunable constants live in the scripts:

| Constant | File | Default | Meaning |
|----------|------|---------|---------|
| `DETACH_MAX_RETRIES` | `TBG_CarryingComponent.c` | `3` | Retries to find the carried grenade child on remote clients before giving up |
| `DETACH_RETRY_DELAY_MS` | `TBG_CarryingComponent.c` | `200` | Delay between those retries |
| `RETRY_MAX` / `RETRY_DELAY_MS` | `TBG_PickUpGrenadeAction.c` | `10` / `50 ms` | Wait for the dummy grenade to replicate after spawn |
| `TBG_EQUIP_MAX_RETRIES` / `TBG_EQUIP_RETRY_DELAY_MS` | `TBG_PlayerController.c` | `10` / `100 ms` | Wait for the grenade to replicate before equip/throw on the owner client |
| landing delta check | `TBG_CarryingComponent.c` | `500 ms` after throw, delta `< 0.05` | Detects that the thrown grenade has come to rest |

The only server‑side requirement is that the grenade prefabs carry the TBG components (see §5) and that `TBG_CarryingComponent` is on the character prefab (`Prefabs/Characters/Core/Character_Base.et` in this repo).

## 5. Modding — adding TBG to a grenade

On the grenade prefab:

1. Add **`TBG_ThrowBackComponent`** (holds the `ETBGGrenadeState` state machine).
2. On its **`ActionsManagerComponent → Additional Actions`**, add **`TBG_PickUpGrenadeAction`** with `ParentContextList = { "default" }`. Configure its UIInfo name so the action appears.

On the character prefab: add **`TBG_CarryingComponent`**.

The modded `SCR_PlayerController` (`TBG_PlayerController.c`) and `SCR_WeaponSwitchingBaseUI` (`TBG_WeaponSwitchingUI.c`) are applied automatically by the mod.

## 6. Architecture (short)

```
Character Entity
└── TBG_CarryingComponent    — tracks carried grenade, dummy-follow loop, all RPCs

Grenade Entity (prefab)
├── TBG_ThrowBackComponent   — ETBGGrenadeState: IDLE, THROWN, CARRIED, ACTIVATED
└── TBG_PickUpGrenadeAction  — pickup action + dummy spawn callback

SCR_PlayerController (modded) — TBG_EquipAndThrow on the owner client
SCR_WeaponSwitchingBaseUI (modded) — detects slot switch → drop
```

### Network model

```
Owner Client → RpcServer_TBG_* → Server → RpcBroadcast_TBG_* → Remote Clients
```

The **server is a pure relay** — it never sets member variables, only forwards RPCs. All gameplay logic runs on the **owning client** (`_Client` methods). Remote clients mirror state from broadcast RPCs. `PerformAction` runs on every machine (so remote clients get `CARRIED` state + the grenade in the character hierarchy for tracking); inventory operations (slot clear, dummy spawn) are server‑only.

### Dummy‑follow mechanic (5 phases, owning client)

1. Wait until the dummy grenade leaves the character hierarchy (= thrown).
2. First frame after throw → create a **flight proxy**, attach the carried grenade to it.
3. Every tick: proxy position = dummy position (follows the throw arc).
4. 500 ms after throw detection → arm the delta check.
5. Delta `< 0.05` → grenade landed → pin it to the landing position, delete the dummy. **The flight proxy is kept** as a landing anchor (`RemoveChild(false)` does not preserve world position, and the inventory system would otherwise snap the grenade back to the character).

### Key invariants

- The flight proxy must stay alive after landing — never delete or `RemoveChild` it.
- `ConfirmLandingPos` fires 500 ms after the swap on remote clients to re‑pin the position (something overwrites it after the first set).
- `SetDummyGrenade` runs on the server (from `TBG_PickupSpawnCallback.OnComplete`) so the inventory parent change is visible server‑side.
- `TBG_EquipAndThrow` routes via RPC to the owning client — `SelectWeapon` / `SetThrow` must run on the owner, not the server.

### Key files

| File | Purpose |
|------|---------|
| `Scripts/Game/TBG_ThrowBackComponent.c` | State enum + state component on the grenade |
| `Scripts/Game/TBG_CarryingComponent.c` | Main logic: carrying, dummy‑follow, RPCs (~32 KB) |
| `Scripts/Game/TBG_PickUpGrenadeAction.c` | Pickup action + spawn callback |
| `Scripts/Game/TBG_PlayerController.c` | Modded PlayerController: equip & throw, slot‑switch drop |
| `Scripts/Game/TBG_WeaponSwitchingUI.c` | Modded switching UI: slot‑change detection |
| `Scripts/Game/TBG_DebugConfig.c` | Debug logging gate |

### Naming conventions

`TBG_` classes/files · `ETBG_` enums · members: `m_p` pointer/entity, `m_b` bool, `m_v` vector, `m_i` int · `_Client` = owning‑client logic · `_Server` = server relay · `RpcServer_TBG_` = owner→server · `RpcBroadcast_TBG_` = server→all.

## 7. Known behaviour / limitations

- Pickup is blocked while a normal engine "Pick Up" action is available for the item, and while the grenade is already parented to a character.
- Switching quick‑slots while carrying always drops the grenade (by design — you can't stow a live grenade).
- Multiplayer edge cases are handled with retry loops (replication lag) rather than hard guarantees; see the retry constants in §4.

---

# Throw Back Grenade (TBG) — Deutsch

> Quelle: `…/ArmaReforgerWorkbench/addons/Throw Back Grenade` · Präfix `TBG_` · Enfusion / Enforce Script. Verifiziert gegen `Scripts/Game/TBG_*.c`, `Configs/System/*.conf`, `Prefabs/Weapons/Grenades/*.et`.

## 1. Kurzbeschreibung

Throw Back Grenade erlaubt es einem Spieler, eine **scharfe, bereits geworfene Granate aufzuheben und sofort zurückzuwerfen** (oder woanders hin), bevor sie hochgeht. Funktioniert für Splitter- und Rauchgranaten.

- Eine Welt-Aktion „**Pick Up and Throw Grenade**" erscheint an einer scharfen Granate, die am Boden liegt.
- Beim Aufheben kommt die Granate in die Hand des Charakters, eine **Dummy-Inventarkopie** wird in den Wurfslot gespawnt und automatisch ausgerüstet, und der Spieler wirft sie mit der normalen Wurftaste.
- Die getragene Granate **folgt der Wurfbahn des Dummys** über einen versteckten „Flight-Proxy", damit die echte (weiter tickende) Granate dort landet, wo gezielt wurde, und dann nach ihrem eigenen Timer detoniert / raucht.
- Läuft die Granate ab, während sie noch in der Hand ist (Rauch endet, Explosion), wird der Dummy automatisch aus dem Inventar entfernt.

### Unterstützte Granaten (mitgelieferte Prefabs)

| Prefab | Typ |
|--------|-----|
| `Grenade_M67.et` | Splitter (US) |
| `Grenade_RGD5.et` | Splitter (UdSSR) |
| `Smoke_ANM8HC.et` | Rauch (HC weiß) |
| `Smoke_RDG2.et` | Rauch (UdSSR) |
| `Smoke_M18_Base.et` | Rauch (M18, farbig) |

Jedes Granaten-Prefab wird durch Hinzufügen von zwei Komponenten unterstützt (siehe §5).

## 2. Bedienung (Spieler)

1. Eine feindliche (oder befreundete) Granate landet neben dir und ist noch scharf.
2. Draufschauen — die Aktion **„Pick Up and Throw Grenade"** erscheint (nur wenn *keine* normale „Pick Up"-Aktion für das Item verfügbar ist und die Granate nicht bereits an einem Charakter hängt).
3. Aktion ausführen. Die Granate kommt in die Hand und wird im Wurfslot automatisch ausgewählt.
4. Mit der normalen Wurftaste **werfen**, oder mit **`Strg+X`** (`TBG_Throw`, Standard). Vorher zielen — die scharfe Granate folgt dem Wurf.
5. Wechselst du stattdessen zu einer anderen Quickslot-Waffe, während du sie hältst, wird die Granate **fallengelassen** (sie verschwindet nicht).

⚠️ Die Granate zählt die ganze Zeit weiter herunter. Eine aufzuheben ist ein Glücksspiel.

## 3. Tastenbelegung

Kategorie **TBG** unter *Einstellungen → Tastenbelegung*.

| Aktion | Standard | Kontext | Anmerkung |
|--------|----------|---------|-----------|
| `TBG_Throw` („Throw Granade") | `Strg + X` (`KC_LCONTROL` + `KC_X`) | `TBG_Action` (Priorität 100) | Wirft die aktuell getragene Granate. Die normale Granaten-Wurftaste funktioniert ebenfalls. |

Das Aufheben selbst ist eine **Welt-Aktion** („Pick Up and Throw Grenade"), keine Taste — normale Interaktionstaste nutzen.

## 4. Einstellungen

### Client-Einstellungen

Keine. Es gibt kein In-Game-Einstellungsmenü für TBG.

### Server-Einstellungen

Keine eigene Config-Datei. `Configs/Editor/TBG_DebugMode.conf` (`TBG_DebugConfig`) steuert nur das Debug-Logging (`TBG_DebugConfig.Log(...)`). Einstellbare Konstanten stehen in den Scripts:

| Konstante | Datei | Standard | Bedeutung |
|-----------|-------|----------|-----------|
| `DETACH_MAX_RETRIES` | `TBG_CarryingComponent.c` | `3` | Versuche, das getragene Granaten-Child auf Remote-Clients zu finden, bevor aufgegeben wird |
| `DETACH_RETRY_DELAY_MS` | `TBG_CarryingComponent.c` | `200` | Verzögerung zwischen diesen Versuchen |
| `RETRY_MAX` / `RETRY_DELAY_MS` | `TBG_PickUpGrenadeAction.c` | `10` / `50 ms` | Warten, bis die Dummy-Granate nach dem Spawn repliziert ist |
| `TBG_EQUIP_MAX_RETRIES` / `TBG_EQUIP_RETRY_DELAY_MS` | `TBG_PlayerController.c` | `10` / `100 ms` | Warten, bis die Granate repliziert ist, vor Equip/Wurf auf dem Owner-Client |
| Landungs-Delta-Check | `TBG_CarryingComponent.c` | `500 ms` nach Wurf, Delta `< 0.05` | Erkennt, dass die geworfene Granate zur Ruhe gekommen ist |

Serverseitig ist nur nötig, dass die Granaten-Prefabs die TBG-Komponenten tragen (siehe §5) und `TBG_CarryingComponent` am Charakter-Prefab ist (`Prefabs/Characters/Core/Character_Base.et` in diesem Repo).

## 5. Modding — TBG an eine Granate hängen

Am Granaten-Prefab:

1. **`TBG_ThrowBackComponent`** hinzufügen (enthält den `ETBGGrenadeState`-Zustandsautomaten).
2. An dessen **`ActionsManagerComponent → Additional Actions`** die Aktion **`TBG_PickUpGrenadeAction`** mit `ParentContextList = { "default" }` hinzufügen. UIInfo-Namen setzen, damit die Aktion erscheint.

Am Charakter-Prefab: **`TBG_CarryingComponent`** hinzufügen.

Die modded `SCR_PlayerController` (`TBG_PlayerController.c`) und `SCR_WeaponSwitchingBaseUI` (`TBG_WeaponSwitchingUI.c`) werden automatisch vom Mod angewandt.

## 6. Architektur (kurz)

```
Charakter-Entität
└── TBG_CarryingComponent    — getragene Granate, Dummy-Follow-Schleife, alle RPCs

Granaten-Entität (Prefab)
├── TBG_ThrowBackComponent   — ETBGGrenadeState: IDLE, THROWN, CARRIED, ACTIVATED
└── TBG_PickUpGrenadeAction  — Aufheben-Aktion + Dummy-Spawn-Callback

SCR_PlayerController (modded) — TBG_EquipAndThrow auf dem Owner-Client
SCR_WeaponSwitchingBaseUI (modded) — erkennt Slot-Wechsel → Fallenlassen
```

### Netzwerkmodell

```
Owner-Client → RpcServer_TBG_* → Server → RpcBroadcast_TBG_* → Remote-Clients
```

Der **Server ist ein reiner Relay** — er setzt nie Member-Variablen, sondern leitet nur RPCs weiter. Die gesamte Gameplay-Logik läuft auf dem **Owner-Client** (`_Client`-Methoden). Remote-Clients spiegeln den Zustand aus Broadcast-RPCs. `PerformAction` läuft auf jeder Maschine (damit Remote-Clients den `CARRIED`-Zustand + die Granate in der Charakter-Hierarchie fürs Tracking haben); Inventar-Operationen (Slot leeren, Dummy spawnen) sind server-only.

### Dummy-Follow-Mechanik (5 Phasen, Owner-Client)

1. Warten, bis die Dummy-Granate die Charakter-Hierarchie verlässt (= geworfen).
2. Erster Frame nach dem Wurf → **Flight-Proxy** erstellen, getragene Granate daran hängen.
3. Jeder Tick: Proxy-Position = Dummy-Position (folgt der Wurfbahn).
4. 500 ms nach Wurf-Erkennung → Delta-Check scharfschalten.
5. Delta `< 0.05` → Granate gelandet → an Landeposition pinnen, Dummy löschen. **Der Flight-Proxy bleibt** als Lande-Anker (`RemoveChild(false)` erhält die Weltposition nicht, und das Inventarsystem würde die Granate sonst zum Charakter zurückschnappen).

### Wichtige Invarianten

- Der Flight-Proxy muss nach der Landung am Leben bleiben — nie löschen oder `RemoveChild`.
- `ConfirmLandingPos` feuert 500 ms nach dem Swap auf Remote-Clients, um die Position neu zu pinnen (etwas überschreibt sie nach dem ersten Setzen).
- `SetDummyGrenade` läuft auf dem Server (aus `TBG_PickupSpawnCallback.OnComplete`), damit der Inventar-Parent-Wechsel serverseitig sichtbar ist.
- `TBG_EquipAndThrow` läuft per RPC zum Owner-Client — `SelectWeapon` / `SetThrow` müssen auf dem Owner laufen, nicht auf dem Server.

### Wichtige Dateien

| Datei | Zweck |
|-------|-------|
| `Scripts/Game/TBG_ThrowBackComponent.c` | State-Enum + State-Komponente an der Granate |
| `Scripts/Game/TBG_CarryingComponent.c` | Hauptlogik: Tragen, Dummy-Follow, RPCs (~32 KB) |
| `Scripts/Game/TBG_PickUpGrenadeAction.c` | Aufheben-Aktion + Spawn-Callback |
| `Scripts/Game/TBG_PlayerController.c` | Modded PlayerController: Equip & Wurf, Slot-Wechsel-Drop |
| `Scripts/Game/TBG_WeaponSwitchingUI.c` | Modded Switching-UI: Slot-Wechsel-Erkennung |
| `Scripts/Game/TBG_DebugConfig.c` | Debug-Logging-Gate |

### Namenskonventionen

`TBG_` Klassen/Dateien · `ETBG_` Enums · Member: `m_p` Pointer/Entität, `m_b` Bool, `m_v` Vektor, `m_i` Int · `_Client` = Owner-Client-Logik · `_Server` = Server-Relay · `RpcServer_TBG_` = Owner→Server · `RpcBroadcast_TBG_` = Server→Alle.

## 7. Bekanntes Verhalten / Einschränkungen

- Aufheben ist blockiert, solange eine normale Engine-„Pick Up"-Aktion für das Item verfügbar ist, und solange die Granate bereits an einem Charakter hängt.
- Ein Quickslot-Wechsel während des Tragens lässt die Granate immer fallen (Absicht — eine scharfe Granate kann nicht verstaut werden).
- Multiplayer-Randfälle werden mit Retry-Schleifen (Replikations-Latenz) behandelt, nicht mit harten Garantien; siehe die Retry-Konstanten in §4.
