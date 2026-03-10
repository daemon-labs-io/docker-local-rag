# 🧠 Building Local RAG Systems: Give Your AI a Private Brain

Learn how to give your local AI a private memory using RAG (Retrieval-Augmented Generation). This hands-on workshop builds on our previous session to connect Ollama to your own documents without sending data to the cloud.

---

## 🛑 Prerequisites

Before beginning this workshop, please ensure your environment is correctly set up by following the instructions in our prerequisites documentation:

➡️ **[Prerequisites guide](https://github.com/daemon-labs-io/prerequisites)**

### Download models

The instructor will provide the local network address for the model files:

```bash
mkdir -p data/model

# Download the LLM model
docker run --rm -v $(pwd):/app curlimages/curl -o /app/data/model/llama3.2-3b.tar.gz http://<INSTRUCTOR-IP>:8080/llama3.2-3b.tar.gz

# Download the embedding model
docker run --rm -v $(pwd):/app curlimages/curl -o /app/data/model/nomic-embed-text.tar.gz http://<INSTRUCTOR-IP>:8080/nomic-embed-text.tar.gz
```

---

## What is RAG?

RAG (Retrieval-Augmented Generation) gives your AI access to your own documents:

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Your      │    │  Generate   │    │    LLM      │
│  Documents  │───▶│  Embeddings │───▶│  (Ollama)   │
└─────────────┘    └─────────────┘    └─────────────┘
                          ▲
                          │
                   ┌──────┴──────┐
                   │  ChromaDB   │
                   │(Vector Store)│
                   └─────────────┘
```

**Why RAG?**
- Your AI can cite specific documents
- No need to retrain models with your data
- Works completely offline
- No data leaves your machine

---

## 2. Setup & Start Services

**Goal:** Start the services from the previous workshop and explore our documents.

### Start services

If you don't have the services running from the previous workshop:

```bash
docker compose up
```

Verify Ollama and Open WebUI are running:

```bash
docker compose ps
```

You should see `ollama` and `open-webui` running.

### Explore sample documents

We've included sample documents in `data/sample-docs/`:

```bash
ls -la data/sample-docs/
```

Feel free to add your own Markdown files to this folder.

---

## 3. Add ChromaDB & Python

**Goal:** Add the vector database and Python environment to your docker-compose.yaml.

### Add ChromaDB and Python to docker-compose

Open your `docker-compose.yaml` and add the ChromaDB and Python services:

```yaml
services:
  ollama:
    image: ollama/ollama:latest
    healthcheck:
      test: ["CMD-SHELL", "/usr/bin/ollama list > /dev/null 2>&1 || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    ports:
      - 11434:11434
    volumes:
      - ollama_data:/root/.ollama
      - .:/root/workshop

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    depends_on:
      ollama:
        condition: service_healthy
    ports:
      - 8080:8080
    environment:
      OLLAMA_BASE_URL: http://ollama:11434
    volumes:
      - open-webui_data:/app/backend/data

  chromadb:
    image: chromadb/chroma:latest
    ports:
      - 8000:8000
    volumes:
      - chroma_data:/chroma/chroma
    environment:
      IS_PERSISTENT: "true"
      ANONYMIZED_TELEMETRY: "false"

  python:
    image: python:3.11-slim
    profiles: ["python"]
    volumes:
      - .:/app
      - python_packages:/usr/local/lib/python3.11/site-packages
    working_dir: /app

volumes:
  ollama_data:
  open-webui_data:
  chroma_data:
  python_packages:
```

### Start the new services

```bash
docker compose up
```

Verify all services are running:

```bash
docker compose ps
```

You should see `ollama`, `open-webui`, and `chromadb` running.

### Install Python dependencies

Create a new file `src/requirements.txt`:

```
chromadb>=0.4.22
langchain>=0.1.0
langchain-text-splitters>=0.0.1
requests>=2.31.0
```

Install the dependencies:

```bash
docker compose run --rm python pip install -r src/requirements.txt
```

---

## 4. Document Ingestion

**Goal:** Load and chunk your documents into smaller pieces.

### Create the config file

Create a new file `src/config.py`:

```python
from pathlib import Path

DATA_DIR = Path(__file__).parent.parent / "data" / "sample-docs"
EMBEDDING_MODEL = "nomic-embed-text"
OLLAMA_BASE_URL = "http://localhost:11434"
CHROMA_HOST = "localhost:8000"
COLLECTION_NAME = "workshop-docs"
CHUNK_SIZE = 500
CHUNK_OVERLAP = 50
```

### Create the ingestion script

Create a new file `src/ingest.py`:

```python
import sys
from pathlib import Path
from langchain_text_splitters import RecursiveCharacterTextSplitter

sys.path.insert(0, str(Path(__file__).parent))
import config


def load_documents():
    documents = []
    for file_path in config.DATA_DIR.glob("*.md"):
        with open(file_path, "r", encoding="utf-8") as f:
            content = f.read()
            documents.append({
                "id": file_path.stem,
                "source": file_path.name,
                "content": content
            })
    return documents


def chunk_documents(documents):
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=config.CHUNK_SIZE,
        chunk_overlap=config.CHUNK_OVERLAP,
        separators=["\n\n", "\n", " ", ""]
    )
    
    chunks = []
    for doc in documents:
        splits = splitter.split_text(doc["content"])
        for i, split in enumerate(splits):
            chunks.append({
                "id": f"{doc['id']}-{i}",
                "source": doc["source"],
                "content": split
            })
    return chunks


def main():
    print("Loading documents...")
    documents = load_documents()
    print(f"Loaded {len(documents)} documents")
    
    print("Chunking documents...")
    chunks = chunk_documents(documents)
    print(f"Created {len(chunks)} chunks")
    
    return chunks


if __name__ == "__main__":
    main()
```

### Run the ingestion script

```bash
docker compose run --rm python python src/ingest.py
```

You should see:
```
Loading documents...
Loaded 3 documents
Chunking documents...
Created X chunks
```

> [!TIP]
> The script uses `RecursiveCharacterTextSplitter` with 500 character chunks and 50 character overlap. This preserves context across chunk boundaries.

---

## 5. Generate Embeddings

**Goal:** Convert text chunks into vector embeddings and store in ChromaDB.

### Import the models

If you downloaded the models from the instructor's local server:

```bash
# Import the LLM model
docker compose exec ollama ollama import --name llama3.2:3b /root/workshop/data/model/llama3.2-3b.tar.gz

# Import the embedding model
docker compose exec ollama ollama import --name nomic-embed-text /root/workshop/data/model/nomic-embed-text.tar.gz
```

Verify they're available:

```bash
docker compose exec ollama ollama list
```

### Create the embedding script

Create a new file `src/embed.py`:

```python
import sys
import requests
from pathlib import Path
import chromadb

sys.path.insert(0, str(Path(__file__).parent))
import config


def get_embedding(text):
    response = requests.post(
        f"{config.OLLAMA_BASE_URL}/api/embeddings",
        model=config.EMBEDDING_MODEL,
        json={"prompt": text}
    )
    response.raise_for_status()
    return response.json()["embedding"]


def store_embeddings(chunks):
    client = chromadb.HttpClient(host=config.CHROMA_HOST)

    try:
        client.delete_collection(name=config.COLLECTION_NAME)
    except:
        pass

    collection = client.create_collection(name=config.COLLECTION_NAME)

    ids = [chunk["id"] for chunk in chunks]
    documents = [chunk["content"] for chunk in chunks]
    metadatas = [{"source": chunk["source"]} for chunk in chunks]

    print("Generating embeddings...")
    embeddings = [get_embedding(chunk["content"]) for chunk in chunks]

    collection.add(
        ids=ids,
        documents=documents,
        metadatas=metadatas,
        embeddings=embeddings
    )

    print(f"Stored {len(chunks)} embeddings in ChromaDB")


def main():
    from ingest import load_documents, chunk_documents

    print("Loading documents...")
    documents = load_documents()

    print("Chunking documents...")
    chunks = chunk_documents(documents)
    print(f"Created {len(chunks)} chunks")

    store_embeddings(chunks)

    print("Done!")


if __name__ == "__main__":
    main()
```

### Run the embedding script

```bash
docker compose run --rm python python src/embed.py
```

> [!WARNING]
> First run may take several minutes - it needs to download the embedding model and process all chunks.

You should see:
```
Loading documents...
Chunking documents...
Created X chunks
Generating embeddings...
Stored X embeddings in ChromaDB
Done!
```

> [!NOTE]
> Each chunk is converted to a 768-dimensional vector. These vectors capture semantic meaning - similar concepts will be close together in "vector space".

---

## 6. Query the Knowledge Base

**Goal:** Test RAG with real questions.

### Create the query script

Create a new file `src/query.py`:

```python
import sys
import requests
from pathlib import Path
import chromadb

sys.path.insert(0, str(Path(__file__).parent))
import config


def get_embedding(text):
    response = requests.post(
        f"{config.OLLAMA_BASE_URL}/api/embeddings",
        model=config.EMBEDDING_MODEL,
        json={"prompt": text}
    )
    response.raise_for_status()
    return response.json()["embedding"]


def query_chroma(query_text, n_results=3):
    client = chromadb.HttpClient(host=config.CHROMA_HOST)

    collection = client.get_collection(name=config.COLLECTION_NAME)

    query_embedding = get_embedding(query_text)

    results = collection.query(
        query_embeddings=[query_embedding],
        n_results=n_results
    )

    return results


def generate_response(query, context):
    prompt = f"""Context information:
{context}

Question: {query}

Instructions: Answer the question based on the context above. If the context doesn't contain relevant information, say so. Cite your sources using [source filename]."""

    response = requests.post(
        f"{config.OLLAMA_BASE_URL}/api/generate",
        model="llama3.2:3b",
        json={"prompt": prompt, "stream": False}
    )
    response.raise_for_status()
    return response.json()["response"]


def main():
    print("Local RAG Query Engine")
    print("=" * 40)
    print("Ask questions about your documents (or 'quit' to exit)")
    print()

    while True:
        query = input("You: ").strip()
        if query.lower() in ["quit", "exit", "q"]:
            break

        print("Searching knowledge base...")
        results = query_chroma(query)

        context = "\n\n".join(results["documents"][0])
        sources = list(set([m["source"] for m in results["metadatas"][0]]))

        print("Generating response...")
        response = generate_response(query, context)

        print(f"\nAI: {response}\n")
        print(f"Sources: {', '.join(sources)}\n")


if __name__ == "__main__":
    main()
```

### Run the query engine

```bash
docker compose run --rm python python src/query.py
```

Try these questions:

```
What are the hardware requirements?
How does RAG work?
What security practices are recommended?
```

> [!TIP]
> The query engine:
> 1. Converts your question to an embedding
> 2. Searches ChromaDB for similar document chunks
> 3. Passes the retrieved context + question to Ollama
> 4. Returns an answer with source citations

### Exit the query engine

Type `quit` or `exit` to return to your terminal.

---

## 7. Integration with Open WebUI

**Goal:** Use RAG directly from the chat interface.

### Integration demo

> [!NOTE]
> This section demonstrates advanced integration. The instructor will show a live demo.

### Manual query approach

You can also query directly using curl:

```bash
curl -X POST http://localhost:11434/api/generate \
  -d '{
    "model": "llama3.2:3b",
    "prompt": "Based on the context: [retrieved chunks]. Question: What is RAG?",
    "stream": false
  }'
