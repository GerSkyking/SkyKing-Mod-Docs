# SkyMap-X — Workflows (step by step)

Which keys and actions, in which order, get you to your goal. All keys are the defaults (rebindable under *Settings → Keybindings → SkyMap-X*, see [Keybinds](Keybinds)). Items are listed in [SkyMap-X → Items / Prefabs](SkyMap-X#items--prefabs).

> **Clicking on the tablet:** In **Big mode** (`Shift+E`) the mouse cursor is always on. In **Hand** and **Mini** mode turn it on with `Alt+Ctrl` first. Then left‑click buttons, lists, checkboxes and text fields.

## 0. Basics: switch on and pick a view

Every other workflow starts here.

```mermaid
flowchart TD
    A["Tablet / K23 in inventory or on vest"] --> B{"Tablet on?"}
    B -- "no" --> C["Ctrl+F1: power on<br/>(≈2 s start-up)"]
    B -- "yes" --> D
    C --> D{"Which view?"}
    D -- "Hand" --> E["Ctrl+F5: take into hand"]
    D -- "Big (≈85 % screen)" --> F["Shift+E<br/>cursor is on automatically"]
    D -- "Mini (corner)" --> G["Shift+Q"]
    E --> H["Ctrl+1 GPS · Ctrl+2 Chat<br/>Ctrl+3 Settings · Ctrl+4 Feeds"]
    F --> H
    G --> H
```

Tip: *Game Settings → SkyMap-X → Auto start / Auto mode* powers the tablet on at mission start and opens your default mode.

## 1. Set up channels (once)

Chat contacts, feeds and BFT markers only show devices whose **sending channels** match your **receiving channels**. Without matching channels the lists stay empty.

```mermaid
flowchart TD
    A["Tablet on (see 0)"] --> B["Ctrl+3: Settings"]
    B --> C["Cursor on<br/>(Big: automatic · Hand/Mini: Alt+Ctrl)"]
    C --> D["Set Device ID / Info<br/>(may be locked or forced by the server)"]
    D --> E["Tick send / receive channels:<br/>Faction, CIV, sub-channel 1–4<br/>(sub-channel = free text, same text = same channel)"]
    E --> F["Click Save"]
```

## 2. GPS map

```mermaid
flowchart TD
    A["Tablet on (see 0)"] --> B["Ctrl+1: GPS"]
    B --> C["Page Down / Page Up: zoom in / out"]
    B --> D["Right-Ctrl + arrow keys: pan"]
    B --> E["Ctrl+F2 / Ctrl+F3: center / lock on own position"]
    B --> F["Ctrl+Page Up / Down: brightness"]
    B --> G["BFT markers of other devices<br/>appear by themselves (channels, see 1)"]
    B --> H["M: full-screen map<br/>(tablet as paper map, or your own map is taken into hand)"]
```

## 3. Write a chat message

```mermaid
flowchart TD
    A["Tablet on (see 0),<br/>channels set (see 1)"] --> B["Ctrl+2: Chat"]
    B --> C["Cursor on<br/>(Big: automatic · Hand/Mini: Alt+Ctrl)"]
    C --> D{"Contact in the list?"}
    D -- "no" --> E["Click Refresh<br/>or type ≥3 letters into the search field"]
    E --> D
    D -- "yes" --> F["Click the contact<br/>(conversation opens, unread counter resets)"]
    F --> G["Click the message field, type"]
    G --> H["Click Send"]
```

Unread messages are counted per contact in the list; opening the contact resets the counter.

## 4. Watch a helmet camera

```mermaid
flowchart TD
    subgraph W["Wearer"]
        A["Wear helmet camera<br/>(SCX_Cam_Helm_* or PONOS)"] --> B["Camera is active automatically<br/>while the wearer carries it"]
    end
    subgraph V["Viewer"]
        C["Tablet on (see 0),<br/>receive channel matches (see 1)"] --> D["Ctrl+4: Feeds"]
        D --> E["Cursor on, click the device<br/>in the list (Refresh / search if missing)"]
        E --> F["Live image; zoom: Page Down / Up<br/>brightness: Ctrl+Page Up / Down"]
        F --> G{"Swivelling camera?"}
        G -- "yes" --> H["Swivel: Right-Ctrl + arrow keys<br/>or the on-screen arrows · Lock button"]
        G -- "no (fixed)" --> I["Image follows the wearer's view"]
    end
    B --> E
```

There are **fixed** and **swivelling** helmet cameras (set per item). A fixed one always shows where the wearer is looking; a swivelling one can also be turned from the tablet.

## 5. Use the ARC‑40 camera grenade

```mermaid
flowchart TD
    A["Have ARC-40 rounds<br/>(M320: SCX_Cam_40mm_UGL_* · GP-25: SCX_Cam-40mm_UGL_GP_*)"] --> B["Load into the underbarrel launcher<br/>(like any 40 mm round)"]
    B --> C["Fire high over the target area"]
    C --> D["After ≈1.5 s the parachute camera deploys<br/>and sinks slowly, looking straight down"]
    D --> E["Tablet: Ctrl+4 Feeds"]
    E --> F["Click the ARC-40 camera in the list<br/>(Refresh if missing)"]
    F --> G["Swivel: Right-Ctrl + arrow keys or the on-screen arrows<br/>zoom: Page Down / Up · Lock button: lock the view"]
```

Everyone on a matching receive channel sees the ARC‑40 in their Feeds list — you can spot for the whole squad.

---

# SkyMap-X — Abläufe (Schritt für Schritt)

Welche Tasten und Aktionen in welcher Reihenfolge zum Ziel führen. Alle Tasten sind die Standardbelegung (änderbar unter *Einstellungen → Tastenbelegung → SkyMap-X*, siehe [Keybinds](Keybinds)). Die Items stehen unter [SkyMap-X → Items / Prefabs](SkyMap-X#items--prefabs-1).

> **Klicken am Tablet:** Im **Big-Modus** (`Shift+E`) ist der Mauszeiger immer an. Im **Hand**- und **Mini**-Modus zuerst mit `Alt+Strg` einschalten. Dann per Linksklick Buttons, Listen, Checkboxen und Textfelder bedienen.

## 0. Grundlage: Einschalten und Ansicht wählen

Jeder andere Ablauf beginnt hier.

```mermaid
flowchart TD
    A["Tablet / K23 im Inventar oder an der Weste"] --> B{"Tablet an?"}
    B -- "nein" --> C["Strg+F1: einschalten<br/>(≈2 s Startzeit)"]
    B -- "ja" --> D
    C --> D{"Welche Ansicht?"}
    D -- "Hand" --> E["Strg+F5: in die Hand nehmen"]
    D -- "Big (≈85 % Bildschirm)" --> F["Shift+E<br/>Cursor automatisch an"]
    D -- "Mini (Ecke)" --> G["Shift+Q"]
    E --> H["Strg+1 GPS · Strg+2 Chat<br/>Strg+3 Settings · Strg+4 Feeds"]
    F --> H
    G --> H
```

Tipp: *Spiel-Einstellungen → SkyMap-X → Auto-Start / Auto-Modus* schaltet das Tablet bei Missionsstart ein und öffnet deinen Standardmodus.

## 1. Kanäle einrichten (einmalig)

Chat-Kontakte, Feeds und BFT-Marker zeigen nur Geräte, deren **Sendekanäle** zu deinen **Empfangskanälen** passen. Ohne passende Kanäle bleiben die Listen leer.

```mermaid
flowchart TD
    A["Tablet an (siehe 0)"] --> B["Strg+3: Settings"]
    B --> C["Cursor an<br/>(Big: automatisch · Hand/Mini: Alt+Strg)"]
    C --> D["Geräte-ID / -Info setzen<br/>(kann vom Server gesperrt oder vorgegeben sein)"]
    D --> E["Sende- / Empfangskanäle anhaken:<br/>Fraktion, CIV, Subkanal 1–4<br/>(Subkanal = Freitext, gleicher Text = gleicher Kanal)"]
    E --> F["Save klicken"]
```

## 2. GPS-Karte

```mermaid
flowchart TD
    A["Tablet an (siehe 0)"] --> B["Strg+1: GPS"]
    B --> C["Bild ab / Bild auf: rein- / rauszoomen"]
    B --> D["Rechts-Strg + Pfeiltasten: verschieben"]
    B --> E["Strg+F2 / Strg+F3: auf eigene Position zentrieren / sperren"]
    B --> F["Strg+Bild auf / ab: Helligkeit"]
    B --> G["BFT-Marker anderer Geräte<br/>erscheinen von selbst (Kanäle, siehe 1)"]
    B --> H["M: Vollbildkarte<br/>(Tablet als Papierkarte oder eigene Karte wird in die Hand genommen)"]
```

## 3. Chat-Nachricht schreiben

```mermaid
flowchart TD
    A["Tablet an (siehe 0),<br/>Kanäle eingerichtet (siehe 1)"] --> B["Strg+2: Chat"]
    B --> C["Cursor an<br/>(Big: automatisch · Hand/Mini: Alt+Strg)"]
    C --> D{"Kontakt in der Liste?"}
    D -- "nein" --> E["Refresh klicken<br/>oder ≥3 Zeichen ins Suchfeld tippen"]
    E --> D
    D -- "ja" --> F["Kontakt anklicken<br/>(Verlauf öffnet sich, Ungelesen-Zähler wird zurückgesetzt)"]
    F --> G["Nachrichtenfeld anklicken, tippen"]
    G --> H["Send klicken"]
```

Ungelesene Nachrichten werden pro Kontakt in der Liste gezählt; Öffnen des Kontakts setzt den Zähler zurück.

## 4. Helmkamera ansehen

```mermaid
flowchart TD
    subgraph W["Träger"]
        A["Helmkamera tragen<br/>(SCX_Cam_Helm_* oder PONOS)"] --> B["Kamera ist automatisch aktiv,<br/>solange der Träger sie bei sich hat"]
    end
    subgraph V["Zuschauer"]
        C["Tablet an (siehe 0),<br/>Empfangskanal passt (siehe 1)"] --> D["Strg+4: Feeds"]
        D --> E["Cursor an, Gerät in der Liste anklicken<br/>(fehlt es: Refresh / Suche)"]
        E --> F["Live-Bild; Zoom: Bild ab / auf<br/>Helligkeit: Strg+Bild auf / ab"]
        F --> G{"Schwenkbare Kamera?"}
        G -- "ja" --> H["Schwenken: Rechts-Strg + Pfeiltasten<br/>oder Pfeil-Buttons · Lock-Button"]
        G -- "nein (fest)" --> I["Bild folgt dem Blick des Trägers"]
    end
    B --> E
```

Es gibt **feste** und **schwenkbare** Helmkameras (pro Item eingestellt). Eine feste zeigt immer, wohin der Träger schaut; eine schwenkbare lässt sich zusätzlich vom Tablet aus drehen.

## 5. ARC-40-Kameragranate benutzen

```mermaid
flowchart TD
    A["ARC-40-Munition dabei<br/>(M320: SCX_Cam_40mm_UGL_* · GP-25: SCX_Cam-40mm_UGL_GP_*)"] --> B["In den Unterlaufgranatwerfer laden<br/>(wie jede 40-mm-Granate)"]
    B --> C["Hoch über das Zielgebiet schießen"]
    C --> D["Nach ≈1,5 s öffnet sich die Fallschirmkamera<br/>und sinkt langsam, Blick senkrecht nach unten"]
    D --> E["Tablet: Strg+4 Feeds"]
    E --> F["ARC-40-Kamera in der Liste anklicken<br/>(fehlt sie: Refresh)"]
    F --> G["Schwenken: Rechts-Strg + Pfeiltasten oder Pfeil-Buttons<br/>Zoom: Bild ab / auf · Lock-Button: Blick sperren"]
```

Jeder mit passendem Empfangskanal sieht die ARC-40 in seiner Feeds-Liste — du kannst so für den ganzen Trupp aufklären.
