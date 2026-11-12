# SIDC – C2 — Map & tools

**What is this page about?** Reference for the plan view: toolbar and keys, top bar, timeline, marker wizard, text boxes and images, the *Map* and *Layers* menus (incl. contour lines/elevation points), search and settings. **For whom?** Everyone who works on the map. Step‑by‑step guides are on [Workflows](SIDC-C2-Workflows); overview: [SIDC – C2](SIDC-C2-Command-Control).

## 1. Toolbar (left)

The drawing tools appear only with edit rights; *Map*, *Layers*, *ORBAT in plan* are always there.

| Tool | Key | What it does |
|---|---|---|
| **Pan the map** | `W` | pan/zoom/rotate; click a marker = edit; middle mouse button on a marker drags it; select lines/measure lines; right‑click = menu with the other tools at that spot (right‑drag rotates) |
| **Point** | `A` | your cursor appears for everyone (player colour, yellow ring), map is locked; right mouse button draws an arrow. Also for viewers |
| **Draw line** | `D` | straight segments; double‑click / green tick = done, right‑click / Esc = discard; clicking the green start point (from 3 points) closes it to a **named area**; colour, width, phase, *More phases*, layer in the menu |
| **Move marker** | `M` | drag markers with the left button (mainly touch; the button shows on mobile) |
| **Ruler** | `F` | any number of points, segment + total length (with 3D height); double‑click / green tick = done, right‑click / Esc = discard; measure lines stay, with phase/layer menu |
| **Eraser** | `S` | deletes markers, lines and measure lines inside a circle (size adjustable, panel movable/pinnable) |
| **Text box** | `T` | Markdown note on the map (collapsible, fixed size or scaling with the map, resize at the edges) |
| **Place marker** | `Space` | click the position first, *then* the wizard opens; cancelling keeps the mode |
| **Mark a spot** | – | mission builders, project plans only: drops a **star**, then asks *What is here?* — see [Workflows](SIDC-C2-Workflows) |
| **Map** | – | base‑map menu (section 6) |
| **Layers** | – | layer menu (section 7) |
| **ORBAT in plan** | – | linked ORBATs, place units on the map — see [ORBAT](SIDC-C2-ORBAT) |
| **Favorites** | `R` | place saved markers (with colour) |

The middle mouse button pans the map in every other tool. Panels can be moved and pinned.

### Other keys

| Key | Action |
|---|---|
| `Q` / `E` | undo / redo (own actions: move, edit, lock, lines) |
| `X` / `C` / `V` | cut / copy / paste marker |
| `←` / `→` | previous / next phase |
| `Y` | phase notes |
| `N` | map to north |
| `Ctrl+F` or `/` | search the plan |
| `Esc` | cancel / close window |

All tool keys are rebindable: gear → *Keyboard shortcuts (tools)* (click a key, press the new key, `Esc` cancels). The **?** button (*Help – keys & functions*) always shows the current layout.

## 2. Top bar

| Element | Function |
|---|---|
| **3D** | tilt on/off (in 2D only rotation) |
| **Lock Rotation** / compass **↑** | lock map rotation (always north‑up) / click = face north |
| **Undo / Redo** | as `Q` / `E` |
| **Date/time** | timestamp for screenshot and briefing (empty = now) |
| **Camera** | screenshot of the map (no controls); ▾ = options *Resolution* / *Format* |
| **Briefing** | *Briefing as PDF (one page per phase)* |
| **Map label language** | language of the place names (13 languages, independent of the UI language) |
| **Magnifier** | *Search the plan* (section 9) |
| **Presentation mode** | full screen, map + phase switcher only |
| **Version history** | save / restore versions |
| **Help** | keys & functions |
| **Gear** | settings (section 10) |
| **Access / sharing** | permissions and public links (owner) |
| Cursor display (right) | world X / Y and elevation under the mouse pointer (also in 2D) |
| **UI Scale** | marker and text size just for you (collapsible box) |

On narrow screens part of the top bar moves into a *Menu* button.

## 3. Timeline (below the top bar)

