# Instructor Guide: 90-Minute RAG Workshop

## Before the Workshop

### 1. Local Network Setup
```bash
# Set up HTTP server for model distribution
python3 -m http.server 8080 --directory /path/to/models &

# Models needed (download beforehand):
# - llama3.2-3b.tar.gz (~2GB)
# - nomic-embed-text.tar.gz (~0.4GB)
```

### 2. Instructor Machine Setup
```bash
# Test complete flow
./test-workshop.sh
docker compose up -d
docker compose run --rm python pip install -r src/requirements.txt
docker compose run --rm python python src/ingest.py
# ... test full flow
```

### 3. Backup Preparation
- USB drives with model files
- Printed troubleshooting guide
- Screenshots of each verification step

## Workshop Flow (90 Minutes)

### **Section 1: Environment & Infrastructure (0-15 mins)**
**Instructor actions:**
1. Welcome (2 min) - Quick RAG overview
2. Share local IP address for model downloads
3. Guide through model download commands
4. Verify `docker compose ps` shows ollama + open-webui

**Critical:** Ensure all have models downloaded by minute 10

### **Section 2: Python Setup & Configuration (15-30 mins)**
**Instructor actions:**
1. Show updated docker-compose.yaml on projector
2. Guide copy-paste of ChromaDB + Python services
3. Start all services, verify 3 services running
4. Create requirements.txt, install dependencies

**Watch for:** Docker Compose errors, port conflicts

### **Section 3: Document Ingestion (30-45 mins)**
**Instructor actions:**
1. Create config.py (explain paths)
2. Create ingest.py (explain chunking)
3. Run ingestion, verify 3 documents loaded
4. **Announce break at 45 minutes**

### **BREAK: 45-50 minutes**

### **Section 4: Embedding Generation (50-65 mins)**
**Instructor actions:**
1. Import models into Ollama (2 commands)
2. Verify both models show in `ollama list`
3. Create embed.py (explain embedding process)
4. **Start embedding generation** - this takes time!

**While embedding runs:** Explain vector concepts, ChromaDB

### **Section 5: Query Testing (65-80 mins)**
**Instructor actions:**
1. Create query.py (explain RAG flow)
2. Test with sample questions
3. Show source citations in answers
4. Compare with/without RAG

**If embeddings not done:** Use instructor's pre-generated system for demo

### **Section 6: Demo & Cleanup (80-90 mins)**
**Instructor actions:**
1. Demo Open WebUI integration (2 min)
2. Show manual API query with curl (2 min)
3. Cleanup commands (2 min)
4. Q&A + resources (4 min)

## Timing Management Tips

### **Keep Everyone Together:**
- Projector shows exact copy-paste targets
- Wait at verification points until all complete
- Announce "We should be at step X by minute Y"
- Designate assistants to help those falling behind

### **Critical Time Points:**
- **Minute 10:** All should have models downloaded
- **Minute 25:** All should have services running
- **Minute 40:** All should have documents ingested
- **Minute 65:** Embedding should be running
- **Minute 75:** All should have tested queries

### **Buffer Time Allocation:**
- Section 1: +2 min for model downloads
- Section 4: +3 min for embedding start
- Section 6: +2 min for Q&A

## Common Issues & Solutions

### **Docker Issues:**
```bash
# If services won't start
docker compose down
docker system prune -a
docker compose up -d

# Port conflicts
lsof -i :11434  # Ollama
lsof -i :8080   # Open WebUI  
lsof -i :8000   # ChromaDB
```

### **Model Import Issues:**
- USB backup: `cp /path/to/usb/models/*.tar.gz data/model/`
- Manual import commands in README

### **Python Issues:**
```bash
# Reinstall dependencies
docker compose run --rm python pip install --upgrade -r src/requirements.txt

# Check Python version
docker compose run --rm python python --version
```

### **ChromaDB Issues:**
```bash
# Check if running
curl http://localhost:8000/api/v1/heartbeat

# Restart
docker compose restart chromadb
```

## Success Criteria

By workshop end, attendees should have:

1. ✅ Running services: Ollama, Open WebUI, ChromaDB
2. ✅ Python environment with dependencies
3. ✅ Documents ingested and chunked  
4. ✅ Embeddings generated (or in progress)
5. ✅ Query engine working with citations
6. ✅ Understanding of RAG workflow

## Post-Workshop

### **Share Resources:**
- Repository URL
- Extended documentation
- Community channels

### **Collect Feedback:**
- What worked well?
- What was confusing?
- Timing suggestions
- Technical issues encountered

### **Follow-up:**
- Share completed code
- Provide advanced exercises
- Schedule office hours for questions