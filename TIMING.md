# 90-Minute Workshop Timing Guide

## Overview
- **Total time:** 90 minutes
- **Sections:** 6 × 15-minute segments
- **Break:** 5 minutes after Section 3 (45-50 minute mark)
- **Approach:** Copy-paste only, instructor-led pace
- **Goal:** Everyone completes each section together

## Section Breakdown

### **Section 1: Environment & Infrastructure (0-15 mins)**
**Goal:** Get all services running

| Time | Activity | Copy-Paste Commands | Verification |
|------|----------|---------------------|--------------|
| 0-5 | Welcome + RAG overview | - | - |
| 5-10 | Download models | `mkdir -p data/model`<br>`docker run --rm -v $(pwd):/app curlimages/curl -o /app/data/model/llama3.2-3b.tar.gz http://<INSTRUCTOR-IP>:8080/llama3.2-3b.tar.gz`<br>`docker run --rm -v $(pwd):/app curlimages/curl -o /app/data/model/nomic-embed-text.tar.gz http://<INSTRUCTOR-IP>:8080/nomic-embed-text.tar.gz` | `ls data/model/*.tar.gz` |
| 10-15 | Start services | `docker compose up -d`<br>`docker compose ps` | Should show ollama, open-webui running |

**Deliverable:** Running services + models downloaded

---

### **Section 2: Python Setup & Configuration (15-30 mins)**
**Goal:** Set up Python environment

| Time | Activity | Copy-Paste Commands | Verification |
|------|----------|---------------------|--------------|
| 15-20 | Append to docker-compose.yaml | Copy ChromaDB + Python services from README | `docker compose config` (no errors) |
| 20-25 | Create requirements.txt | Create `src/requirements.txt` with 4 lines | `cat src/requirements.txt` |
| 25-30 | Install dependencies | `docker compose run --rm python pip install -r src/requirements.txt` | Should see successful installation |

**Deliverable:** Python environment ready

---

### **Section 3: Document Ingestion (30-45 mins)**
**Goal:** Load and chunk documents

| Time | Activity | Copy-Paste Commands | Verification |
|------|----------|---------------------|--------------|
| 30-35 | Create config.py | Create `src/config.py` with 11 lines | `cat src/config.py` |
| 35-40 | Create ingest.py | Create `src/ingest.py` with 55 lines | Check file exists |
| 40-45 | Run ingestion | `docker compose run --rm python python src/ingest.py` | Should show "Loaded 3 documents" |

**Deliverable:** Document chunks ready for embedding

---

### **BREAK: 45-50 minutes**

---

### **Section 4: Embedding Generation (50-65 mins)**
**Goal:** Generate and store embeddings

| Time | Activity | Copy-Paste Commands | Verification |
|------|----------|---------------------|--------------|
| 50-55 | Import models | `docker compose exec ollama ollama import --name llama3.2:3b /root/workshop/data/model/llama3.2-3b.tar.gz`<br>`docker compose exec ollama ollama import --name nomic-embed-text /root/workshop/data/model/nomic-embed-text.tar.gz` | `docker compose exec ollama ollama list` (2 models) |
| 55-60 | Create embed.py | Create `src/embed.py` with 64 lines | Check file exists |
| 60-65 | Start embedding | `docker compose run --rm python python src/embed.py` | Should start "Generating embeddings..." |

**Deliverable:** Embeddings stored in ChromaDB (or in progress)

---

### **Section 5: Query Testing (65-80 mins)**
**Goal:** Test complete RAG pipeline

| Time | Activity | Copy-Paste Commands | Verification |
|------|----------|---------------------|--------------|
| 65-70 | Create query.py | Create `src/query.py` with 78 lines | Check file exists |
| 70-75 | Test queries | `docker compose run --rm python python src/query.py`<br>Test: "What are the hardware requirements?" | Should return answer with `[source filename]` |
| 75-80 | Explore results | Test: "How does RAG work?"<br>Test: "What security practices are recommended?" | Verify citations work |

**Deliverable:** Working RAG query system

---

### **Section 6: Demo & Cleanup (80-90 mins)**
**Goal:** See integration options, clean up

| Time | Activity | Copy-Paste Commands | Verification |
|------|----------|---------------------|--------------|
| 80-83 | Open WebUI demo | Instructor shows integration | Visual confirmation |
| 83-86 | Manual API example | `curl -X POST http://localhost:11434/api/generate -d '{"model": "llama3.2:3b", "prompt": "Based on context...", "stream": false}'` | Should return response |
| 86-89 | Cleanup | `docker compose down` | Services stop |
| 89-90 | Q&A + resources | - | - |

**Deliverable:** Clean environment + understanding of next steps

---

## Instructor Notes

### **Pace Management:**
1. **Projector display:** Show exact copy-paste targets from README
2. **Wait for all:** Pause at verification points until all complete
3. **Time announcements:** "We should be at step X by minute Y"
4. **Assistants:** Designate helpers for those falling behind

### **Critical Timing:**
- **Model downloads:** Must complete in Section 1 (use local network)
- **Embedding generation:** Start in Section 4, may continue into Section 5
- **Break timing:** Perfect after ingestion, before embedding
- **Buffer:** 1-2 minutes buffer in each section

### **Verification Commands Summary:**
```bash
# Section 1
docker compose ps
ls data/model/*.tar.gz

# Section 2
docker compose config
cat src/requirements.txt

# Section 3
cat src/config.py
docker compose run --rm python python src/ingest.py

# Section 4
docker compose exec ollama ollama list
docker compose run --rm python python src/embed.py

# Section 5
docker compose run --rm python python src/query.py

# Section 6
docker compose down
docker compose ps
```

### **Troubleshooting Ready:**
```bash
# If Docker issues
docker system prune -a
docker compose down --volumes

# If port conflicts
# Check ports 11434, 8080, 8000
lsof -i :11434

# If model import fails
# Use instructor USB backup
```

### **Success Criteria:**
1. ✅ All services running (Section 1)
2. ✅ Python environment + config (Section 2)
3. ✅ Documents ingested (Section 3)
4. ✅ Embeddings generated (Section 4)
5. ✅ Query engine working (Section 5)
6. ✅ Clean shutdown (Section 6)