- **Phases** as bars: click = select, drag = move, edges = duration, double‑click = phase notes. **+ Phases** adds a player phase. The first phase **Base** is locked. *H‑hour (operation start)* sets the time reference.
- **Two lanes:** player phases (top) and builder phases (bottom, mission builders only; extra *Sub‑phases* such as 1.1 possible).
- **View switch** *acting as*: **Player** · **Builder** · **Zeus** (see [Roles & visibility](SIDC-C2-Roles-Visibility)).
- Sliders: *Visibility of other phases* (0 % = hidden) and *Opacity player ↔ builder*.
- Zoom with the +/– buttons or `Ctrl` + mouse wheel. On phones the timeline is replaced by a phase dropdown.
- **Phase notes:** movable, resizable window, Markdown with images, one tab per phase, auto‑saved. For builder phases the release options are here (*Release to players*, *Release to Zeus*).

## 4. Marker wizard

| Part | Content |
|---|---|
| **Quick menu** | affiliation → category (marked with ↳) → symbol; **Place** button also at the top next to *← back* |
| **Catalog** | full search through `SIDC_AllMarkersCatalog.json` |
| **Marker builder** | dropdowns for category / marker / identity / echelon / modifiers, each with search, live preview + SIDC display; *SIDC number (full or partial)* → *Place best match* |
| Fields | *Unit text (amplifier)*, *AI / extra text*, *Direction*, colour, *Tactical layer* (default *Auto*), *Locked*, *Timestamp* |
| **Advanced (modifiers)** | dropdowns *Modifier 1*, *Modifier 2*, *HQ / Task Force / Dummy*, *Condition / capability* — read from `SIDC_ModifierCatalog.json` |

Only the four affiliations friend / hostile / neutral / unknown. Line symbols (and line‑building favorites) repeat until a **right‑click**. **Text‑only markers** (*Special → Clear for Text*) show only text with a free text colour. A marker colour tints the black parts of the symbol (frame colours stay).

### Label, hover, edit

- **Permanent label** under every marker: unit text (else the catalog symbol name) + the set modifiers. Toggle in gear → *Marker display* → *Show marker text*.
- **Hover:** creator, phase and layer + quick buttons **Delete** (with confirmation) · **Move** · **Edit**. *Detailed marker info on hover* (gear) adds the long description.
- **Edit window** (click outside / `Esc` closes): unit & extra text, *Icon rotation (°)*, *Icon size (everyone)*, phase + *More phases*, layer, *Locked*, modifiers, *ORBAT node* (mission builders), **Clone**, **★ Favorite**, **Delete**.

### Symbol rendering

Primary: **milsymbol.js** (`standard: "APP6"`, 30‑digit SIDC). Fallback: a pre‑rendered PNG from the catalog. Last resort: a coloured dot. **Direction arrows** are vector geometry, oriented like ATAKmaps (air = extended line + arrowhead, ground = fixed stem + rotating arrow).

| SIDC position (0‑based) | Field | Set by |
|---|---|---|
| 3 | affiliation (identity) | marker selection |
| 6 | status / condition | modifier 4 |
| 7 | HQ / task force / dummy | modifier 3 |
| 8–9 | echelon | land units only |
| 10–15 | main icon (entity/type/subtype) | catalog |
| 16–17 / 18–19 | modifier 1 / 2 | *Advanced* |

## 5. Text boxes, Markdown toolbar, images

- **Text box** (`T`): Markdown; empty boxes are discarded on save. In project plans mission builders choose *Visible to* (see [Roles & visibility](SIDC-C2-Roles-Visibility)). Link codes (`{{doc:…}}` …) render as chips; the toolbar button *Insert link* inserts them.
- **Markdown toolbar** (keyboard icon above every Markdown field): bold, italic, strikethrough, highlight, code, maths, comments, headings, lists, task lists, quote, callouts, divider, table, link.
- **Images** (side panel *Images*): *Upload image* (PNG, JPEG, GIF, WebP), caption (hover tooltip), Markdown note, phase; drag onto the map to place — several placements, move/scale, *Scale with the map*; click = lightbox. Images automatically belong to the plan's project.

