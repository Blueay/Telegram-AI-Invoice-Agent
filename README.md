
# 🤖 Telegram AI Agent – n8n Invoice Processor

Ein n8n-Workflow, der einen Telegram-Bot als KI-Agenten betreibt. Der Bot kann:

- **Dokumente (PDFs) verarbeiten** → Rechnungsdaten automatisch extrahieren, in Google Drive speichern und in Notion protokollieren
- **Textnachrichten beantworten** → via GPT als intelligenter Chatbot

---

## 📋 Inhaltsverzeichnis

- [Was macht dieser Workflow?](#-was-macht-dieser-workflow)
- [Architektur & Ablauf](#-architektur--ablauf)
- [Voraussetzungen](#-voraussetzungen)
- [Setup-Anleitung](#-setup-anleitung)
- [Workflow importieren](#-workflow-importieren)
- [Konfiguration anpassen](#-konfiguration-anpassen)
- [Nutzung](#-nutzung)

---

## 🔍 Was macht dieser Workflow?

### Fall 1 – PDF-Rechnung wird gesendet
Wenn ein Nutzer eine **PDF-Datei** an den Telegram-Bot sendet:

1. Die PDF wird automatisch aus Telegram heruntergeladen
2. Der Text wird aus der PDF extrahiert
3. Ein KI-Modell (GPT-4.1-mini) liest die folgenden Rechnungsfelder:
   - Rechnungsnummer
   - Gesamtbetrag
   - Währung
   - Datum
   - Name / Beschreibung der Rechnung
4. Die PDF-Datei wird in **Google Drive** hochgeladen und öffentlich freigegeben
5. Ein neuer Eintrag wird in einer **Notion-Datenbank** angelegt mit allen extrahierten Feldern + Link zur Datei in Drive

### Fall 2 – Textnachricht wird gesendet
Wenn ein Nutzer eine **Textnachricht** sendet:

1. Die Nachricht wird an **GPT-5** weitergeleitet
2. Die Antwort des Modells wird direkt als Telegram-Nachricht zurückgeschickt

---

## 🏗 Architektur & Ablauf

```
Telegram Trigger
       │
       ▼
┌─────────────────────────────────┐
│ Hat die Nachricht ein Dokument? │
└─────────────────────────────────┘
       │                    │
      JA                   NEIN
       │                    │
       ▼                    ▼
┌──────────────────┐  ┌──────────────────┐
│ Extract from PDF │  │ GPT-5 (Chat)     │
│ + Upload to      │  └──────────────────┘
│   Google Drive   │         │
└──────────────────┘         ▼
       │              ┌──────────────────┐
       ▼              │ Telegram Antwort │
┌──────────────────┐  └──────────────────┘
│ Information      │
│ Extractor        │
│ (GPT-4.1-mini)   │
└──────────────────┘
       │
       ▼
┌──────────────┐
│    Merge     │ ◄── Google Drive Share Link
└──────────────┘
       │
       ▼
┌────────────────────────┐
│ Notion Datenbank       │
│ Eintrag erstellen      │
└────────────────────────┘
```

---

## ✅ Voraussetzungen

Bevor du startest, benötigst du Accounts und API-Zugänge für:

| Dienst | Zweck |
|---|---|
| [n8n](https://n8n.io) | Workflow-Automatisierung (Self-hosted oder Cloud) |
| [Telegram Bot API](https://core.telegram.org/bots) | Bot erstellen via @BotFather |
| [OpenAI API](https://platform.openai.com) | GPT-Modelle für Chat & Extraktion |
| [Google Drive API](https://console.cloud.google.com) | PDF-Dateien speichern |
| [Notion API](https://www.notion.so/my-integrations) | Rechnungsdaten protokollieren |

---

## 🚀 Setup-Anleitung

### 1. Telegram Bot erstellen

1. Öffne Telegram und suche nach **@BotFather**
2. Sende `/newbot` und folge den Anweisungen
3. Kopiere den **Bot Token** – du brauchst ihn gleich

### 2. Notion Datenbank vorbereiten

Erstelle eine neue Notion-Datenbank mit folgenden Feldern:

| Feldname | Typ |
|---|---|
| `Name` | Title (Standard) |
| `Invoice Number` | Title |
| `Total Amount` | Number |
| `Currency` | Select |
| `Date` | Date |
| `Status` | Status |
| `Invoice File` | URL |

Erstelle dann eine **Notion Integration** unter [notion.so/my-integrations](https://www.notion.so/my-integrations) und teile die Datenbank mit der Integration (über „Connect to" in den Datenbankeinstellungen).

### 3. Google Drive vorbereiten

- Stelle sicher, dass dein Google-Konto aktiv ist
- Erstelle optional einen dedizierten Ordner für Rechnungen
- Notiere die **Ordner-ID** aus der URL: `https://drive.google.com/drive/folders/DEINE_ORDNER_ID`

### 4. n8n Credentials einrichten

Füge in n8n unter **Settings → Credentials** folgende Zugänge hinzu:

**Telegram:**
- Typ: `Telegram API`
- API Token: *(dein Bot Token von @BotFather)*

**OpenAI:**
- Typ: `OpenAI API`
- API Key: *(dein OpenAI API Key aus platform.openai.com)*

**Google Drive:**
- Typ: `Google Drive OAuth2 API`
- Verbindung über OAuth2 mit deinem Google-Konto herstellen

**Notion:**
- Typ: `Notion API`
- API Key: *(dein Notion Integration Secret Token)*

---

## 📥 Workflow importieren

1. Lade die Datei `Telegram_Agent_AI.json` aus diesem Repository herunter
2. Öffne dein n8n Dashboard
3. Klicke auf **„Import from File"** (oder `Strg+O`)
4. Wähle die JSON-Datei aus
5. Ersetze in allen Nodes die Credential-Verweise mit deinen eigenen (rot markierte Nodes)
6. Passe die **Notion Datenbank-ID** im Node `Create a database page` an
7. Aktiviere den Workflow mit dem Toggle oben rechts ✅

---

## 🛠 Konfiguration anpassen

### Zielordner in Google Drive ändern
Im Node **„Upload file"** → `folderId` auf die ID deines gewünschten Ordners setzen.

### Notion Datenbank wechseln
Im Node **„Create a database page"** → `databaseId` mit der ID deiner Notion-Datenbank ersetzen.  
Die ID findest du in der URL deiner Datenbank: `https://www.notion.so/DEINE_DB_ID`

### Weitere Extraktionsfelder hinzufügen
Im Node **„Information Extractor"** kannst du zusätzliche Felder definieren, z. B.:
- `vendor` – Name des Lieferanten
- `vat` – Mehrwertsteuerbetrag
- `due_date` – Zahlungsziel

### GPT-Modell wechseln
- Für den Chatbot: Node **„Message a model"** → `modelId` ändern
- Für die Extraktion: Node **„OpenAI Chat Model"** → `model` ändern (z. B. auf `gpt-4o` für bessere Genauigkeit)

---

## 💬 Nutzung

1. Starte eine Unterhaltung mit deinem Telegram-Bot
2. **Textnachricht senden** → Der Bot antwortet direkt per GPT
3. **PDF-Rechnung senden** → Der Bot verarbeitet sie vollautomatisch:
   - Die Datei wird in Google Drive gespeichert
   - Alle Rechnungsdaten werden per KI extrahiert
   - Ein neuer Eintrag erscheint in deiner Notion-Datenbank
   - Der Status wird automatisch auf `Pending` gesetzt

---

## 🔒 Sicherheitshinweise

- Speichere API-Keys **niemals** direkt im Workflow-JSON – nutze ausschließlich n8n's Credential-Manager
- Beschränke den Zugriff auf den Bot ggf. per `chat.id`-Prüfung (nur bestimmte Nutzer erlauben)
- Setze Google Drive-Freigaben nur so offen wie nötig
- Rotiere API-Keys regelmäßig

---

## 📄 Lizenz

MIT License – frei verwendbar und anpassbar.
