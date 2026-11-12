# SIDC – C2 — Projects (mission building)

**What is this page about?** The area **Mission building**: projects with files and templates, directory, timeline, plans, task board, milestones and status, feedback (also from outside), whiteboard, activity and the link codes that tie everything together. **For whom?** Mission builders and project members. How a mission comes together step by step: [Workflows → build a mission](SIDC-C2-Workflows); permissions: [Roles & visibility](SIDC-C2-Roles-Visibility).

## 1. Projects and project folders

- Sidebar **Mission building** opens the project list (or the last project you had open).
- **New project** (admins and mission builders): *Project name*, *Description* → **Create project**. You become owner.
- Projects can be sorted into **project folders** (*New folder*, subfolders, drag & drop). Pick a folder in the tree to create the new project there. Deleting a folder moves its content one level up — nothing is deleted.
- **Archive** hides a project from the standard list (*Show archived*) and makes it read‑only; **Restore** reverses it. **Delete project** removes files, tasks and notes — plans stay (without project).
- The search in the project header (`/`) finds titles and contents of files, tasks, notes, plans and images.

## 2. Project tabs

| Tab | Content |
|---|---|
| **Overview** | project status, milestones, open tasks per person, due soon / overdue, recently changed, new feedback |
| **Activity** | who was how active (24 hours / 7 days / 30 days / 1 year) and an activity feed (tasks, files, notes, members) |
| **Mission building** | file tree + Markdown editor, images — **mission builders only** |
| **Directory** | files by type and **On the map** (all stars/notes of the project's plans) |
| **Timeline** | all headings with a time from all files |
| **Plans** | plans of the project (folders, drag & drop, *Move existing plan here*, *Remove from project*) |
| **Tasks** | kanban with discussion and history |
| **Feedback** | feedback from members and outsiders, *Make a task* |
| **Whiteboard** | sticky notes with arrows |
| **Members** | users/groups and their roles |

Older projects with files in the former area *Public* keep them; they stay reachable via links, directory and timeline.

## 3. Files, templates and directory

- Tab **Mission building** → **New document** (or *New folder*). The dialog *New file* offers a **Template**:

  | Template | Document type → directory group |
  |---|---|
  | Empty | no type → *Other* |
  | Mission concept | *Mission concepts* |
  | Order (5‑paragraph) | *Orders and briefings* |
  | Briefing | *Orders and briefings* |
  | Character | *People* |
  | Faction | *Factions* |
  | Place | *Places* |
  | Scene | *Scenes* |

- **Editor:** views *Write* / *Split* / *Preview*; `Ctrl+S` saves; if someone else saved in the meantime you get a conflict notice (*Save my version* / *Load latest version*). Files can be moved by drag & drop and renamed.
- **File tools:** *Comments* (discussion with `@mentions`; viewers may comment) · *History* (the previous version is kept on every save, up to 50 per file; *View*, *Restore* — the current version is saved first) · *Export* (*Download as Markdown*, *Print / PDF*).
- **Directory:** all files grouped by type (*Mission concepts*, *People*, *Factions*, *Places*, *Scenes*, *Orders and briefings*, *Other*), search in titles **and** contents, *New* per group. **On the map** lists all stars and notes of all plans (pink/yellow) with *Open plan*.
- **Images** are uploaded in a plan (image panel) and appear automatically in the project; *Markdown* copies an embed reference.

## 4. Timeline

Headings that start with a time are sorted in automatically (levels 1–6; code blocks are ignored). Click an entry to open the file at that heading.

| Notation | Meaning |
|---|---|
| `## 06:30 Approach` | time on day 1 (also `06:30 Uhr`) |
| `## Tag 2 · 06:30 Attack` / `## T2 06:30 …` / `Day 2 06:30` | time on day 2 |
| `## T+01:30 Enemy contact` / `## H+90 …` / `H+1:30` / `T+90` | relative to mission start (`H+` in minutes) |

Absolute entries are sorted by (day, time) **before** the relative ones.

## 5. Tasks (board)

- Kanban with columns **Open · In progress · Review · Done**; drag cards by the ⋮⋮ grip; filter **All** / **Mine**.
- Task fields: *Title*, *Description (Markdown)*, *Status*, *Priority* (Low / Normal / High), *Assignee*, *Due on* (overdue in red), *Link: plan*, *Link: document*.
- **Discussion** in every task (also for viewers): `@name`, `@group`, `@project` with autocomplete; `Ctrl+Enter` sends. A **History** per task shows who changed what.
- Assigning someone and mentioning someone notifies them (see [Chat & notifications](SIDC-C2-Chat-Notifications)).
- Tasks can also come from a star on the map or from feedback (*Make a task*).

## 6. Project status and milestones

- **Project status** (Overview): *Draft → Review → Test → Release* — editors set it.
- **Milestones**: *Add milestone* with title and optional due date, tick *Done*; the overview shows progress ("x of y done"), overdue ones are highlighted, and the next open date.

## 7. Feedback

- Tab **Feedback** → **New feedback**: *Type* (Bug / Idea / Praise / Other), *Title*, *Description*, optional reference to a plan or file. Every member (also viewers) may submit.
- List with search and filters (*Status*: New / In progress / Done / Rejected; *Type*); source *internal* or *external*. Editors change the status; **Make a task** creates a linked task and opens it.
- **External interface** (owner): *Switch on* creates a link to a public form `#/feedback/<token>` — anyone with the link can send feedback **without login** and only sees the project name. *Copy link*, *Copy curl example*, *Regenerate* (old link stops working), *Switch off*. Archived projects accept nothing.
- Limits: title 200, text 5000, name 80 characters; 10 submissions per IP per 10 minutes, 60 per project per hour, max. 5000 per project. The field `website` is a bot trap and must stay empty. The app's security policy forbids embedding via `<iframe>`; an embedded form needs a proxy exception for `#/feedback/<token>`.

```bash
curl https://c2.example.org/api/public/feedback/<TOKEN>
# -> {"project":"Op Example","kinds":["bug","idea","praise","other"]}
curl -X POST https://c2.example.org/api/public/feedback/<TOKEN> -H "Content-Type: application/json" \
  -d '{"title":"Bridge at 040/040 missing","text":"Mentioned in the briefing, not on the map.","kind":"bug","name":"Max"}'
# -> 201 {"ok":true}   (404 unknown token, 410 project archived, 429 too many requests)
```

- Owners can opt in to a notification for new feedback (see [Chat & notifications](SIDC-C2-Chat-Notifications)).

## 8. Whiteboard

- **Sticky note** in the chosen colour (yellow, green, blue, pink, orange, gray); type directly, drag the header to move, resize, *Change colour*, delete (max. 4000 characters).
- Drag the dot on a note's edge onto another note = **arrow**; notes can *Link a task* or *Link a user*.
- Pan by dragging the free area or with the arrow buttons; zoom, *Fit* (all notes), reset to 100 %.
- Stars on the map can create sticky notes directly (see [Workflows](SIDC-C2-Workflows)).

## 9. Link codes

Anywhere in Markdown (files, tasks, notes, phase notes, text boxes) you can insert:

| Code | Target |
|---|---|
| `{{doc:<id>}}` | file |
| `{{task:<id>}}` | task |
| `{{plan:<id>}}` | plan |
| `{{note:<id>}}` | sticky note |
| `{{img:<id>}}` | image |
| `{{place:<map>:<place name>}}` | place on a map |

- Insert with the toolbar button *Insert link* (search window, maps → places) or *Copy code* on the target.
- They render as a chip with the **current** name; targets that are gone or not visible to you appear as a grey chip *unavailable*.
- Every target shows **Mentioned in …** (backlinks), including stars on maps. In plans, place chips of the same map fly the map to the place; other targets open in a new tab.

---

# SIDC – C2 — Projekte (Missionsbau)

**Worum geht's?** Der Bereich **Missionsbau**: Projekte mit Dateien und Vorlagen, Verzeichnis, Zeitleiste, Plänen, Aufgaben-Board, Meilensteinen und Status, Feedback (auch von außen), Whiteboard, Aktivität und den Verknüpfungs-Codes, die alles verbinden. **Für wen?** Missionsbau und Projekt-Mitglieder. Wie eine Mission Schritt für Schritt entsteht: [Abläufe → Mission bauen](SIDC-C2-Workflows); Rechte: [Rollen & Sichtbarkeit](SIDC-C2-Roles-Visibility).

## 1. Projekte und Projekt-Ordner

- Sidebar **Missionsbau** öffnet die Projektliste (bzw. das zuletzt geöffnete Projekt).
- **Neues Projekt** (Admins und Missionsbauer): *Projektname*, *Beschreibung* → **Projekt anlegen**. Du wirst Owner.
- Projekte lassen sich in **Projekt-Ordner** sortieren (*Neuer Ordner*, Unterordner, Drag & Drop). Einen Ordner im Baum wählen, um das neue Projekt dort anzulegen. Ein gelöschter Ordner schiebt seinen Inhalt eine Ebene nach oben — gelöscht wird nichts.
- **Archivieren** blendet ein Projekt in der Standardliste aus (*Archivierte anzeigen*) und macht es schreibgeschützt; **Wiederherstellen** hebt das auf. **Projekt löschen** entfernt Dateien, Aufgaben und Notizen — die Pläne bleiben (ohne Projekt).
- Die Suche im Projektkopf (`/`) findet Titel und Inhalte von Dateien, Aufgaben, Notizen, Plänen und Bildern.

## 2. Projekt-Reiter

| Reiter | Inhalt |
|---|---|
| **Übersicht** | Projektstatus, Meilensteine, offene Aufgaben je Person, Fällig bald / überfällig, zuletzt geändert, neues Feedback |
| **Aktivität** | wer wie aktiv war (24 Stunden / 7 Tage / 30 Tage / 1 Jahr) und ein Aktivitätsfeed (Aufgaben, Dateien, Notizen, Mitglieder) |
| **Missionsbau** | Dateibaum + Markdown-Editor, Bilder — **nur Missionsbau** |
| **Verzeichnis** | Dateien nach Typ und **Auf der Karte** (alle Sterne/Notizen der Projekt-Pläne) |
| **Zeitleiste** | alle Überschriften mit Zeitangabe aus allen Dateien |
| **Pläne** | Pläne des Projekts (Ordner, Drag & Drop, *Bestehenden Plan hierher verschieben*, *Aus Projekt lösen*) |
| **Aufgaben** | Kanban mit Diskussion und Verlauf |
| **Feedback** | Rückmeldungen von Mitgliedern und Außenstehenden, *Zu Aufgabe machen* |
| **Whiteboard** | Haftnotizen mit Pfeilen |
| **Mitglieder** | Nutzer/Gruppen und ihre Rollen |

Ältere Projekte mit Dateien im früheren Bereich *Öffentlich* behalten diese; sie bleiben über Verweise, Verzeichnis und Zeitleiste erreichbar.

## 3. Dateien, Vorlagen und Verzeichnis

- Reiter **Missionsbau** → **Neues Dokument** (oder *Neuer Ordner*). Das Fenster *Neue Datei* bietet eine **Vorlage**:

  | Vorlage | Dokumenttyp → Gruppe im Verzeichnis |
  |---|---|
  | Leer | ohne Typ → *Sonstige* |
  | Missionskonzept | *Missionskonzepte* |
  | Befehl (5-Absatz) | *Befehle und Briefings* |
  | Briefing | *Befehle und Briefings* |
  | Charakter | *Personen* |
  | Fraktion | *Fraktionen* |
  | Ort | *Orte* |
  | Szene | *Szenen* |

- **Editor:** Ansichten *Schreiben* / *Geteilt* / *Vorschau*; `Strg+S` speichert; hat jemand anderes inzwischen gespeichert, kommt ein Konflikthinweis (*Meine Version speichern* / *Neueste Version laden*). Dateien lassen sich per Drag & Drop verschieben und umbenennen.
- **Datei-Werkzeuge:** *Kommentare* (Diskussion mit `@Erwähnungen`; Betrachter dürfen kommentieren) · *Verlauf* (beim Speichern wird der vorherige Stand gesichert, höchstens 50 pro Datei; *Ansehen*, *Wiederherstellen* — der aktuelle Stand wird vorher gesichert) · *Export* (*Als Markdown herunterladen*, *Drucken / PDF*).
- **Verzeichnis:** alle Dateien nach Typ gruppiert (*Missionskonzepte*, *Personen*, *Fraktionen*, *Orte*, *Szenen*, *Befehle und Briefings*, *Sonstige*), Suche in Titel **und** Inhalt, *Neu* je Gruppe. **Auf der Karte** listet alle Sterne und Notizen aller Pläne (pink/gelb) mit *Plan öffnen*.
- **Bilder** werden in einem Plan hochgeladen (Bild-Leiste) und erscheinen automatisch im Projekt; *Markdown* kopiert einen Einbettungs-Verweis.

## 4. Zeitleiste

Überschriften, die mit einer Zeitangabe beginnen, werden automatisch einsortiert (Ebenen 1–6; Code-Blöcke werden ignoriert). Klick auf einen Eintrag öffnet die Datei an der Überschrift.

| Schreibweise | Bedeutung |
|---|---|
| `## 06:30 Anmarsch` | Uhrzeit an Tag 1 (auch `06:30 Uhr`) |
| `## Tag 2 · 06:30 Angriff` / `## T2 06:30 …` / `Day 2 06:30` | Uhrzeit an Tag 2 |
| `## T+01:30 Feindkontakt` / `## H+90 …` / `H+1:30` / `T+90` | relativ zum Missionsbeginn (`H+` in Minuten) |

Absolute Einträge stehen nach (Tag, Uhrzeit) sortiert **vor** den relativen.

## 5. Aufgaben (Board)

- Kanban mit den Spalten **Offen · In Arbeit · Review · Erledigt**; Karten am Griff ⋮⋮ ziehen; Filter **Alle** / **Meine**.
- Felder: *Titel*, *Beschreibung (Markdown)*, *Status*, *Priorität* (Niedrig / Normal / Hoch), *Zuständig*, *Fällig am* (überfällig rot), *Bezug: Plan*, *Bezug: Dokument*.
- **Diskussion** in jeder Aufgabe (auch für Betrachter): `@name`, `@gruppe`, `@projekt` mit Autovervollständigung; `Strg+Enter` sendet. Ein **Verlauf** je Aufgabe zeigt, wer was geändert hat.
- Zuweisen und Erwähnen benachrichtigt die Person (siehe [Chat & Benachrichtigungen](SIDC-C2-Chat-Notifications)).
- Aufgaben entstehen auch aus einem Stern auf der Karte oder aus Feedback (*Zu Aufgabe machen*).

## 6. Projektstatus und Meilensteine

- **Projektstatus** (Übersicht): *Entwurf → Review → Test → Freigabe* — setzen dürfen Bearbeiter.
- **Meilensteine**: *Meilenstein hinzufügen* mit Titel und optionalem Termin, *Erledigt* ankreuzen; die Übersicht zeigt den Fortschritt („x von y erledigt"), hebt überfällige hervor und nennt den nächsten offenen Termin.

## 7. Feedback

- Reiter **Feedback** → **Neues Feedback**: *Art* (Fehler / Idee / Lob / Sonstiges), *Titel*, *Beschreibung*, optional Bezug zu Plan oder Datei. Jedes Mitglied (auch Betrachter) darf einsenden.
- Liste mit Suche und Filtern (*Status*: Neu / In Bearbeitung / Erledigt / Abgelehnt; *Art*); Quelle *intern* oder *extern*. Bearbeiter ändern den Status; **Zu Aufgabe machen** legt eine verknüpfte Aufgabe an und öffnet sie.
- **Externe Schnittstelle** (Owner): *Einschalten* erzeugt einen Link zu einem öffentlichen Formular `#/feedback/<token>` — wer den Link kennt, kann **ohne Login** Feedback senden und sieht nur den Projektnamen. *Link kopieren*, *curl-Beispiel kopieren*, *Neu erzeugen* (alter Link wird ungültig), *Abschalten*. Archivierte Projekte nehmen nichts an.
- Grenzen: Titel 200, Text 5000, Name 80 Zeichen; 10 Einsendungen je IP und 10 Minuten, 60 je Projekt und Stunde, höchstens 5000 je Projekt. Das Feld `website` ist eine Bot-Falle und muss leer bleiben. Die Sicherheitsrichtlinie der App verbietet das Einbetten per `<iframe>`; für ein eingebettetes Formular braucht `#/feedback/<token>` eine Ausnahme im Proxy.

```bash
curl https://c2.example.org/api/public/feedback/<TOKEN>
# -> {"project":"Op Beispiel","kinds":["bug","idea","praise","other"]}
curl -X POST https://c2.example.org/api/public/feedback/<TOKEN> -H "Content-Type: application/json" \
  -d '{"title":"Brücke bei 040/040 fehlt","text":"Im Briefing erwähnt, auf der Karte nicht.","kind":"bug","name":"Max"}'
# -> 201 {"ok":true}   (404 unbekanntes Token, 410 Projekt archiviert, 429 zu viele Anfragen)
```

- Owner können eine Benachrichtigung bei neuem Feedback zuschalten (siehe [Chat & Benachrichtigungen](SIDC-C2-Chat-Notifications)).

## 8. Whiteboard

- **Haftnotiz** in der gewählten Farbe (Gelb, Grün, Blau, Rosa, Orange, Grau); direkt tippen, am Kopf ziehen zum Verschieben, Größe ändern, *Farbe wechseln*, löschen (max. 4000 Zeichen).
- Punkt am Notizrand auf eine andere Notiz ziehen = **Pfeil**; Notizen können *Aufgabe verlinken* oder *Nutzer verlinken*.
- Ansicht verschieben durch Ziehen der freien Fläche oder mit den Pfeil-Knöpfen; Zoom, *Alles* (alle Notizen einpassen), auf 100 % zurücksetzen.
- Sterne auf der Karte können direkt Haftnotizen anlegen (siehe [Abläufe](SIDC-C2-Workflows)).

## 9. Verknüpfungs-Codes

Überall in Markdown (Dateien, Aufgaben, Notizen, Phasen-Notizen, Textfelder) lassen sich einsetzen:

| Code | Ziel |
|---|---|
| `{{doc:<id>}}` | Datei |
| `{{task:<id>}}` | Aufgabe |
| `{{plan:<id>}}` | Plan |
| `{{note:<id>}}` | Haftnotiz |
| `{{img:<id>}}` | Bild |
| `{{place:<karte>:<ortsname>}}` | Ort einer Karte |

- Einfügen über den Leisten-Knopf *Verweis einfügen* (Suchfenster, Karten → Orte) oder *Code kopieren* am Ziel.
- Sie erscheinen als Chip mit dem **aktuellen** Namen; verschwundene oder für dich unsichtbare Ziele als grauer Chip *nicht verfügbar*.
- Jedes Ziel zeigt **Erwähnt in …** (Rückverweise), auch Sterne auf Karten. In Plänen fliegt ein Ort-Chip derselben Karte zum Ort; andere Ziele öffnen in einem neuen Tab.
