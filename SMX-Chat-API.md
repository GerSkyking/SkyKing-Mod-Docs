# SMX Chat API (for other mods)

> Source: `SkyMap-X-DEV` → `Scripts/Game/Helper/SMX_ChatAPI.c`. Requires SkyMap-X as a dependency of your addon.

## 1. What it is

A small read-only API so other mods can read the SkyMap-X tablet chat: all messages of the current session, with sender and text.

Every message is an `SMX_ChatData`:

| Field | Content |
|---|---|
| `sender` | Chat ID of the sender tablet: `Device ID [Info]`, e.g. `Alpha 1-1 [SkyKing]` |
| `senderName` | Name the sender used (player / device name) |
| `receiver` | Chat ID of the receiving tablet |
| `message` | Message text |
| `timestamp` | `HH:MM:SS` |

> **Chat ID:** a conversation is exactly between two tablets. The ID is `Device ID [Info]` (same text as in the Chat contact list). The plain Device ID is the group name by default and is shared by all group members, so it is not used alone.

## 2. Calls

| Call | Where | Returns |
|---|---|---|
| `SMX_ChatAPI.GetSessionMessages()` | Client | `array<ref SMX_ChatData>`: all messages of this session that this client sent or received (history loaded from files is **not** included). |
| `SMX_ChatAPI.GetOnMessage()` | Client | `ScriptInvoker`, fires on every new message of this client. Handler: `void Fn(SMX_ChatData data)` |
| `SMX_ChatAPI.GetOnServerMessage()` | Server / host | `ScriptInvoker`, fires for **all** chat messages of **all** players. Handler: `void Fn(SMX_ChatData data)` |

A client only knows conversations it takes part in. For truly all messages of a session use the server hook. It fires after rate limit and length trimming (500 chars) were applied. Calls return `null` while the managers do not exist yet.

## 3. Example

```c
// Server-side mod: log every chat message
SMX_ChatAPI.GetOnServerMessage().Insert(OnChat);

void OnChat(SMX_ChatData d)
{
	Print(d.timestamp + " " + d.senderName + " -> " + d.receiver + ": " + d.message);
}
```

---

# SMX Chat-API (für andere Mods)

> Quelle: `SkyMap-X-DEV` → `Scripts/Game/Helper/SMX_ChatAPI.c`. SkyMap-X muss als Abhängigkeit deines Addons eingetragen sein.

## 1. Kurzbeschreibung

Kleine Lese-API, mit der andere Mods den Chat des SkyMap-X-Tablets auslesen können: alle Nachrichten der aktuellen Session mit Absender und Text.

Jede Nachricht ist ein `SMX_ChatData`:

| Feld | Inhalt |
|---|---|
| `sender` | Chat-Kennung des sendenden Tablets: `Geräte-ID [Info]`, z.B. `Alpha 1-1 [SkyKing]` |
| `senderName` | Name, den der Absender verwendet hat (Spieler-/Gerätename) |
| `receiver` | Chat-Kennung des empfangenden Tablets |
| `message` | Nachrichtentext |
| `timestamp` | `HH:MM:SS` |

> **Chat-Kennung:** Ein Gespräch läuft immer zwischen genau zwei Tablets. Die Kennung ist `Geräte-ID [Info]` (derselbe Text wie in der Chat-Kontaktliste). Die reine Geräte-ID ist standardmäßig der Gruppenname und für alle Gruppenmitglieder gleich, deshalb wird sie nicht allein verwendet.

## 2. Aufrufe

| Aufruf | Wo | Liefert |
|---|---|---|
| `SMX_ChatAPI.GetSessionMessages()` | Client | `array<ref SMX_ChatData>`: alle Nachrichten dieser Session, die dieser Client gesendet oder empfangen hat (aus Dateien geladener Verlauf ist **nicht** enthalten). |
| `SMX_ChatAPI.GetOnMessage()` | Client | `ScriptInvoker`, feuert bei jeder neuen Nachricht dieses Clients. Handler: `void Fn(SMX_ChatData data)` |
| `SMX_ChatAPI.GetOnServerMessage()` | Server / Host | `ScriptInvoker`, feuert für **alle** Chat-Nachrichten **aller** Spieler. Handler: `void Fn(SMX_ChatData data)` |

Ein Client kennt nur Gespräche, an denen er beteiligt ist. Für wirklich alle Nachrichten einer Session den Server-Hook nutzen. Er feuert, nachdem Rate-Limit und Längenkürzung (500 Zeichen) angewendet wurden. Die Aufrufe liefern `null`, solange die Manager noch nicht existieren.

## 3. Beispiel

```c
// Server-Mod: jede Chat-Nachricht loggen
SMX_ChatAPI.GetOnServerMessage().Insert(OnChat);

void OnChat(SMX_ChatData d)
{
	Print(d.timestamp + " " + d.senderName + " -> " + d.receiver + ": " + d.message);
}
```
