# Workshop Quick Reference

## Essential Commands

### Service Management
```bash
# Start all services (except Python)
docker compose up -d

# Start with Python profile
docker compose --profile python up -d

# Check service status
docker compose ps

# View logs
docker compose logs ollama
docker compose logs chromadb

# Stop all services
docker compose down
```

### Python Operations (using Docker profile)
```bash
# Run any Python script
docker compose run --rm python python src/script.py

# Install/update dependencies
docker compose run --rm python pip install -r src/requirements.txt

# Enter Python container shell
docker compose run --rm python bash
```

### Model Management
```bash
# List available models
docker compose exec ollama ollama list

# Import model from downloaded file
docker compose exec ollama ollama import --name model-name /root/workshop/data/model/file.tar.gz

# Pull model from Ollama library
docker compose exec ollama ollama pull model-name
```

## Key Files Created

### `docker-compose.yaml`
- **Ollama:** AI model server (port 11434)
- **Open WebUI:** Chat interface (port 8080)
- **ChromaDB:** Vector database (port 8000)
- **Python:** Development environment (profile: python)

### `src/config.py`
- `DATA_DIR`: Path to sample documents
- `EMBEDDING_MODEL`: "nomic-embed-text"
- `OLLAMA_BASE_URL`: "http://localhost:11434"
- `CHROMA_HOST`: "localhost:8000"
- `COLLECTION_NAME`: "workshop-docs"
- `CHUNK_SIZE`: 500
- `CHUNK_OVERLAP`: 50

### Python Scripts
1. **`ingest.py`** - Loads and chunks documents
2. **`embed.py`** - Generates and stores embeddings
3. **`query.py`** - Interactive RAG query engine

## Testing Your Setup

### Step 1: Verify Services
```bash
docker compose ps
# Should show: ollama, open-webui, chromadb
```

### Step 2: Test Ollama
```bash
curl http://localhost:11434/api/tags
# Should return JSON (may be empty)
```

### Step 3: Test Open WebUI
- Open browser: http://localhost:8080
- Should see chat interface

### Step 4: Test ChromaDB
```bash
curl http://localhost:8000/api/v1/heartbeat
# Should return: {"nanosecond heartbeat": ...}
```

### Step 5: Test Python
```bash
docker compose run --rm python python --version
# Should show: Python 3.11.x
```

## Common Queries to Try

```python
# In query.py interactive mode
What are the hardware requirements?
How does RAG work?
What security practices are recommended?
What is this workshop about?
```

## Troubleshooting

### Port Conflicts
```bash
# Check what's using ports
lsof -i :11434  # Ollama
lsof -i :8080   # Open WebUI
lsof -i :8000   # ChromaDB
```

### Docker Issues
```bash
# Clean restart
docker compose down
docker system prune -a
docker compose up -d
```

### Model Issues
```bash
# Check model files
ls -lh data/model/*.tar.gz

# Re-import model
docker compose exec ollama ollama import --name llama3.2:3b /root/workshop/data/model/llama3.2-3b.tar.gz
```

## Useful URLs

- **Open WebUI:** http://localhost:8080
- **Ollama API:** http://localhost:11434
- **ChromaDB:** http://localhost:8000
- **Workshop Docs:** `data/sample-docs/`

## Next Steps After Workshop

1. **Add more documents** to `data/sample-docs/`
2. **Experiment with chunk sizes** in `config.py`
3. **Try different embedding models** (mxbai-embed-large, etc.)
4. **Build a web interface** with Streamlit/FastAPI
5. **Explore advanced RAG techniques** (hybrid search, re-ranking)

## Quick Start After Break

```bash
# If services were stopped
docker compose up -d

# Check status
docker compose ps

# Continue from current section
# (Follow instructor guidance)
```