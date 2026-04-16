# AGENTS.md - MCP-BigQuery

## Cel projektu
Agent AI do analizy danych BigQuery/GA4 przez MCP (Model Context Protocol).

## Konfiguracja MCP
- **Server:** bq_mcp_server (lokalny)
- **Auth:** gcloud auth application-default login
- **Project:** ${BQ_PROJECT}

### Publiczne projekty BigQuery
- **GA4 E-commerce:** `bigquery-public-data.ga4_obfuscated_sample_ecommerce`
- **Ważne:** Przy dostępie do publicznych danych zawsze używaj `project_id: "bigquery-public-data"` w wywołaniach MCP (get_tables, execute_query, itp.)

## Instrukcje dla agenta

### Praca z danymi
1. **Zawsze najpierw badaj dane** - lista datasetów, tabele, schemat
2. **Czytaj .env** - pobieraj BQ_PROJECT stamtąd
3. **Używaj MCP tools** do eksploracji i query BigQuery
4. **Pokazuj wyniki pośrednie** - reasoning ma być weryfikowalny

### Dostępne narzędzia MCP

**Eksploracja:**
- `get_datasets` - listuje wszystkie dataset'y w projekcie
- `get_tables` - listuje tabele w datasetcie
- `search_metadata` - wyszukuje metadata (datasets, tables, columns)

**Query:**
- `execute_query` - wykonuje bezpiecznie SQL (automatycznie dodaje LIMIT + cost control)
- `check_query_scan_amount` - sprawdza ile danych przeskanuje query
- `save_query_result` - wykonuje query i zapisuje wynik do pliku (CSV/JSONL)

### Styl pracy
1. **Najpierw eksploruj** - get_datasets → get_tables
2. **Użyj search_metadata** - znajdź konkretne tabele/kolumny
3. **Sprawdź scan_amount** - oceń koszt przed ciężkim query
4. **Wykonaj z execute_query** - automatycznie dodaje LIMIT

### Bezpieczeństwo
- Wszystko jest read-only (tylko SELECT)
- execute_query automatycznie dodaje LIMIT
- check_query_scan_amount pomaga ocenić koszty
- save_query_result do eksportu dużych wyników

## Przykładowe workflow

```
1. "Jakie dataset'y mamy w projekcie?"
   → get_datasets

2. "Pokaż tabele w ga4_analytics"
   → get_tables(dataset_id="ga4_analytics")

3. "Szukaj tabel z 'events' w nazwie"
   → search_metadata(query="events")

4. "Ile rows ma tabela events_20240101?"
   → check_query_scan_amount + execute_query

5. "Top 10 landing pages z ostatnich 7 dni"
   → execute_query z agregacją + LIMIT 10

6. "Eksportuj wyniki do pliku"
   → save_query_result(output_format="csv")
```