## 6. Menu *Map* (base map)

Satellite, *Sat (simplified)*, Grid, Buildings, Roads & trails, Water, Power lines, Trees, Places, Contour lines, Elevation points — each with an **opacity slider** (stored per user/browser). The layers come from the map package; missing files simply do not show.

### Contour lines & elevation points

Both are derived from the game heightmap, **pre‑computed once in the ATAKmaps importer** (`pipeline/terrain_features.py`, numpy/scipy only) and shipped in the map package as `contours.geojson` / `peaks.geojson`. ATAKmaps viewer and C2 only display them; default **off**.

| Layer | How it is built |
|---|---|
| **Contour lines** | vectorised marching‑squares on a 2 m grid, one line every 10 m, every 5th line bold + labelled (white text, black outline), Douglas‑Peucker‑simplified |
| **Elevation points** | *local* prominence (Prim bottleneck search within a radius) on a 4 m grid, hard prominence cut‑off + spatial non‑maximum suppression, rough shape class (hill / ridge / plateau / dominant peak); rendered as `▲ NNN m` |

The parameters (contour interval, bold interval, min. prominence, min. distance, search radius, work‑cell size) are in the importer's *Heightmap CSV* box. The button **"Nur Höhenlinien / Höhenpunkte neu berechnen"** regenerates the two files without re‑tiling; then re‑export the map package and re‑upload it to C2. For **more** elevation points: lower *min. distance* (biggest lever, e.g. 250 → 130) and *min. prominence* (e.g. 20 → 12), optionally the *search radius* (300 → 150).

## 7. Menu *Layers*

- **Automatic** (by identity): BLUFOR · REDFOR · Civilian / GREENFOR · Unknown.
- **Custom layers** per plan (*New layer …* → *Create*; editors). Markers, lines and measure lines are assigned via *Tactical layer* (*Auto* = automatic layer). Deleting a layer keeps its elements.
- In project plans for mission builders: **Builder notes & stars**.
- Checkboxes and opacity apply **only to you**.

## 8. ViewCube & navigation

ViewCube bottom right: *Top view* / N / S / E / W / corners, drag = rotate/tilt, home = default view (north, top‑down). Map boundaries keep the camera on the map.

## 9. Search the plan

Magnifier, `Ctrl+F` or `/`. Searches markers (label, extra text, symbol name), notes and star pins (incl. linked places/titles), named lines/areas, map images (name, file, note), map places and phases — case‑insensitive, umlaut‑tolerant, text start before word start before partial match. Result list with type icon, name and phase/layer; arrow keys / Enter / click fly there and highlight ~3 s; switches phase or shows a hidden layer if needed. Only what your current view may show. Step by step: [Workflows → search](SIDC-C2-Workflows).

## 10. Settings (gear)

| Section | Content |
|---|---|
| Colour scheme / language | dark / light / system; DE / EN |
| *Display name* | shown to others instead of the login name |
| *Keyboard shortcuts (tools)* | rebind keys, *Reset* |
| *Top bar size* | scale of the upper bar |
| *Marker display* | *Show marker text*, *Detailed marker info on hover* |
| Map | adjust sat image (brightness, contrast, saturation), road colours |
| *Notifications* | external channels — see [Chat & notifications](SIDC-C2-Chat-Notifications) |
| *API tokens* | tokens for the ATAKmaps client — see [Admin & install](SIDC-C2-Admin-Install) |
| *My data* | *Download my data (.zip)*, *Delete account* |

---

# SIDC – C2 — Karte & Werkzeuge

**Worum geht's?** Nachschlagen für die Plan-Ansicht: Werkzeugleiste und Tasten, Topbar, Zeitstrahl, Marker-Wizard, Textfelder und Bilder, die Menüs *Karte* und *Layer* (inkl. Höhenlinien/Höhenpunkte), Suche und Einstellungen. **Für wen?** Alle, die auf der Karte arbeiten. Schritt-für-Schritt-Anleitungen stehen unter [Abläufe](SIDC-C2-Workflows); Überblick: [SIDC – C2](SIDC-C2-Command-Control).

