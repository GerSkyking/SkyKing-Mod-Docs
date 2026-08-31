# SIDC-Framework — Channel System

> Source: `Configs/SIDC_ChannelConfig.conf`, `Configs/SIDC_PhysicalChannelConfig.conf`, `Scripts/Game/SIDC/MarkerConfig/SIDC_Channel*.c`.

## 1. Two independent layers

| Layer | Config | What it does |
|-------|--------|--------------|
| **Communication channel** | `SIDC_ChannelConfig.conf` | The channel a marker is *published* on. Others see it based on **visibility‑percentage** rules between the sender's channel and the viewer's channel. |
| **Physical channel** | `SIDC_PhysicalChannelConfig.conf` | A hard gate on top: `Map` (default), `ATAK`, or `All`. A player only sees markers whose physical channel matches theirs — unless they are on `All` and the server allows it (`allowAllPhysicalChannel`). |

`SIDC_ChannelVisibilityResolver.ResolvePercent()` combines both: the physical gate is applied first, then the communication‑channel visibility percentage.

## 2. Communication channels

Each `SIDC_ChannelEntry` has a name, a language key, a **scope** (`ALL`, `SIDE`, `GROUP`), an optional `m_bIsDefault`, an optional `m_sCategoryKey`, and a list of `m_aVisibilityRules`. A visibility rule = "a marker on **this** channel is visible to a viewer on `m_sTargetChannelLanguageKey` at `m_fVisibilityPercent` %".

| Channel | Scope | Default | Notes |
|---------|-------|---------|-------|
| **All** | `ALL` | | Everyone. |
| **Command** | `SIDE` | | Visible to Group 50 %, Air 50 %, Artillery 50 %. |
| **Group** | `GROUP` | | Only the player's own group. |
| **Side** | `SIDE` | ✅ default | Visible to Group 25 %, All 5 %, Command 50 %. |
| **Air** | `SIDE` (category `Air`) | | Air 100 %, Side 50 %, Artillery 10 %, Group 75 %. |
| **Artillery** | `SIDE` (category `Air`) | | Side 50 %, Air 10 %, Group 75 %. |
| **Infantry** | `SIDE` | | Artillery 25 %, Air 10 %, Group 50 %. |
| **BFT – Blue Force Tracking** | `SIDE` | | Artillery 25 %, Air 10 %, Group 50 %. |

> "Visibility percent" is a per‑marker roll — a marker on *Side* shows to a *Group*‑channel viewer 25 % of the time, giving a deliberately noisy / partial common picture between channels. 100 % = always, absent rule = never (outside the same channel).

## 3. Physical channels

| Entry | Language key | Flag |
|-------|--------------|------|
| **Map** | `#SIDC-Channel-Physical-Map` | `m_bIsDefault 1` |
| **ATAK** | `#SIDC-Channel-Physical-ATAK` | |
| **All** | `#SIDC-UI-text_All` | `m_bIsAllChannel 1` |

Server flags that govern the physical layer (replicated to clients so the spinbox can decide what to offer):

- `allowAllPhysicalChannel` — if off, the "All" entry is hidden everywhere and the physical gate is always enforced, even for a player locally stuck on "All".
- `allPhysicalChannelIsDefault` — if on (and the above is on), fresh profiles start on "All".

## 4. Changing your channel

- Communication channel: the map channel UI (`SIDC_Map_Channel_UI` / `SIDC_ChangeChannelController`).
- Physical channel: `SpinBox_PhysicalMarkerChannel` in the settings menu (`SIDC_PhysicalChannelConfig.PopulateSpinBox()`).

## 5. Tuning for admins

To change the visibility matrix, edit `Configs/SIDC_ChannelConfig.conf` in the mod and rebuild — the channel config is **not** part of `Settings.json` and is not runtime‑editable. Add / remove `SIDC_ChannelVisibilityRule` blocks under the relevant `SIDC_ChannelEntry`.

---

# SIDC-Framework — Kanalsystem (Deutsch)

> Quelle: `Configs/SIDC_ChannelConfig.conf`, `Configs/SIDC_PhysicalChannelConfig.conf`, `Scripts/Game/SIDC/MarkerConfig/SIDC_Channel*.c`.

## 1. Zwei unabhängige Ebenen

