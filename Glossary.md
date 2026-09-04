# Glossary

| Term | Meaning |
|------|---------|
| **SMX** | SkyMap-X — the tablet / wrist‑gadget mod. |
| **SUI** | SpeedUI — the speed HUD mod. |
| **SIDC** | SIDC-Framework — the military‑symbol marker system. Also: *Symbol Identification Code*, the APP‑6 / MIL‑STD‑2525D 20‑digit code identifying a symbol. |
| **TBG** | Throw Back Grenade — pick up a live thrown grenade and rethrow it. |
| **Dummy grenade** | TBG: an inventory copy of the picked‑up grenade, spawned into the throwable slot so the player can throw with normal input. |
| **Flight proxy** | TBG: a hidden entity the carried grenade is parented to during flight so it follows the dummy's arc; kept after landing as a position anchor. |
| **P1 snap** | TBG: the inventory system snapping an item back to the character's position (P1) — what the flight proxy prevents. |
| **`ETBGGrenadeState`** | TBG grenade state: IDLE / THROWN / CARRIED / ACTIVATED. |
| **ATAKmaps** | External web-map tool (local server + browser) that shows live player position and SIDC markers by reading the SIDC-Framework mod's file exports. Not a mod. |
| **LocalMapData** | The folder `$profile:SIDC_Framework/LocalMapData/` where the SIDC-Framework mod writes its JSON exports and reads request files — the interface ATAKmaps uses. |
| **`ownerFaction`** | Field in `SIDC_PlacedMarkers.json`: the marker author's real game faction (`SCR_FactionManager` key), separate from the SIDC `channel`. |
| **APP‑6 / MIL‑STD‑2525D** | NATO / US standards for military map symbology. |
| **BFT** | Blue Force Tracking — showing friendly (or channel‑visible) devices/units as map markers. |
| **Affiliation / Identity** | SIDC digit 3: Unknown / Assumed friend / Friend / Neutral / Suspect / Hostile. |
| **Echelon** | Unit size (team … theatre), SIDC digits 8–9 for land units. |
| **Mobility** | Movement type for equipment, SIDC digits 8–9 for `Land_Equipment` (values ≥ 30). |
| **Amplifier** | Umbrella term for the SIDC digit 8–9 field (echelon or mobility). |
| **HQTFD** | Headquarters / Task Force / Dummy indicator, SIDC digit 7. |
| **Communication channel** | SIDC: the channel a marker is published on (Side, Group, Command, Air, Artillery, Infantry, BFT). |
| **Physical channel** | SIDC: hard visibility gate — Map / ATAK / All. |
| **Visibility percent** | SIDC: chance a marker on one channel is shown to a viewer on another channel. |
| **Phase line** | A control‑measure line drawn on the map; SIDC drawing mode. |
| **Quick‑Marker menu** | SIDC's nested `Ctrl+T` menu for picking and placing a symbol. |
| **K23** | SMX wrist mount worn on the vest, with its own input context. |
| **TabMode** | SMX: display variant — OFFtab / ONtab / HANDtab / BIGtab / MINITab / PAPERMap / GMmode. |
| **Mode** | SMX: content mode — OFF / ON / GPS / CHAT / SETTINGS / FEEDS. |
| **Enforce Script** | The C#/C++‑like scripting language of the Enfusion engine (`.c` files). |
| **Workbench** | The Arma Reforger mod editor; the only build/test path for these mods. |
| **`$profile:`** | Path prefix for the player (or dedicated‑server) profile directory. |
| **`RplProp` / `Replication.BumpMe()`** | Enfusion replication: mark a member for sync / force a sync. |
| **`modded class` / `modded enum`** | Enforce mechanism to extend or override an existing engine class/enum. |
| **`[BaseContainerProps]`** | Attribute marking a class as Workbench‑configurable (`.conf` files). |
| **`GetInstance()`** | The singleton access pattern used throughout all three mods. |
| **`Settings.json`** | SIDC: the admin‑editable server settings file in `$profile:SIDC_Framework/ServerSettings/`. |
| **`SMX_ConfigV1.Json`** | SMX: the server‑loaded, client‑replicated config file. |
| **QWERTZ note** | `KC_*` key constants are US‑QWERTY scancode positions; on a German keyboard Y/Z are physically swapped. |

---

# Glossar (Deutsch)