## 1. Werkzeugleiste (links)

Die Zeichenwerkzeuge erscheinen nur mit Bearbeitungsrecht; *Karte*, *Layer*, *ORBAT im Plan* sind immer da.

| Werkzeug | Taste | Was es tut |
|---|---|---|
| **Karte bewegen** | `W` | schieben/zoomen/drehen; Marker anklicken = bearbeiten; mittlere Maustaste auf einem Marker zieht ihn; Linien/Messlinien auswählen; Rechtsklick = Menü mit den anderen Werkzeugen an dieser Stelle (Rechts-Ziehen dreht) |
| **Zeigen** | `A` | dein Cursor erscheint bei allen (Spielerfarbe, gelber Ring), Karte fixiert; rechte Maustaste zieht einen Pfeil. Auch für Betrachter |
| **Linie zeichnen** | `D` | gerade Segmente; Doppelklick / grüner Haken = fertig, Rechtsklick / Esc = verwerfen; Klick auf den grünen Startpunkt (ab 3 Punkten) schließt zur **benannten Fläche**; Farbe, Stärke, Phase, *Weitere Phasen*, Layer im Menü |
| **Marker verschieben** | `M` | Marker mit der linken Taste ziehen (v. a. Touch; Knopf erscheint am Handy) |
| **Lineal** | `F` | beliebig viele Punkte, Teil- und Gesamtstrecke (mit 3D-Höhe); Doppelklick / grüner Haken = fertig, Rechtsklick / Esc = verwerfen; Messlinien bleiben stehen, mit Phase/Layer-Menü |
| **Radierer** | `S` | löscht Marker, Linien und Messlinien im Kreis (Größe einstellbar, Panel verschieb-/fixierbar) |
| **Textfeld** | `T` | Markdown-Notiz auf der Karte (einklappbar, feste Größe oder mit der Karte skalierend, an den Rändern größenverstellbar) |
| **Marker setzen** | `Leertaste` | erst Position klicken, *dann* öffnet der Wizard; Abbrechen lässt den Modus aktiv |
| **Ort markieren** | – | Missionsbau, nur Projekt-Pläne: setzt einen **Stern**, dann *Was ist hier?* — siehe [Abläufe](SIDC-C2-Workflows) |
| **Karte** | – | Menü Grundkarte (Abschnitt 6) |
| **Layer** | – | Layer-Menü (Abschnitt 7) |
| **ORBAT im Plan** | – | verknüpfte ORBATs, Einheiten auf die Karte setzen — siehe [ORBAT](SIDC-C2-ORBAT) |
| **Favoriten** | `R` | gespeicherte Marker (mit Farbe) setzen |

Die mittlere Maustaste schiebt in allen anderen Werkzeugen die Karte. Panels lassen sich verschieben und anheften.

### Weitere Tasten

| Taste | Aktion |
|---|---|
| `Q` / `E` | Rückgängig / Wiederholen (eigene Aktionen: verschieben, bearbeiten, sperren, Linien) |
| `X` / `C` / `V` | Marker ausschneiden / kopieren / einfügen |
| `←` / `→` | Phase zurück / vor |
| `Y` | Phasen-Notizen |
| `N` | Karte nach Norden |
| `Strg+F` oder `/` | im Plan suchen |
| `Esc` | abbrechen / Fenster schließen |

Alle Werkzeug-Tasten sind umbelegbar: Zahnrad → *Tastenkürzel (Werkzeuge)* (Taste anklicken, neue Taste drücken, `Esc` bricht ab). Der Knopf **?** (*Hilfe – Tasten & Funktionen*) zeigt immer die aktuelle Belegung.

## 2. Topbar

