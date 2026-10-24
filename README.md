# Hybrid RAG — Spotify Web App Architecture

A PDF-based **Hybrid Retrieval-Augmented Generation (RAG)** application that answers questions about a Spotify-like web application architecture by combining **vector similarity search** with **knowledge-graph retrieval**.

**Live demo:** 

> **Project scope:** This is a question-answering application about the architecture document, **not** a music-streaming implementation of Spotify.

## Features

- Loads a Spotify-style web application architecture PDF.
- Splits document pages into searchable text chunks.
- Generates OpenAI embeddings and stores them in a persistent ChromaDB vector index.
- Extracts entities and relationships into a Neo4j knowledge graph.
- Retrieves information using both semantic similarity (ChromaDB) and graph relationships (Neo4j).
- Generates document-grounded answers with an OpenAI language model.
- Provides a browser-based question-answering interface using Streamlit.
- Runs on a Hostinger VPS using Docker Compose, with HTTPS routing through Traefik.

## Architecture

```text
                   User / Browser
                         |
                         v
               Streamlit UI (app.py)
                         |
                         v
               Hybrid RAG (main.py)
                    /          \
                   v            v
        ChromaDB Vector DB    Neo4j Graph DB
        (semantic search)     (relationships)
                   \            /
                    v          v
                Combined Context
                         |
                         v
                  OpenAI LLM
                         |
                         v
                 Grounded Answer
```

### Document ingestion

```text
Spotify architecture PDF
        |
        v
PDF loading and text chunking
        |                     \
        v                      v
OpenAI embeddings        Entity/relation extraction
        |                      |
        v                      v
ChromaDB                    Neo4j
```

## Technology stack

| Component | Technology |
|---|---|
| Programming language | Python 3.12 |
| User interface | Streamlit |
| RAG orchestration | LangChain |
| Embeddings | OpenAI `text-embedding-3-small` |
| Answer generation / graph extraction | OpenAI `gpt-4.1-mini` |
| Vector database | ChromaDB |
| Knowledge graph | Neo4j AuraDB |
| PDF processing | PyPDF / LangChain PDF loader |
| Containerization | Docker and Docker Compose |
| Reverse proxy / HTTPS | Traefik |
| Hosting | Hostinger VPS |
| Source control | GitHub |

## Project structure

```text
hybrid-rag-spotify/
├── app.py                 # Streamlit web interface
├── main.py                # PDF processing and hybrid retrieval
├── requirements.txt       # Python dependencies
├── Dockerfile             # Application container image
├── compose.yaml           # Docker Compose deployment
├── data/
│   └── spotify_web_app_architecture.pdf
├── .gitignore
├── .dockerignore
└── README.md
```

Generated files and secrets such as `chroma_db/`, `venv/`, `.env`, and `.env.production` should not be committed.

## Run locally (Windows CMD)

### 1. Clone and create a virtual environment

```cmd
git clone https://github.com/Prithvi11J/hybrid-rag-spotify.git
cd hybrid-rag-spotify
py -3.12 -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

This repository is private, so cloning requires authorized GitHub access.

### 2. Configure environment variables

The application expects these variables:

```dotenv
OPENAI_API_KEY=your_openai_api_key
NEO4J_URI=your_neo4j_uri
NEO4J_USERNAME=your_neo4j_username
NEO4J_PASSWORD=your_neo4j_password
NEO4J_DATABASE=neo4j
```

On the original development machine, these were configured as Windows environment variables. Never place real credentials in GitHub or source code.

### 3. Initialize and run

```cmd
python main.py
python -m streamlit run app.py
```

`main.py` prepares/reuses the document indexes and may make billable OpenAI API calls. The Streamlit app is then available at `http://localhost:8501`.

## Deploy to Hostinger VPS

The VPS uses an existing Traefik reverse proxy, which is connected to the Docker network `n8n_default`.

1. Clone the private GitHub repository into `/opt/hybrid-rag` using a **read-only GitHub SSH deploy key**.
2. Create `/opt/hybrid-rag/.env.production` with the required OpenAI and Neo4j credentials; set file permissions with `chmod 600`.
3. Confirm `compose.yaml` routes the `hybrid-rag` service to `rag.prithvi11j.tech` on internal port `8501` through Traefik's HTTPS entrypoint.
4. Build and start the service:

```bash
cd /opt/hybrid-rag
docker compose config --quiet
docker compose up -d --build
docker compose ps
```

5. If the ChromaDB index has not been generated on the VPS, initialize it and restart the application:

```bash
docker compose exec hybrid-rag python main.py
docker compose restart hybrid-rag
```

6. Visit **https://rag.prithvi11j.tech** and test a document-based question.

### Updating after new GitHub commits

```bash
cd /opt/hybrid-rag
GIT_SSH_COMMAND="ssh -i /root/.ssh/hybrid_rag_deploy -o IdentitiesOnly=yes" git pull origin main
docker compose up -d --build
```

The VPS and local laptop have separate environment-variable configurations. Credentials must never be pushed to GitHub.

## Example questions

- Which database stores user profiles, playlists, and subscriptions? (**PostgreSQL**)
- Which service manages playback sessions and active devices? (**Playback & Session Service**)
- What does the architecture describe about playlist creation?
- What services and technologies support the web application's design?

The application is designed to answer only from the supplied PDF and retrieved context. It should acknowledge when the document does not contain an answer.

## Security and cost considerations

- Keep API keys and Neo4j credentials server-side and out of Git history.
- Use a read-only SSH deploy key for VPS access to GitHub.
- Be aware that a public demo allows visitors to trigger **billable OpenAI API calls** without seeing the API key.
- Configure OpenAI project spend alerts and, if available, enforced limits.
- Use rate limiting or authentication if traffic increases.
- Do not share sensitive information through screenshots, terminal output, or commits.

## Project outcome

Built and tested an end-to-end Hybrid RAG system combining vector search and graph relationships, then deployed the Streamlit interface to a custom HTTPS domain using GitHub, Docker Compose, and a Hostinger VPS.