| Begriff | Bedeutung |
|---------|-----------|
| **SMX** | SkyMap-X — der Tablet-/Armband-Gadget-Mod. |
| **SUI** | SpeedUI — der Geschwindigkeits-HUD-Mod. |
| **SIDC** | SIDC-Framework — das Militärsymbol-Markersystem. Auch: *Symbol Identification Code*, der 20-stellige APP-6-/MIL-STD-2525D-Code, der ein Symbol identifiziert. |
| **TBG** | Throw Back Grenade — eine scharfe geworfene Granate aufheben und zurückwerfen. |
| **Dummy-Granate** | TBG: eine Inventarkopie der aufgehobenen Granate, in den Wurfslot gespawnt, damit der Spieler mit normaler Eingabe werfen kann. |
| **Flight-Proxy** | TBG: eine versteckte Entität, an die die getragene Granate während des Flugs gehängt wird, damit sie der Wurfbahn des Dummys folgt; nach der Landung als Positions-Anker behalten. |
| **P1-Snap** | TBG: das Inventarsystem schnappt ein Item zurück auf die Charakterposition (P1) — genau das verhindert der Flight-Proxy. |
| **`ETBGGrenadeState`** | TBG-Granatenzustand: IDLE / THROWN / CARRIED / ACTIVATED. |
| **ATAKmaps** | Externes Web-Karten-Tool (lokaler Server + Browser), das Live-Spielerposition und SIDC-Marker zeigt, indem es die Dateiexporte des SIDC-Framework-Mods liest. Kein Mod. |
| **LocalMapData** | Der Ordner `$profile:SIDC_Framework/LocalMapData/`, in den der SIDC-Framework-Mod seine JSON-Exporte schreibt und Request-Dateien liest — die Schnittstelle, die ATAKmaps nutzt. |
| **`ownerFaction`** | Feld in `SIDC_PlacedMarkers.json`: die echte Spiel-Fraktion des Marker-Erstellers (`SCR_FactionManager`-Key), getrennt vom SIDC-`channel`. |
| **APP-6 / MIL-STD-2525D** | NATO-/US-Standards für militärische Kartensymbolik. |
| **BFT** | Blue Force Tracking — befreundete (oder kanal-sichtbare) Geräte/Einheiten als Kartenmarker. |
| **Affiliation / Identity** | SIDC-Ziffer 3: Unbekannt / Vermutet freund / Freund / Neutral / Verdächtig / Feindlich. |
| **Echelon** | Einheitengröße (Team … Theatre), SIDC-Ziffern 8–9 bei Land-Units. |
| **Mobility** | Bewegungstyp bei Ausrüstung, SIDC-Ziffern 8–9 bei `Land_Equipment` (Werte ≥ 30). |
| **Amplifier** | Oberbegriff für das SIDC-Feld Ziffern 8–9 (Echelon oder Mobility). |
| **HQTFD** | Headquarters/Task-Force/Dummy-Indikator, SIDC-Ziffer 7. |
| **Kommunikationskanal** | SIDC: der Kanal, auf dem ein Marker veröffentlicht wird (Side, Group, Command, Air, Artillery, Infantry, BFT). |
| **Physischer Kanal** | SIDC: hartes Sichtbarkeits-Gate — Map / ATAK / All. |
| **Sichtbarkeits-Prozent** | SIDC: Chance, dass ein Marker eines Kanals einem Betrachter auf einem anderen Kanal angezeigt wird. |
| **Phase Line** | Eine auf der Karte gezeichnete Control-Measure-Linie; SIDC-Zeichnen-Modus. |
| **Quick-Marker-Menü** | SIDCs verschachteltes `Strg+T`-Menü zum Wählen und Setzen eines Symbols. |
| **K23** | SMX-Armhalterung an der Weste, mit eigenem Eingabekontext. |
| **TabMode** | SMX: Anzeige-Variante — OFFtab / ONtab / HANDtab / BIGtab / MINITab / PAPERMap / GMmode. |
| **Mode** | SMX: Inhalts-Modus — OFF / ON / GPS / CHAT / SETTINGS / FEEDS. |
| **Enforce Script** | Die C#/C++-ähnliche Skriptsprache der Enfusion-Engine (`.c`-Dateien). |
| **Workbench** | Der Arma-Reforger-Mod-Editor; der einzige Build-/Test-Weg für diese Mods. |
| **`$profile:`** | Pfad-Präfix für das Spieler- (oder Dedicated-Server-) Profilverzeichnis. |
| **`RplProp` / `Replication.BumpMe()`** | Enfusion-Replikation: Member für Sync markieren / Sync erzwingen. |
| **`modded class` / `modded enum`** | Enforce-Mechanismus zum Erweitern/Überschreiben einer bestehenden Engine-Klasse/-Enum. |
| **`[BaseContainerProps]`** | Attribut, das eine Klasse Workbench-konfigurierbar macht (`.conf`-Dateien). |
| **`GetInstance()`** | Das Singleton-Zugriffsmuster in allen drei Mods. |
| **`Settings.json`** | SIDC: die admin-editierbare Server-Settings-Datei in `$profile:SIDC_Framework/ServerSettings/`. |
| **`SMX_ConfigV1.Json`** | SMX: die server-geladene, an Clients replizierte Config-Datei. |
| **QWERTZ-Hinweis** | `KC_*`-Tastenkonstanten sind US-QWERTY-Scancode-Positionen; auf einer deutschen Tastatur sind Y/Z physisch vertauscht. |