| Element | Funktion |
|---|---|
| **3D** | Kippen an/aus (in 2D nur drehen) |
| **Lock Rotation** / Kompass **↑** | Kartendrehung sperren (immer genordet) / Klick = nach Norden ausrichten |
| **Rückgängig / Wiederholen** | wie `Q` / `E` |
| **Datum/Zeit** | Zeitstempel für Screenshot und Briefing (leer = jetzt) |
| **Kamera** | Screenshot der Karte (ohne Bedienelemente); ▾ = Optionen *Auflösung* / *Format* |
| **Briefing** | *Briefing als PDF (je Phase eine Seite)* |
| **Sprache der Kartenbeschriftung** | Sprache der Ortsnamen (13 Sprachen, unabhängig von der UI-Sprache) |
| **Lupe** | *Im Plan suchen* (Abschnitt 9) |
| **Präsentationsmodus** | Vollbild, nur Karte + Phasen-Umschalter |
| **Versionsverlauf** | Versionen sichern / wiederherstellen |
| **Hilfe** | Tasten & Funktionen |
| **Zahnrad** | Einstellungen (Abschnitt 10) |
| **Zugriff / Freigaben** | Rechte und öffentliche Links (Owner) |
| Cursor-Anzeige (rechts) | Welt-X / Y und Höhe unter dem Mauszeiger (auch in 2D) |
| **UI Scale** | Marker- und Textgröße nur für dich (einklappbare Box) |

Auf schmalen Bildschirmen wandert ein Teil der Topbar in einen Knopf *Menü*.

## 3. Zeitstrahl (unter der Topbar)

- **Phasen** als Balken: Klick = wählen, Ziehen = verschieben, Kanten = Dauer, Doppelklick = Phasen-Notizen. **+ Phasen** legt eine Spieler-Phase an. Die erste Phase **Base** ist gesperrt. *H-Stunde (Operationsbeginn)* setzt den Zeitbezug.
- **Zwei Spuren:** Spieler-Phasen (oben) und Missionsbau-Phasen (unten, nur Missionsbau; zusätzliche *Zwischenphasen* wie 1.1 möglich).
- **Ansichts-Umschalter** *wirke als*: **Spieler** · **Missionsbau** · **Zeus** (siehe [Rollen & Sichtbarkeit](SIDC-C2-Roles-Visibility)).
- Regler: *Sichtbarkeit fremder Phasen* (0 % = ausgeblendet) und *Transparenz Spieler ↔ Missionsbau*.
- Zoom mit den +/–-Knöpfen oder `Strg` + Mausrad. Am Handy ersetzt eine Phasen-Auswahl den Zeitstrahl.
- **Phasen-Notizen:** verschiebbares, größenverstellbares Fenster, Markdown mit Bildern, ein Reiter je Phase, automatisch gespeichert. Bei Missionsbau-Phasen stehen hier die Freigaben (*Für Spieler freigeben*, *Für Zeus freigeben*).

## 4. Marker-Wizard

| Teil | Inhalt |
|---|---|
| **QuickMenü** | Zugehörigkeit → Kategorie (mit ↳ markiert) → Symbol; Knopf **Platzieren** auch oben neben *← zurück* |
| **Katalog** | volle Suche in `SIDC_AllMarkersCatalog.json` |
| **Marker-Baukasten** | Dropdowns für Kategorie / Marker / Identität / Echelon / Modifikatoren, je mit Suche, Live-Vorschau + SIDC-Anzeige; *SIDC-Nummer (ganz oder teilweise)* → *Passenden Marker setzen* |
| Felder | *Einheitstext (Amplifier)*, *KI-/Zusatztext*, *Richtung*, Farbe, *Taktischer Layer* (Standard *Auto*), *Gesperrt*, *Zeitstempel* |
| **Erweitert (Modifikatoren)** | Dropdowns *Modifikator 1*, *Modifikator 2*, *HQ / Task Force / Dummy*, *Zustand / Einsatzbereitschaft* — gelesen aus `SIDC_ModifierCatalog.json` |

Nur die vier Zugehörigkeiten Freund / Feind / Neutral / Unbekannt. Linien-Symbole (und linienbildende Favoriten) wiederholen bis zum **Rechtsklick**. **Nur-Text-Marker** (*Spezial → Clear for Text*) zeigen nur Text mit frei wählbarer Textfarbe. Eine Markerfarbe färbt die schwarzen Symbolanteile (Rahmenfarben bleiben).

