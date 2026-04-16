# MCP-BigQuery Troubleshooting

## Problem: MCP server się nie uruchamia

### Błąd: `command not found: bq_mcp_server`

**Rozwiązanie:**
```bash
pip install bq_mcp_server
# lub z lokalnym Python
python -m bq_mcp_server
```

### Błąd: `No credentials found`

**Rozwiązanie:**
```bash
gcloud auth application-default login
```

Sprawdź czy credentials istnieją:
```bash
ls ~/.config/gcloud/application_default_credentials.json
```

### Błąd: `bq_mcp_server not in PATH`

W `opencode.json` użyj pełnej ścieżki:
```bash
which python
# wynik: /usr/bin/python3

# w opencode.json:
"command": ["/usr/bin/python3", "-m", "bq_mcp_server"]
```

---

## Problem: Environment variables nie działają

### Błąd: `BQ_PROJECT is not set`

OpenCode nie ładuje automatycznie `.env`. Rozwiązania:

**Opcja 1: Uruchom z exportem**
```bash
export BQ_PROJECT=twoj-project-id
export BQ_LOCATION=EU
opencode
```

**Opcja 2: Dodaj do shell profile**
```bash
echo 'export BQ_PROJECT=twoj-project-id' >> ~/.zshrc
echo 'export BQ_LOCATION=EU' >> ~/.zshrc
source ~/.zshrc
opencode
```

---

## Problem: Autentykacja GCP

### Błąd: `Permission denied` lub `ACCESS_DENIED`

Sprawdź uprawnienia:
```bash
gcloud projects get-iam-policy YOUR_PROJECT_ID \
  --flatten="bindings[].members" \
  --filter="bindings.role:roles/bigquery.user"
```

Dodaj uprawnienia:
```bash
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
  --member="user:twoj-email@gmail.com" \
  --role="roles/bigquery.user"
```

### Sprawdź aktualny projekt
```bash
gcloud config get-value project
```

---

## Problem: BigQuery query nie działa

### Błąd: `Table not found`

Upewnij się że dataset istnieje:
```bash
bq ls
```

### Błąd: Query timeout

Zmniejsz LIMIT lub użyj partition filtering:
```sql
SELECT * FROM `project.dataset.table_20240101`
WHERE _PARTITIONTIME = TIMESTAMP('2024-01-15')
LIMIT 1000
```

---

## Problem: OpenCode nie widzi MCP

### Sprawdź konfigurację
```bash
opencode --info  # lub opencode -v
```

### Restart OpenCode
Czasem pomaga pełny restart:
```bash
exit  # i uruchom ponownie opencode
```

---

## Diagnostyka

```bash
# Test MCP server lokalnie
python -m bq_mcp_server --help

# Test połączenia BigQuery
bq query --use_legacy_sql=false "SELECT 1"

# Sprawdź uprawnienia
gcloud auth list
```

---

## Kontakt

W razie problemów - zgłoś issue na GitHub.
