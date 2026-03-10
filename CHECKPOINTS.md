# Workshop Progress Checkpoints

## Section 1 Checkpoints (0-15 mins)

### ✅ Models Downloaded
```bash
ls -la data/model/
```
**Should show:**
```
llama3.2-3b.tar.gz
nomic-embed-text.tar.gz
```

### ✅ Services Running
```bash
docker compose ps
```
**Should show:**
```
NAME        STATUS              PORTS
ollama      Up X minutes        0.0.0.0:11434->11434/tcp
open-webui  Up X minutes        0.0.0.0:8080->8080/tcp
```

### ✅ Ollama Accessible
```bash
curl http://localhost:11434/api/tags
```
**Should return:** JSON with available models (may be empty initially)

---

## Section 2 Checkpoints (15-30 mins)

### ✅ docker-compose.yaml Updated
```bash
docker compose config
```
**Should show:** No errors, includes chromadb and python services

### ✅ requirements.txt Created
```bash
cat src/requirements.txt
```
**Should show:**
```
chromadb>=0.4.22
langchain>=0.1.0
langchain-text-splitters>=0.0.1
requests>=2.31.0
```

### ✅ Dependencies Installed
```bash
docker compose run --rm python pip list | grep -E "(chromadb|langchain|requests)"
```
**Should show:** All packages installed (first run will install them)

---

## Section 3 Checkpoints (30-45 mins)

### ✅ config.py Created
```bash
cat src/config.py
```
**Should show:** Configuration with correct paths and settings

### ✅ ingest.py Created
```bash
ls -la src/ingest.py
```
**Should show:** File exists with ~55 lines

### ✅ Documents Ingested
```bash
docker compose run --rm python python src/ingest.py
```
**Should show:**
```
Loading documents...
Loaded 3 documents
Chunking documents...
Created X chunks
```

---

## Section 4 Checkpoints (50-65 mins)

### ✅ Models Imported
```bash
docker compose exec ollama ollama list
```
**Should show:**
```
NAME               ID              SIZE      MODIFIED
llama3.2:3b        ...             2.0 GB    X seconds ago
nomic-embed-text   ...             0.4 GB    X seconds ago
```

### ✅ embed.py Created
```bash
ls -la src/embed.py
```
**Should show:** File exists with ~64 lines

### ✅ Embeddings Started
```bash
docker compose run --rm python python src/embed.py
```
**Should show:**
```
Loading documents...
Chunking documents...
Created X chunks
Generating embeddings...
Stored X embeddings in ChromaDB
Done!
```

---

## Section 5 Checkpoints (65-80 mins)

### ✅ query.py Created
```bash
ls -la src/query.py
```
**Should show:** File exists with ~78 lines

### ✅ Query Engine Working
```bash
echo -e "What are the hardware requirements?\nquit" | docker compose run --rm python python src/query.py
```
**Should show:**
```
Local RAG Query Engine
========================================
Ask questions about your documents (or 'quit' to exit)

You: What are the hardware requirements?
Searching knowledge base...
Generating response...

AI: [Answer with citations]
Sources: [filename.md]
You: quit
```

### ✅ Citations Working
**Check:** Response includes `[source filename]` citations

---

## Section 6 Checkpoints (80-90 mins)

### ✅ Services Stopped
```bash
docker compose down
docker compose ps
```
**Should show:** No services running

### ✅ Clean Environment
```bash
ls -la
```
**Should show:** All workshop files intact, ready for future use

---

## Troubleshooting Quick Reference

### Docker Issues
```bash
# Restart Docker
sudo systemctl restart docker  # Linux
# Restart Docker Desktop  # macOS/Windows

# Clean up
docker compose down --volumes
docker system prune -a
```

### Port Conflicts
```bash
# Check what's using ports
lsof -i :11434  # Ollama
lsof -i :8080   # Open WebUI
lsof -i :8000   # ChromaDB

# Alternative ports (edit docker-compose.yaml)
# ports: - 11435:11434  # Ollama
# ports: - 8081:8080    # Open WebUI
# ports: - 8001:8000    # ChromaDB
```

### Model Import Issues
```bash
# Verify model files
ls -lh data/model/*.tar.gz

# Manual import
docker compose exec ollama ollama import --name llama3.2:3b /root/workshop/data/model/llama3.2-3b.tar.gz

# Check Ollama logs
docker compose logs ollama
```

### Python Dependency Issues
```bash
# Reinstall
docker compose run --rm python pip install --upgrade -r src/requirements.txt

# Check Python version
docker compose run --rm python python --version
```

### ChromaDB Issues
```bash
# Check if running
curl http://localhost:8000/api/v1/heartbeat

# Restart ChromaDB
docker compose restart chromadb

# Check logs
docker compose logs chromadb
```

---

## Success Criteria Summary

By the end of each section, you should have:

1. **Section 1:** ✅ Services running + models downloaded
2. **Section 2:** ✅ Python environment + docker-compose updated
3. **Section 3:** ✅ Documents ingested and chunked
4. **Section 4:** ✅ Embeddings generated in ChromaDB
5. **Section 5:** ✅ Query engine working with citations
6. **Section 6:** ✅ Clean shutdown + next steps understood

If you're missing any checkpoint, ask for help or review the previous steps before proceeding.