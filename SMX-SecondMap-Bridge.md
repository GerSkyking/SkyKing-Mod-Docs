# SMX × SecondMap Bridge (SkyMap-X map provider)

> Source: `SkyMap-X-DEV` (`Scripts/Game/Map/Provider/`, hooks in `SMX_MainComponentX`), bridge addons `SIDC-SecondMap-SMX-Bridge(-Beta)` and `SIDC-SMX-Bridge(-Beta)`. **Status: beta** — the production bridges still carry the older scripts and need a release SkyMap-X that contains the provider and hooks.

## 1. What it is

The map inside the SkyMap-X tablet (Mini / Big / Hand) is replaceable. SkyMap-X only asks a **map provider** to open, close, zoom and pan; the tablet UI stays the same. By default the provider is the vanilla map (`SCR_MapEntity`). With the bridge addons loaded, the tablet shows the **SIDC-SecondMap** (its own renderer, embedded mode) instead.

SkyMap-X, SIDC-Framework and SIDC-SecondMap do **not** depend on each other. Reforger has no optional dependencies, so two small bridge addons link them (by ID) and hook in with `modded class` only:

| Addon | Needs | Does |
|---|---|---|
| `SIDC-SMX-Bridge` (`-Beta`) | SkyMap-X, SIDC-Framework | Sets the physical SIDC channel (ATAK) for the tablet |
| `SIDC-SecondMap-SMX-Bridge` (`-Beta`) | SkyMap-X, SIDC-Framework, SIDC-SecondMap | Shows the SecondMap in the tablet frame, draws BFT trackers as SIDC markers |

Load **either the Beta or the Production pair, never both.** The Beta pair needs the SkyMap-X beta project (`SkyMapXbeta`); an old copy of the same GUID in the Workbench `addons` folder must be removed, otherwise the old scripts load.

## 2. What you see in game

- The tablet map is the SecondMap: SIDC markers, drawing, the SIDC keybinds and the channel info all work on the tablet.
- **Physical channel:** while the tablet is on and whenever its map opens, you are on **ATAK**. Exception: the server allows "All" and set it as default — then it stays on All. When the tablet is switched off, your previous channel is restored.
- **Normal map (paper map):** opening it while the tablet is on sets the channel to the default again (**Map**, or **All** if the server default is All).
- Markers you place on the tablet are ATAK markers (visible on the tablet), markers placed on the normal map are Map markers.
- **BFT trackers** (device, SMT, UTD) appear as SIDC symbols with their name in the info field and no direction arrow. Friend (own faction) / neutral (CIV) / hostile (other) is set automatically.
- **Server setting `bftShowMarkersOnMap`:** `false` → BFT markers are shown only on the tablet, never on a map (even on the All channel). `true` → also on the normal map.
- Pressing **Big/Mini** while the paper map is open closes the map and then opens the requested mode.
- **Not synced to world displays:** BFT markers (trackers) are not shown on tablet-watching displays or monitors, only SIDC markers; the server does not stream trackers to viewers without an active SMX tablet. See [SIDC-SecondMap §10](SIDC-SecondMap).
- **Brightness:** the tablet brightness (keybinds / settings slider) is forwarded to the SecondMap (`SMX_MapProvider.SetBrightness` → `SIDC_SecondMap_ViewBase.SetSyncBrightness`) and synced to the world displays that show your tablet (tablet watching, monitors), which dim accordingly. Details: [SIDC-SecondMap §10](SIDC-SecondMap).

## 3. Tracker symbols (admin / mod maker)

`SIDC-SecondMap-SMX-Bridge` → `Configs/SMX_SIDC_TrackerMarkers.conf`: one entry per faction and tracker kind (`DEVICE`, `SMT`, `UTD`).

- `m_FactionConfig`: drag and drop e.g. `SMX_Faction_FIA.conf` — the first faction key in it (e.g. `FIA`) is the tracker's key. Or `m_sFactionKey` as plain text. Both empty = default for all factions.
- `m_sSidc`: 30-digit SIDC number from the SIDC AllMarkers list. The affiliation digit (4th) is overwritten automatically.
- Write the number on **one line** (`m_sSidc "100327000011010100000000000000"`).

## 4. For mod makers: replace the map

Override one static function; return your own provider (subclass of `SMX_MapProvider`):

```
modded class SMX_MapProviderFactory
{
	override static SMX_MapProvider Create()
	{
		return new MyMapProvider();
	}
}
```

