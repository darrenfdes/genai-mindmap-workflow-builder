# GenAI Mind-Map Workflow Builder

> 🥉 **3rd place, AWS Hackathon 2025**: [LinkedIn post](https://www.linkedin.com/posts/darren-fernandes-909138219_amazonhackathon-genai-mindmap-activity-7370102909772054528-DMzJ)

This app turns long-form content into an interactive mind map. Add a source (a PDF, a slide deck, a spreadsheet, a database, a web page, a YouTube video or a meeting recording) and ask questions about it. Every answer becomes a node on a canvas that you can branch, follow up on, compare and summarise into a single report.

![Landing page](screenshots/image1.png)

<!-- TODO: add 1 screenshot of a populated canvas (e.g. screenshots/imageN.png) -->

## What it does

- **13 source types:** PDF, DOCX, PPTX, TXT, Markdown, HTML, CSV, SQL (SQLite), web pages, YouTube, video, audio and images.
- **Canvas-based Q&A:** each source is a root node, and each question and answer is a child node. Follow-up questions branch from any answer.
- **Personas:** answers can be framed as *Strategic Advisor*, *Research Assistant*, *Productivity Coach*, *Data Interpreter* or a custom prompt.
- **Auto mind map:** in "automatic" mode the LLM reads a document summary and produces the React Flow graph (nodes, edges, layout) directly as JSON.
- **Ask multiple and summarise:** you can ask several questions at once, then roll the whole flow up into an executive summary with tables and charts, and export it as a PDF.

## Architecture

```
React + Vite (React Flow canvas, Zustand, MUI, Plotly)
        │  REST
        ▼
FastAPI ──► MongoDB          flows / components / nodes
   │    ──► Amazon S3        uploaded files
   │
   ├─ Documents ─► Textract (async, tables + forms) │ unstructured + Camelot │ GPT-4o
   │               └─► semantic chunking (OpenAI embeddings) ─► Chroma vector store ─► RAG Q&A
   ├─ CSV / SQL ─► Vanna text-to-SQL (GPT-4o + Chroma) over SQLite ─► table + chart
   ├─ Web pages ─► Crawl4AI ─► GPT-4o
   └─ Image / audio / video / YouTube ─► Gemini 2.0 Flash (Vertex AI)
```

Notable details:

- **Three PDF pipelines, chosen per upload:** GPT-4o directly for short documents (within a token-limit check), **Amazon Textract** asynchronous document analysis (`TABLES` + `FORMS`) via S3 for scanned or table-heavy files, and a local `unstructured` + Camelot path that extracts text and tables without a cloud OCR call.
- **Map-reduce summarisation** over semantically chunked documents (Textract and Camelot paths).
- **Deduplication:** uploads are SHA-256 hashed and rejected if the same file already exists in the flow.
- **Text-to-SQL:** each uploaded CSV is loaded into its own SQLite table. Its DDL is used to train a Vanna agent, so natural-language questions return a result table and a generated Plotly chart.

## Tech stack

**Frontend:** React 18, Vite, @xyflow/react (React Flow), dagre, Zustand, MUI, Plotly, AG Grid, jsPDF
**Backend:** Python, FastAPI, LangChain, OpenAI GPT-4o, Gemini 2.0 Flash (Vertex AI), ChromaDB, Vanna, Crawl4AI, unstructured, Camelot
**AWS:** Amazon S3, Amazon Textract
**Data:** MongoDB, SQLite

## Running locally

```bash
# backend
cd backend
poetry install            # or: pip install -r requirements.txt
# create .env with the variables below
uvicorn app:app --reload

# frontend
cd frontend
npm install
npm run dev
```

The backend reads these variables from `.env`: `mongo_db_url`, `openai_api_key`, `gemini_api_key`, `gcp_project_id`, `aws_access_key_id`, `aws_secret_access_key` and `bucket_name`. It also needs a GCP service-account JSON file for Vertex AI. The file path is set in `app.py`.

## Team

Built as a team at AWS Hackathon 2025. My part was **document extraction and processing**: the PDF pipelines (Amazon Textract, unstructured + Camelot, and direct GPT-4o), table extraction, semantic chunking into the vector store, and map-reduce summarisation described above.

## Status

This is hackathon code, built in a few days and kept as a snapshot. It isn't maintained or production-hardened.
