# Design Document – Cline Conversation History Discovery

## Overview & Goals
- Investigate the Cursor extension storage path `~/Library/Application Support/Cursor/User/globalStorage/saoudrizwan.claude-dev/`.
- Identify whether conversation history or related session artifacts exist, and summarize their structure without exposing sensitive payloads.
- Produce a repeatable inspection approach that respects privacy and project security rules.

## Architecture / Workflow
```mermaid
graph TD
    A[Target Directory] --> B[List Top-Level Entries]
    B --> C[Recursive File Survey]
    C --> D[Classify Candidates
(JSON, sqlite, logs)]
    D --> E[Sample Inspections
(head, jq, strings)]
    E --> F[Capture Findings
(non-sensitive summaries)]
    F --> G[Report Summary & Next Steps]
```

## Tech Stack & Key Decisions
- **Shell Utilities**: `ls`, `find`, `file`, `stat` for structure and metadata.
- **Text Inspection**: Prefer `jq`, `head`, `rg` for JSON/log review; fallback to `strings` for binaries.
- **Privacy Preservation**: Do not echo full conversation contents; redact or summarize if personal data detected.
- **Snapshot Orientation**: Treat analysis as read-only; avoid modifying any file timestamps or contents.

## Data Models / Files of Interest
- Expect data formats like:
  - JSON cache files (potential conversation payloads, metadata).
  - SQLite databases or LevelDB stores for session data.
  - Log archives containing request/response traces.
- Key directories:
  - `state/` – likely persistent extension state (primary candidate).
  - `tasks/` – may hold scratch or queued operations.
  - `checkpoints/` – potential autosave snapshots.

## Non-Functional Requirements
- **Security**: Follow `copilot-rules.md`; avoid leaking secrets or PII.
- **Repeatability**: Commands documented so user can rerun.
- **Performance**: Use targeted recursion to avoid unnecessary large reads; skip huge binaries unless needed.
- **Auditability**: Record inspected paths and inspection commands for transparency.

*Prepared: September 24, 2025*
