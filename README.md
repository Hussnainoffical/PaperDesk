# PaperDesk

PaperDesk is an evidence-first research assistant for your own PDF library. It retrieves page-level passages, shows their source before you trust an answer, and abstains when search evidence is weak.

## What it does

- Imports PDFs into a local SQLite database with one-based page references.
- Chunks by page and sentence boundaries; searches with SQLite FTS5 BM25 plus lexical coverage.
- Displays retrieved passages, source page, title, a relative ranking score, and matched query terms.
- Can generate a cited synthesis through an optional OpenAI-compatible endpoint such as a local Ollama server.
- Checks that a generated answer contains valid evidence IDs. If no model endpoint is configured, it stays in retrieval-only mode and does not send document text anywhere.
- Can import the adjacent `../papers/` folder in this workspace, which contains the 50-paper 2026 reading set. This convenience button is hidden when that folder is absent.

## Run locally on Windows

From the repository root in PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
Copy-Item .env.example .env
uvicorn app:app --reload
```

Open http://127.0.0.1:8000. Add PDFs through the library panel. If the sibling `papers` folder exists, choose **Import 2026 paper set** to ingest those PDFs. The parsed text is stored in `paperdesk/data/paperdesk.sqlite3` and excluded from Git.

The current local checkout already has an ignored SQLite database seeded from the extracted text of all 50 papers. A fresh clone starts without that database or the paper PDFs; upload your PDFs, or set `PAPERDESK_SEED_DIR` to a local folder that contains them.

## Optional local answer model

PaperDesk uses local retrieval without a model. To enable synthesized answers through Ollama's OpenAI-compatible API, configure `.env`:

```dotenv
PAPERDESK_LLM_BASE_URL=http://localhost:11434/v1
PAPERDESK_LLM_MODEL=your-installed-model
PAPERDESK_LLM_API_KEY=
```

You can point `PAPERDESK_LLM_BASE_URL` at another OpenAI-compatible service. Only the question and selected evidence passages are sent to that endpoint. If it is remote, those excerpts leave the machine. Keep API keys out of source control. No endpoint or key is required for local retrieval.

## API

- `GET /api/health` — local library counts and model configuration state
- `GET /api/library` — imported papers
- `POST /api/ingest` — upload one PDF as multipart form field `file`
- `POST /api/import-local` — asynchronously ingest PDFs from `PAPERDESK_SEED_DIR` (defaults to `../papers`)
- `GET /api/import-status/{job_id}` — local import progress
- `POST /api/ask` — JSON `{ "question": "...", "top_k": 5 }`

## Current limits

- Scanned PDFs without an embedded text layer need OCR before import; OCR is not included yet.
- It supports uploaded PDFs up to 25 MB and 500 pages.
- Search is lexical BM25 plus query-term coverage. It is a transparent baseline, not a semantic vector index; evaluate it on a fixed question set before claiming quality.
- Citation-ID presence is checked automatically, but citation entailment is not yet verified. Always inspect the displayed evidence.
- Relative ranking values and query-term coverage are diagnostic measurements, not confidence probabilities.
- The app is designed for one trusted local user. It has no authentication or multi-user access controls; do not expose it directly to the public internet.

## Design notes

[Research design trail](RESEARCH_DESIGN.md) links the MVP decisions to specific papers. `../research/papers_2026.json` and `../research/reading_pack.md` keep the source links for the local reading set; they are not included in a standalone product distribution.