```

---

## 8. Cleanup

**Goal:** Clean up resources.

### Remove containers

```bash
docker compose down
```

---

## What we accomplished

✅ **Ingested** documents and created text chunks  
✅ **Generated** local embeddings with nomic-embed-text  
✅ **Stored** vectors in ChromaDB  
✅ **Queried** the knowledge base with RAG  
✅ **Demonstrated** full privacy-preserving AI pipeline

---

## When to use RAG

| Use Case | Recommended Approach |
|----------|---------------------|
| Company internal docs | Local RAG |
| Customer support KB | Local RAG |
| Codebase search | Local RAG |
| General knowledge | Standard Ollama |
| Real-time data | Cloud API |

---

## Next steps

Try these extensions:

1. **Add more documents**: Add PDFs or more Markdown files
2. **Different chunking**: Experiment with chunk sizes
3. **Different models**: Try `mxbai-embed-large` for better quality
4. **Web interface**: Build a simple Streamlit UI
5. **Multi-user**: Add authentication for team sharing

---

## 🎉 Congratulations

You've built a complete local RAG system! Your AI can now:
- Access your private documents
- Cite sources for its answers
- Work completely offline
- Keep all data on your machine

This is the foundation for enterprise-grade, privacy-preserving AI applications.

---

## Resources

- [Ollama Documentation](https://github.com/ollama/ollama)
- [ChromaDB Documentation](https://docs.trychroma.com/)
- [LangChain Text Splitters](https://python.langchain.com/docs/modules/data_connection/document_transformers/)
- [Daemon Labs](https://dae.mn)