| Ebene | Config | Funktion |
|-------|--------|----------|
| **Kommunikationskanal** | `SIDC_ChannelConfig.conf` | Der Kanal, auf dem ein Marker *veröffentlicht* wird. Andere sehen ihn anhand von **Sichtbarkeits-Prozent**-Regeln zwischen Sender- und Betrachterkanal. |
| **Physischer Kanal** | `SIDC_PhysicalChannelConfig.conf` | Ein hartes Gate obendrauf: `Map` (Standard), `ATAK` oder `All`. Ein Spieler sieht nur Marker mit passendem physischen Kanal — außer er ist auf `All` und der Server erlaubt es (`allowAllPhysicalChannel`). |

`SIDC_ChannelVisibilityResolver.ResolvePercent()` kombiniert beides: zuerst das physische Gate, dann der Sichtbarkeits-Prozentwert des Kommunikationskanals.

## 2. Kommunikationskanäle

Jeder `SIDC_ChannelEntry` hat Name, Language-Key, **Scope** (`ALL`, `SIDE`, `GROUP`), optional `m_bIsDefault`, optional `m_sCategoryKey` und eine Liste `m_aVisibilityRules`. Eine Sichtbarkeitsregel = „ein Marker auf **diesem** Kanal ist für einen Betrachter auf `m_sTargetChannelLanguageKey` zu `m_fVisibilityPercent` % sichtbar".

| Kanal | Scope | Standard | Regeln |
|-------|-------|----------|--------|
| **All** | `ALL` | | Alle. |
| **Command** | `SIDE` | | Group 50 %, Air 50 %, Artillery 50 %. |
| **Group** | `GROUP` | | Nur die eigene Gruppe. |
| **Side** | `SIDE` | ✅ Standard | Group 25 %, All 5 %, Command 50 %. |
| **Air** | `SIDE` (Kategorie `Air`) | | Air 100 %, Side 50 %, Artillery 10 %, Group 75 %. |
| **Artillery** | `SIDE` (Kategorie `Air`) | | Side 50 %, Air 10 %, Group 75 %. |
| **Infantry** | `SIDE` | | Artillery 25 %, Air 10 %, Group 50 %. |
| **BFT – Blue Force Tracking** | `SIDE` | | Artillery 25 %, Air 10 %, Group 50 %. |

> „Sichtbarkeits-Prozent" ist ein Wurf pro Marker — ein Marker auf *Side* erscheint einem *Group*-Kanal-Betrachter zu 25 %, was ein bewusst unvollständiges gemeinsames Lagebild zwischen Kanälen ergibt. 100 % = immer, fehlende Regel = nie (außerhalb desselben Kanals).

## 3. Physische Kanäle

| Eintrag | Language-Key | Flag |
|---------|--------------|------|
| **Map** | `#SIDC-Channel-Physical-Map` | `m_bIsDefault 1` |
| **ATAK** | `#SIDC-Channel-Physical-ATAK` | |
| **All** | `#SIDC-UI-text_All` | `m_bIsAllChannel 1` |

Server-Flags für die physische Ebene (an Clients repliziert, damit die Spinbox weiß, was sie anbieten darf):

- `allowAllPhysicalChannel` — wenn aus, wird der „All"-Eintrag überall ausgeblendet und das physische Gate immer erzwungen, auch für einen Spieler, der lokal noch auf „All" steht.
- `allPhysicalChannelIsDefault` — wenn an (und obiges an), starten frische Profile auf „All".

## 4. Kanal wechseln

- Kommunikationskanal: die Karten-Kanal-UI (`SIDC_Map_Channel_UI` / `SIDC_ChangeChannelController`).
- Physischer Kanal: `SpinBox_PhysicalMarkerChannel` im Einstellungsmenü (`SIDC_PhysicalChannelConfig.PopulateSpinBox()`).

## 5. Anpassung für Admins

Um die Sichtbarkeitsmatrix zu ändern, `Configs/SIDC_ChannelConfig.conf` im Mod bearbeiten und neu bauen — die Kanal-Config ist **nicht** Teil von `Settings.json` und nicht zur Laufzeit editierbar. `SIDC_ChannelVisibilityRule`-Blöcke unter dem jeweiligen `SIDC_ChannelEntry` hinzufügen/entfernen.
