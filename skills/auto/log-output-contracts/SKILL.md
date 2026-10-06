---
name: log-output-contracts
description: Use when parsing logs into structured error reports with a required output schema.
---
- Read the output contract before parsing; preserve required top-level fields and metadata values.
- Normalize service names to lowercase and replace hyphens with underscores.
- Convert timestamps to the required UTC representation.
- Sort output entries by service, then by UTC timestamp in ascending order.
- Preserve required per-entry fields and handle multiline details and repeat counts consistently.
- Validate the final JSON structure, field types, normalization, and ordering.
