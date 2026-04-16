# MCP-Vibe

Projekt do analizy danych GA4/BigQuery z wykorzystaniem OpenCode i MCP (Model Context Protocol). Agent AI może eksplorować schematy tabel, wyszukiwać metadata, walidować SQL i wykonywać zapytania bezpośrednio z poziomu terminala.

---

## Publiczne projekty BigQuery
- **GA4 E-commerce:** `bigquery-public-data.ga4_obfuscated_sample_ecommerce`
- **Ważne:** Przy dostępie do publicznych danych zawsze używaj `project_id: "bigquery-public-data"` w wywołaniach MCP (get_tables, execute_query, itp.)

## Troubleshooting

Zobacz [TROUBLESHOOTING.md](./TROUBLESHOOTING.md) - rozwiązania typowych problemów.

---

## Struktura projektu

```
MCP-Vibe/
├── .env                     # Zmienne środowiskowe (nie commituj - zawiera BQ_PROJECT!)
├── .gitignore               # Wykluczenia z git
├── AGENTS.md                # Instrukcje dla agenta AI
├── opencode.json            # Konfiguracja OpenCode + MCP BigQuery (包含 command array)
├── requirements.txt         # Zależności Python
├── README.md               # Ten plik
└── TROUBLESHOOTING.md     # Rozwiązania problemów
```

### Plik .env (zmienne środowiskowe)

```bash
BQ_PROJECT=analytics-492705    # ID projektu GCP
BQ_LOCATION=EU              # Lokalizacja BigQuery (EU, US, etc.)
```

**Ważne:** Plik `.env` NIE jest śledzony w git (jest w .gitignore).

---

## Wymagania

1. Python 3.11+
2. pip
3. Google Cloud SDK
4. OpenCode CLI
5. Konto Google Cloud z BigQuery API

## Setup

### 1. Zainstaluj Google Cloud SDK
```bash
# macOS
brew install google-cloud-sdk

# Lub oficjalny installer
curl https://sdk.cloud.google.com | bash
source ~/.bashrc  # lub ~/.zshrc
```

### 2. Zainstaluj zależności
```bash
pip install -r requirements.txt
```

### 3. Autentykacja
```bash
gcloud auth login
gcloud auth application-default login
```

### 4. Skonfiguruj projekt GCP
```bash
gcloud config set project YOUR_PROJECT_ID
```

### 5. Skonfiguruj środowisko

**Ważne:** OpenCode nie ładuje automatycznie `.env`. Dodaj export do shell:

```bash
# Dodaj do ~/.zshrc (lub ~/.bashrc)
echo 'export BQ_PROJECT=twoj-project-id' >> ~/.zshrc
echo 'export BQ_LOCATION=EU' >> ~/.zshrc
source ~/.zshrc
```

Lub uruchamiaj z exportem:
```bash
export BQ_PROJECT=twoj-project-id
export BQ_LOCATION=EU
opencode
```

### 6. Uruchom OpenCode
```bash
cd /Users/p/Documents/dev/MCP-Vibe
opencode
```

## Konfiguracja MCP BigQuery (opencode.json)

Konfiguracja `command` w sekcji `mcp.bigquery`:

```json
"command": ["bq_mcp_server", "--project-ids", "analytics-492705,bigquery-public-data", "--query-execution-project-id", "analytics-492705"]
```

| Indeks | Element | Wartość przykładowa | Opis |
|-------|--------|-------------------|------|
| 0 | executable | `bq_mcp_server` | Nazwa narzędzia MCP server |
| 1 | flaga | `--project-ids` | Flaga określająca projekt(y) GCP |
| 2 | argument | `analytics-492705,bigquery-public-data` | Projekty oddzielone przecinkiem |
| 3 | flaga | `--query-execution-project-id` | Flaga dla projektu wykonującego query |
| 4 | argument | `analytics-492705` | Projekt do bilingu query |

**Podział na flagi i argumenty:**

```
bq_mcp_server --project-ids analytics-492705,bigquery-public-data --query-execution-project-id analytics-492705
```

| Flaga | Argument | Znaczenie |
|------|----------|-----------|
| `--project-ids` | `analytics-492705,bigquery-public-data` | Które projekty są widoczne dla MCP |
| `--query-execution-project-id` | `analytics-492705` | Który projekt ponosi koszty query |

**Dlaczego tak?**
- `analytics-492705` - Twój projekt z danymi GA4 i aktywnym billingiem
- `bigquery-public-data` - Publiczne dane Google (np. sample GA4 do nauki/testów)

## Dostępne narzędzia MCP BigQuery

**Eksploracja:**
- `get_datasets` - listuje wszystkie dataset'y w projekcie
- `get_tables` - listuje tabele w datasetcie
- `search_metadata` - wyszukuje metadata (datasets, tables, columns)

**Query:**
- `execute_query` - wykonuje bezpiecznie SQL (automatycznie dodaje LIMIT + cost control)
- `check_query_scan_amount` - sprawdza ile danych przeskanuje query
- `save_query_result` - wykonuje query i zapisuje wynik do pliku (CSV/JSONL)

## Przykładowe zapytania do agenta

- "Jakie dataset'y mamy w projekcie?"
- "Pokaż tabele w ga4_analytics"
- "Szukaj tabel z 'events' w nazwie"
- "Ile sesji mieliśmy wczoraj?" (z kosztorysem)
- "Top 10 landing pages z ostatnich 7 dni"