Hooks in `SMX_MainComponentX` (override and call `super`): `OnTabletPoweredOn()`, `OnTabletPoweredOff()`, `OnTabletMapOpening()`. If the provider fails to open, SkyMap-X falls back to the vanilla map.

## 5. Known limitations / open

- The map view is rebuilt on every tablet mode change; keeping it alive across Mini/Big/Hand is planned, not done.
- Tablet zoom is capped at 4 px/m (the SecondMap is one big image; higher zoom turns it black).
- Production bridges not updated yet.

## 5b. Vehicles, AI and carrier icons

- `SMX_TrackedEntityComponent` (SkyMap-X): put it on a vehicle or an AI character to give it a BFT marker. Fields: faction config, SIDC, info text, icon. Unit text = group of the driver / the AI group; info only if set in the editor. Visibility is faction-only (observer needs an active tablet).
- `SMX_SIDC_PlayerIconComponent` (this bridge): put it on a player/character prefab to override the default SIDC icon of that player's tablet / SMT (e.g. GR vs. TL). Only the icon changes, never visibility. Empty = config default.
- Faction: from the faction config if set, otherwise from the entity (`FactionAffiliationComponent`, incl. the vehicle / character variants; re-read every 2 s, so a vehicle follows its crew). Symbol colour per faction = entries of kind `VEHICLE` / `AI` in `Configs/SMX_SIDC_TrackerMarkers.conf` (4th SIDC digit = affiliation).
- Empty vehicle: unit text (T) stays empty until a driver with a group enters; the info text only appears in H.
- **Vehicle name by the player (fuel cap):** the action *Change vehicle name (BFT)* (`SMX_UserAction_EditCarName`) sits on the `fuel_cap` context of the vehicle prefab (`Vehicle_Base.et`), only on vehicles with the component. It opens a dialog (`SMX_UI_Driver_Edit-BFT.layout`: `eb_CarName`, `btn_apply`, `btn_cancel`). The server only accepts it within 12 m of the vehicle; max. 32 characters. Unit text = `group - vehicle name` (e.g. for convoys); without a name only the group. Preset value in the editor: `m_sCarName`.
- **Occupants:** the device markers (tablet / SMT / UTD) of the occupants are hidden while they sit in a tracked vehicle, so they do not stack. The info text shows `driver name [+N]` (N = further occupants with an active tracker).
- **Status:** AI / vehicles show fully capable / damaged / destroyed from damage and consciousness; the Zeus can force it.

### Zeus

- **Sees everything:** a player with the full Zeus editor sees all BFT markers (no tablet / faction / subchannel check), also on the vanilla map. Server option `bftShowZeus` in `SMX_ConfigV3.Json` (default `true`).
- **Zeus map:** the editor map has neither `SCR_MapMarkersUI` nor a `MapFrame`, so vanilla marker widgets are not drawn there. `SMX_ZeusTrackerRenderer` (SkyMap-X) draws the trackers itself (SMX icon + text); with the bridge, `SMX_SIDC_ZeusTrackerView` draws SIDC symbols instead. The framework's `SIDC_MarkerRenderer.FindMapFrame` got a fallback (parent of the map widget) so normal SIDC markers show on the Zeus map as well.
- **Edit BFT:** right-click action `SMX_ContextAction_EditBft` (based on `SCR_SelectedEntitiesContextAction`) on an entity with the component opens a dialog (`SMX_UI_Zeus_Edit-BFT.layout`: `eb_info`, `eb_sidc`, `cb_status`, `btn_apply`, `btn_cancel`). The server checks the sender has an open full editor. The action must be added to the editor's context action list (override of the vanilla config).
- **Status (Zeus only):** fully capable / damaged / destroyed. Auto from damage (AI: unconscious or dead = destroyed, health < 99 % = damaged; vehicle: destroyed state) or fixed in the dialog / component. Shown as the SIDC status digit (index 6: 2 / 3 / 4), only on the Zeus client; never in the export (`Build(tracker, false)`). The state is replicated to all clients but only displayed for the Zeus. Dead AI / destroyed vehicles stay as "destroyed" and are removed after `m_iRemoveDestroyedSec` (default 300 s, -1 = never).

## 5c. Picker, preview, Advanced Builder