### Beschriftung, Hover, Bearbeiten

- **Dauerhafte Beschriftung** unter jedem Marker: Einheitstext (sonst der Symbolname aus dem Katalog) + die gesetzten Modifikatoren. Schalter: Zahnrad → *Marker-Anzeige* → *Marker-Texte anzeigen*.
- **Hover:** Ersteller, Phase und Layer + Schnellknöpfe **Löschen** (mit Bestätigung) · **Verschieben** · **Bearbeiten**. *Ausführliche Marker-Info beim Überfahren* (Zahnrad) blendet die lange Beschreibung ein.
- **Bearbeiten-Fenster** (Klick außerhalb / `Esc` schließt): Einheits- & Zusatztext, *Icon-Drehung (°)*, *Icon-Größe (für alle)*, Phase + *Weitere Phasen*, Layer, *Gesperrt*, Modifikatoren, *ORBAT-Knoten* (Missionsbau), **Klonen**, **★ Favorit**, **Löschen**.

### Symbol-Rendering

Primär: **milsymbol.js** (`standard: "APP6"`, 30-stelliges SIDC). Fallback: ein vorgerendertes PNG aus dem Katalog. Letzte Stufe: ein farbiger Punkt. **Richtungspfeile** als Vektor-Geometrie, orientiert wie ATAKmaps (Luft = verlängerte Linie + Pfeilspitze, Boden = fester Steg + drehender Pfeil).

| SIDC-Stelle (0-basiert) | Feld | gesetzt durch |
|---|---|---|
| 3 | Zugehörigkeit (Identität) | Marker-Auswahl |
| 6 | Status / Zustand | Modifikator 4 |
| 7 | HQ / Task Force / Dummy | Modifikator 3 |
| 8–9 | Echelon | nur Land-Einheiten |
| 10–15 | Haupticon (Entity/Typ/Subtyp) | Katalog |
| 16–17 / 18–19 | Modifikator 1 / 2 | *Erweitert* |

## 5. Textfelder, Markdown-Leiste, Bilder

- **Textfeld** (`T`): Markdown; leere Felder werden beim Speichern verworfen. In Projekt-Plänen wählen Missionsbauer *Sichtbar für* (siehe [Rollen & Sichtbarkeit](SIDC-C2-Roles-Visibility)). Verknüpfungs-Codes (`{{doc:…}}` …) erscheinen als Chips; der Leisten-Knopf *Verweis einfügen* setzt sie ein.
- **Markdown-Leiste** (Tastatur-Symbol über jedem Markdown-Feld): fett, kursiv, durchgestrichen, hervorheben, Code, Mathe, Kommentare, Überschriften, Listen, Aufgabenlisten, Zitat, Hinweise, Trennlinie, Tabelle, Link.
- **Bilder** (Seitenleiste *Bilder*): *Bild hochladen* (PNG, JPEG, GIF, WebP), Beschriftung (Hover-Tooltip), Markdown-Notiz, Phase; zum Platzieren auf die Karte ziehen — mehrere Platzierungen, verschieben/skalieren, *Mit der Karte skalieren*; Klick = Großansicht. Bilder gehören automatisch zum Projekt des Plans.

## 6. Menü *Karte* (Grundkarte)

Satellit, *Sat (vereinfacht)*, Raster, Gebäude, Straßen & Wege, Gewässer, Stromleitungen, Bäume, Orte, Höhenlinien, Höhenpunkte — je mit **Deckkraft-Regler** (pro Nutzer/Browser gespeichert). Die Ebenen kommen aus dem Kartenpaket; fehlende Dateien erscheinen einfach nicht.

### Höhenlinien & Höhenpunkte

Beide stammen aus der Spiel-Heightmap, **einmalig im ATAKmaps-Importer vorberechnet** (`pipeline/terrain_features.py`, nur numpy/scipy) und als `contours.geojson` / `peaks.geojson` im Kartenpaket. ATAKmaps-Viewer und C2 zeigen sie nur an; Standard **aus**.

