---
name: tabular-data-normalization
description: Use when cleaning tabular input and producing structured JSON or CSV outputs.
---
- Inspect the input schema and formats before transforming data.
- Count input data rows before deduplication; include duplicates in that count.
- Deduplicate using the task’s specified entity key, then exclude records with unknown amounts where required.
- Convert money to integer cents before writing outputs; avoid floating-point arithmetic for currency.
- Normalize timestamps to UTC in the required format and categories to canonical spellings.
- Make JSON metadata reflect the source, input-row count, and distinct usable-record count.
- Write the required CSV header and one row per qualifying distinct entity.
- Validate output schemas, row counts, and representative transformed values before finishing.
