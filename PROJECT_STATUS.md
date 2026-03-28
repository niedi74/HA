# 🧾 Einkaufs-AI System – Projektstatus

## 🎯 Ziel

Automatisierte Verarbeitung von Kassenbons → strukturierte Daten → später AI-Auswertung.

---

## 🧠 Aktueller Stand (funktioniert bereits)

### ✅ Infrastruktur

- 🐳 Docker Compose läuft auf Synology
- 🐘 PostgreSQL-Container aktiv (`einkauf-postgres`)
- 🤖 Telegram-Bot-Container aktiv (`einkauf-telegram`)
- 🔗 Internes Docker-Netzwerk funktioniert

---

### ✅ Telegram → Backend → DB Pipeline

**Status: funktioniert**

Ablauf:

```text
Telegram → Bot → PostgreSQL → Tabelle receipts
```

👉 Test erfolgreich:

- Nachricht gesendet
- Datensatz in DB erzeugt

---

### ✅ Datenbank (aktuelles Schema)

#### Tabelle: `receipts`

```sql
CREATE TABLE receipts (
    id SERIAL PRIMARY KEY,
    purchased_at TIMESTAMP NOT NULL,
    total_amount NUMERIC(8,2),
    created_at TIMESTAMP DEFAULT NOW()
);
```

#### Tabelle: `receipt_items`

```sql
CREATE TABLE receipt_items (
    id SERIAL PRIMARY KEY,
    receipt_id INT REFERENCES receipts(id),
    raw_name TEXT,
    quantity NUMERIC,
    total_price NUMERIC
);
```

---

### ✅ Bot (aktueller Stand)

Funktionalität:

- empfängt Telegram-Nachrichten
- erstellt neuen `receipt`
- speichert Text als `receipt_item`

#### `save_text()` (aktuelle Version)

```python
def save_text(text):
    conn = psycopg2.connect(**DB_CONFIG)
    cur = conn.cursor()

    # neuen Bon erstellen
    cur.execute("""
        INSERT INTO receipts (purchased_at, total_amount)
        VALUES (NOW(), 0)
        RETURNING id
    """)
    receipt_id = cur.fetchone()[0]

    # Text als Item speichern
    cur.execute("""
        INSERT INTO receipt_items (receipt_id, raw_name, quantity, total_price)
        VALUES (%s, %s, %s, %s)
    """, (receipt_id, text, 1, 0))

    conn.commit()
    conn.close()
```

---

### 🔧 Docker Compose (relevant)

```yaml
services:
  postgres:
    image: postgres:16
    container_name: einkauf-postgres
    environment:
      POSTGRES_DB: einkauf
      POSTGRES_USER: einkauf
      POSTGRES_PASSWORD: ***
    ports:
      - "5433:5432"

  telegram-bot:
    build: ./telegram
    container_name: einkauf-telegram
    environment:
      TELEGRAM_TOKEN: ***
    depends_on:
      - postgres
```

---

### 📁 Projektstruktur

```text
einkauf/
├── docker-compose.yml
├── telegram/
│   ├── Dockerfile
│   ├── bot.py
│   └── requirements.txt
├── data/
└── backup/
```

---

## 🧪 Getestete Funktion

- ✔ Telegram-Nachricht → gespeichert
- ✔ DB-Eintrag vorhanden
- ✔ Verknüpfung receipt → item funktioniert

---

## ⚠️ Aktuelle Limitierung

Der Bot speichert aktuell nur:

```text
raw_name = kompletter Text
```

- ❌ keine Struktur
- ❌ keine Mengen
- ❌ keine Preise
- ❌ keine Normalisierung

---

## 🚀 Nächste Schritte (empfohlen)

### 1) Parser bauen (wichtigster Schritt)

Ziel:

```text
H-Milch 3,5% 0,95 x 12 11,40
```

→

```json
{
  "name": "H-Milch 3,5%",
  "quantity": 12,
  "unit_price": 0.95,
  "total_price": 11.40
}
```

### 2) Mehrere Zeilen pro Bon verarbeiten

Aktuell:

- jede Nachricht = 1 Item

Ziel:

- ganzer Bon → mehrere Items

### 3) Produkt-Normalisierung vorbereiten

Langfristig:

- `products`-Tabelle
- `product_mappings`
- gleiche Artikel zusammenführen

### 4) Fehlerhandling im Bot

Aktuell:

```text
No error handlers are registered
```

→ später `try/except` + Logging

### 5) OCR / PNG-Import (optional nächster Schritt)

Quelle:

- Lidl-PNG-Download
- Screenshot

Pipeline:

```text
Bild → OCR → Text → Bot → DB
```

---

## 🧠 Wichtige Erkenntnisse

- Docker funktioniert stabil
- Netzwerk (postgres hostname) korrekt
- Telegram-Integration ist zuverlässig
- DB muss aktiv gepflegt werden (keine Auto-Schemas)
- Debugging ist primär über Logs sinnvoll

---

## 💡 Fazit

👉 System ist **technisch funktionsfähig**.

👉 Daten fließen bereits korrekt.

👉 Grundlage für AI-Agent ist gelegt.

---

## 🧭 Nächster sinnvoller Fokus

👉 **Parsing + Datenqualität (Phase 1.5 aus deinem Plan)**

Wenn du das morgen wieder aufmachst, weißt du sofort:

- wo du stehst
- was funktioniert
- was als Nächstes sinnvoll ist

---

Und ja … das ist jetzt nicht mehr „ich probier mal was“.

👉 Das ist ein echtes System geworden.