| Ebene | Erzeugung |
|---|---|
| **Höhenlinien** | vektorisiertes Marching-Squares auf 2-m-Raster, alle 10 m eine Linie, jede 5. fett + beschriftet (weiße Schrift, schwarze Umrandung), Douglas-Peucker-vereinfacht |
| **Höhenpunkte** | *lokale* Prominenz (Prim-Bottleneck-Suche im Radius) auf 4-m-Raster, harter Prominenz-Cutoff + räumliches NMS, grobe Formklasse (Kuppe / Grat / Plateau / dominanter Gipfel); Anzeige `▲ NNN m` |

Die Parameter (Linienabstand, Fett-Intervall, min. Prominenz, min. Abstand, Suchradius, Arbeitsraster) stehen in der Importer-Box *Heightmap CSV*. Der Button **„Nur Höhenlinien / Höhenpunkte neu berechnen"** erzeugt die zwei Dateien ohne neues Kacheln; danach Kartenpaket neu exportieren und in C2 neu hochladen. Für **mehr** Höhenpunkte: *min. Abstand* senken (größter Hebel, z. B. 250 → 130) und *min. Prominenz* (z. B. 20 → 12), optional den *Suchradius* (300 → 150).

## 7. Menü *Layer*

- **Automatisch** (nach Identität): BLUFOR · REDFOR · Zivil / GREENFOR · Unbekannt.
- **Eigene Layer** pro Plan (*Neuer Layer …* → *Anlegen*; Bearbeiter). Marker, Linien und Messlinien ordnest du über *Taktischer Layer* zu (*Auto* = automatischer Layer). Ein gelöschter Layer lässt seine Elemente bestehen.
- In Projekt-Plänen für den Missionsbau: **Missionsbau-Notizen & Sterne**.
- Haken und Deckkraft gelten **nur für dich**.

## 8. ViewCube & Navigation

ViewCube unten rechts: *Draufsicht* / N / S / O / W / Ecken, Ziehen = drehen/neigen, Home = Standardansicht (Norden, von oben). Kamera-Grenzen halten die Ansicht auf der Karte.

## 9. Im Plan suchen

Lupe, `Strg+F` oder `/`. Durchsucht Marker (Beschriftung, Zusatztext, Symbolname), Notizen und Stern-Pins (inkl. verknüpfter Orte/Titel), benannte Linien/Flächen, Kartenbilder (Name, Datei, Notiz), Kartenorte und Phasen — Groß/klein egal, umlaut-tolerant, Textanfang vor Wortanfang vor Teiltreffer. Trefferliste mit Typ-Symbol, Name und Phase/Layer; Pfeiltasten / Enter / Klick fliegen hin und heben ~3 s hervor; wechselt bei Bedarf die Phase bzw. blendet einen Layer ein. Nur was deine aktuelle Ansicht zeigen darf. Schritt für Schritt: [Abläufe → Suche](SIDC-C2-Workflows).

## 10. Einstellungen (Zahnrad)

| Bereich | Inhalt |
|---|---|
| Farbschema / Sprache | Dunkel / Hell / System; DE / EN |
| *Anzeigename* | wird anderen statt des Login-Namens gezeigt |
| *Tastenkürzel (Werkzeuge)* | Tasten umbelegen, *Zurücksetzen* |
| *Größe der oberen Leiste* | Skalierung der Topbar |
| *Marker-Anzeige* | *Marker-Texte anzeigen*, *Ausführliche Marker-Info beim Überfahren* |
| Karte | Sat-Bild anpassen (Helligkeit, Kontrast, Sättigung), Straßenfarben |
| *Benachrichtigungen* | externe Kanäle — siehe [Chat & Benachrichtigungen](SIDC-C2-Chat-Notifications) |
| *API-Tokens* | Tokens für den ATAKmaps-Client — siehe [Admin & Installation](SIDC-C2-Admin-Install) |
| *Meine Daten* | *Meine Daten herunterladen (.zip)*, *Konto löschen* |