- **Edit BFT (Zeus):** the dialog has a marker picker (Builder picker mode of the framework's Advanced Builder) and a live `SIDC_MarkerView` preview next to the SIDC field.
- **Defaults:** player characters get no AI marker; `SMX_SIDC_PlayerIconComponent.GetSidc()` only applies while the character is player-controlled. Destroyed markers are removed after `m_iRemoveDestroyedSec` (default -1 = never).
- **Advanced Builder:** Country and Combat effectiveness fields added; Reinforced and Reduced are mutually exclusive (enabling one switches the other off). `MapMarkerBaseSIDCpreview.layout` needs `Text_Country` / `Text_CombatEffectiveness` widgets.
- **Inspect menu:** lists every field of the marker as `explanation - current value`. Modifier texts come from `SIDC-Modifier-<Name>_Description`, echelon/mobility from `SIDC-Inspect-Echelon_<n>` / `Mobility_<n>`. Assumption: catalog code = SIDC digits 16-17 / 18-19 (else only the generic modifier text shows).

---

# SMX × SecondMap Bridge (SkyMap-X Map-Provider) — Deutsch

> Quelle: `SkyMap-X-DEV` (`Scripts/Game/Map/Provider/`, Hooks in `SMX_MainComponentX`), Bridge-Addons `SIDC-SecondMap-SMX-Bridge(-Beta)` und `SIDC-SMX-Bridge(-Beta)`. **Status: Beta** — die Produktiv-Bridges haben noch den älteren Stand und brauchen ein Release-SkyMap-X mit Provider und Hooks.

## 1. Kurzbeschreibung

Die Karte im SkyMap-X-Tablet (Mini / Big / Hand) ist austauschbar. SkyMap-X bittet nur einen **Map-Provider** zu öffnen, schließen, zoomen und schwenken; die Tablet-UI bleibt gleich. Standard ist die Vanilla-Karte (`SCR_MapEntity`). Mit den Bridge-Addons zeigt das Tablet stattdessen die **SIDC-SecondMap** (eigener Renderer, Embedded-Modus).

SkyMap-X, SIDC-Framework und SIDC-SecondMap hängen **nicht** voneinander ab. Reforger kennt keine optionalen Abhängigkeiten, deshalb verbinden zwei kleine Bridge-Addons sie (per ID) und hängen sich nur per `modded class` ein:

| Addon | Braucht | Macht |
|---|---|---|
| `SIDC-SMX-Bridge` (`-Beta`) | SkyMap-X, SIDC-Framework | Setzt den physischen SIDC-Channel (ATAK) fürs Tablet |
| `SIDC-SecondMap-SMX-Bridge` (`-Beta`) | SkyMap-X, SIDC-Framework, SIDC-SecondMap | Zeigt die SecondMap im Tablet-Rahmen, zeichnet BFT-Tracker als SIDC-Marker |

Es wird **entweder das Beta- oder das Produktiv-Paar geladen, nie beide.** Das Beta-Paar braucht das SkyMap-X-Beta-Projekt (`SkyMapXbeta`); eine alte Kopie mit derselben GUID im Workbench-`addons`-Ordner muss weg, sonst laden die alten Scripts.

## 2. Was man im Spiel sieht

- Die Tablet-Karte ist die SecondMap: SIDC-Marker, Zeichnen, die SIDC-Tasten und die Kanal-Info funktionieren auf dem Tablet.
- **Physischer Channel:** Solange das Tablet an ist und bei jedem Öffnen seiner Karte bist du auf **ATAK**. Ausnahme: Der Server erlaubt „Alle" und hat es als Default gesetzt, dann bleibt es auf Alle. Beim Ausschalten kommt dein vorheriger Channel zurück.
- **Normale Karte (Papierkarte):** Öffnen bei eingeschaltetem Tablet setzt den Channel wieder auf den Default (**Map**, bzw. **Alle**, wenn der Server-Default Alle ist).
- Auf dem Tablet gesetzte Marker sind ATAK-Marker (auf dem Tablet sichtbar), auf der normalen Karte gesetzte sind Map-Marker.
- **BFT-Tracker** (Gerät, SMT, UTD) erscheinen als SIDC-Symbole, der Name im Info-Feld, ohne Richtungspfeil. Freund (eigene Fraktion) / neutral (CIV) / feindlich (andere) wird automatisch gesetzt.
- **Server-Einstellung `bftShowMarkersOnMap`:** `false` → BFT-Marker nur auf dem Tablet, nie auf einer Karte (auch nicht im Alle-Channel). `true` → zusätzlich auf der normalen Karte.
- **Big/Mini** bei offener Papierkarte schließt die Karte und öffnet danach den gewünschten Modus.
- **Nicht an Welt-Anzeigen synchronisiert:** BFT-Marker (Tracker) erscheinen auf Tablet-Zuschauen-Displays und Monitoren nicht, nur SIDC-Marker; der Server streamt Tracker nicht an Zuschauer ohne aktives SMX-Tablet. Siehe [SIDC-SecondMap §10](SIDC-SecondMap).
- **Helligkeit:** Die Tablet-Helligkeit (Keybinds / Settings-Slider) wird an die SecondMap weitergegeben (`SMX_MapProvider.SetBrightness` → `SIDC_SecondMap_ViewBase.SetSyncBrightness`) und an die Welt-Anzeigen synchronisiert, die dein Tablet zeigen (Tablet-Zuschauen, Monitore); diese dimmen entsprechend. Details: [SIDC-SecondMap §10](SIDC-SecondMap).

## 3. Tracker-Symbole (Admin / Mod-Autor)

`SIDC-SecondMap-SMX-Bridge` → `Configs/SMX_SIDC_TrackerMarkers.conf`: ein Eintrag pro Fraktion und Tracker-Art (`DEVICE`, `SMT`, `UTD`).

- `m_FactionConfig`: z.B. `SMX_Faction_FIA.conf` per Drag und Drop — der erste Faction-Key darin (z.B. `FIA`) ist der Key des Trackers. Oder `m_sFactionKey` als Text. Beides leer = Standard für alle Fraktionen.
- `m_sSidc`: 30-stellige SIDC-Nummer aus der SIDC-AllMarkers-Liste. Die Affiliation-Ziffer (4.) wird automatisch überschrieben.
- Die Nummer **einzeilig** schreiben (`m_sSidc "100327000011010100000000000000"`).

## 4. Für Mod-Autoren: die Karte ersetzen

Eine statische Funktion überschreiben und einen eigenen Provider zurückgeben (Unterklasse von `SMX_MapProvider`):

```
modded class SMX_MapProviderFactory
{
	override static SMX_MapProvider Create()
	{
		return new MyMapProvider();
	}
}
```

Hooks in `SMX_MainComponentX` (überschreiben, `super` aufrufen): `OnTabletPoweredOn()`, `OnTabletPoweredOff()`, `OnTabletMapOpening()`. Schlägt das Öffnen des Providers fehl, fällt SkyMap-X auf die Vanilla-Karte zurück.

## 5. Bekannte Einschränkungen / offen

- Die Kartenansicht wird bei jedem Tablet-Moduswechsel neu aufgebaut; sie über Mini/Big/Hand am Leben zu halten ist geplant, nicht umgesetzt.
- Tablet-Zoom ist auf 4 px/m begrenzt (die SecondMap ist ein großes Bild, mehr Zoom macht sie schwarz).
- Produktiv-Bridges noch nicht nachgezogen.

## 5b. Fahrzeuge, KI und Träger-Icons

- `SMX_TrackedEntityComponent` (SkyMap-X): auf Fahrzeug oder KI-Character legen, dann bekommt die Entity einen BFT-Marker. Felder: Faction Config, SIDC, Info, Icon. Unit = Gruppe des Fahrers bzw. der KI-Gruppe; Info nur, wenn im Editor gesetzt. Sichtbarkeit nur per Fraktion (Beobachter braucht aktives Tablet).
- `SMX_SIDC_PlayerIconComponent` (diese Bridge): auf Spieler-/Charakter-Prefab legen, überschreibt das Standard-SIDC-Icon von Tablet/SMT dieses Spielers (z.B. GR vs. TL). Ändert nur das Icon, nie die Sichtbarkeit. Leer = Config-Standard.
- Fraktion: aus der Faction Config, sonst von der Entity (`FactionAffiliationComponent`, inkl. Fahrzeug-/Charakter-Variante; alle 2 s neu gelesen, ein Fahrzeug folgt also seiner Besatzung). Symbolfarbe je Fraktion = Einträge der Art `VEHICLE` / `AI` in `Configs/SMX_SIDC_TrackerMarkers.conf` (4. SIDC-Ziffer = Zugehörigkeit).
- Leeres Fahrzeug: Einheit (T) bleibt leer, bis ein Fahrer mit Gruppe einsteigt; der Info-Text steht nur in H.
- **Fahrzeugname durch den Spieler (Tankdeckel):** Die Aktion *Fahrzeugname (BFT) ändern* (`SMX_UserAction_EditCarName`) liegt im Kontext `fuel_cap` des Fahrzeug-Prefabs (`Vehicle_Base.et`), nur an Fahrzeugen mit der Component. Sie öffnet einen Dialog (`SMX_UI_Driver_Edit-BFT.layout`: `eb_CarName`, `btn_apply`, `btn_cancel`). Der Server akzeptiert es nur im Umkreis von 12 m ums Fahrzeug; max. 32 Zeichen. Unit-Text = `Gruppe - Fahrzeugname` (z. B. für Konvois); ohne Namen nur die Gruppe. Vorgabewert im Editor: `m_sCarName`.
- **Insassen:** Die Geräte-Marker (Tablet / SMT / UTD) der Insassen werden ausgeblendet, solange sie in einem getrackten Fahrzeug sitzen, damit sie sich nicht stapeln. Der Info-Text zeigt `Fahrername [+N]` (N = weitere Insassen mit aktivem Tracker).
- **Status:** KI / Fahrzeuge zeigen einsatzbereit / beschädigt / zerstört aus Schaden und Bewusstsein; der Zeus kann ihn fest vorgeben.

### Zeus

- **Sieht alles:** Ein Spieler mit vollem Zeus-Editor sieht alle BFT-Marker (kein Tablet-/Fraktions-/Subchannel-Check), auch auf der vanilla Karte. Server-Option `bftShowZeus` in `SMX_ConfigV3.Json` (Default `true`).
- **Zeus-Karte:** Die Editor-Karte hat weder `SCR_MapMarkersUI` noch einen `MapFrame`, vanilla Marker-Widgets werden dort nicht gezeichnet. `SMX_ZeusTrackerRenderer` (SkyMap-X) zeichnet die Tracker selbst (SMX-Icon + Text); mit der Bridge zeichnet `SMX_SIDC_ZeusTrackerView` stattdessen SIDC-Symbole. `SIDC_MarkerRenderer.FindMapFrame` im Framework hat einen Fallback (Parent des Karten-Widgets), damit auch normale SIDC-Marker auf der Zeus-Karte erscheinen.
- **Edit BFT:** Rechtsklick-Aktion `SMX_ContextAction_EditBft` (basiert auf `SCR_SelectedEntitiesContextAction`) auf einer Entity mit der Komponente öffnet einen Dialog (`SMX_UI_Zeus_Edit-BFT.layout`: `eb_info`, `eb_sidc`, `cb_status`, `btn_apply`, `btn_cancel`). Der Server prüft, dass der Absender einen offenen vollen Editor hat. Die Aktion muss in die Context-Action-Liste des Editors (Override der vanilla Config).
- **Status (nur Zeus):** einsatzbereit / beschädigt / zerstört. Automatisch aus dem Schaden (KI: bewusstlos oder tot = zerstört, Gesundheit < 99 % = beschädigt; Fahrzeug: Zustand zerstört) oder fest im Dialog / an der Komponente. Dargestellt als SIDC-Status-Ziffer (Index 6: 2 / 3 / 4), nur auf dem Zeus-Client und nie im Export (`Build(tracker, false)`). Der Zustand wird an alle Clients repliziert, aber nur beim Zeus angezeigt. Tote KI / zerstörte Fahrzeuge bleiben als „zerstört“ stehen und werden nach `m_iRemoveDestroyedSec` entfernt (Default 300 s, -1 = nie).

## 5c. Picker, Vorschau, Advanced Builder

- **Edit BFT (Zeus):** Der Dialog hat einen Marker-Picker (Builder-Picker-Modus des Advanced Builders im Framework) und eine Live-Vorschau (`SIDC_MarkerView`) neben dem SIDC-Feld.
- **Defaults:** Spieler-Charaktere bekommen keinen KI-Marker; `SMX_SIDC_PlayerIconComponent.GetSidc()` greift nur, solange der Charakter von einem Spieler gesteuert wird. Zerstörte Marker werden nach `m_iRemoveDestroyedSec` entfernt (Default -1 = nie).
- **Advanced Builder:** Felder Land und Gefechtsstärke ergänzt; Verstärkt und Vermindert schließen sich aus (Aktivieren des einen schaltet das andere aus). `MapMarkerBaseSIDCpreview.layout` braucht die Widgets `Text_Country` / `Text_CombatEffectiveness`.
- **Inspect-Menü:** Listet jedes Feld des Markers als `Erklärung - aktueller Wert`. Modifikator-Texte aus `SIDC-Modifier-<Name>_Description`, Echelon/Mobilität aus `SIDC-Inspect-Echelon_<n>` / `Mobility_<n>`. Annahme: Katalog-Code = SIDC-Ziffern 16-17 / 18-19 (sonst nur der allgemeine Modifikator-Text).
