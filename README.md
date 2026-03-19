# 🧠 Building Local RAG Systems: Give Your AI a Private Brain

Learn how to give your local AI a private memory using RAG (Retrieval-Augmented Generation). This hands-on workshop builds on our previous session to connect Ollama to your own documents without sending data to the cloud.

---

## 🛑 Prerequisites

Before beginning this workshop, please ensure your environment is correctly set up by following the instructions in our prerequisites documentation:

➡️ **[Prerequisites guide](https://github.com/daemon-labs-io/prerequisites)**

### Create project folder

Create a new folder for your project:

```shell
mkdir ./docker-local-rag
```

> [!NOTE]
> You can either create this via a terminal window or your file explorer.

### Open the new folder in your code editor

```shell
code ./docker-local-rag
```

> [!NOTE]
> If this command doesn't work, open Visual Studio Code → Cmd+Shift+P → type "Install 'code' command in PATH" → press Enter.

<!--  -->

> [!TIP]
> If you are using Visual Studio Code, we can now do everything from within the code editor.  
> You can open the terminal pane via Terminal -> New Terminal.

### In-person workshop prerequisites

> [!CAUTION]  
> **This only applies when attending a workshop in person.**

<details>
<summary>⚠️ If you are in an in-person workshop, expand this section.</summary>

#### Pull Docker images from local mirror

The facilitator will provide the local mirror address. Pull the required images directly:

```shell
# Pull from local mirror (instructor will provide the address)
docker pull registry.labs.dae.mn/curl:latest
docker pull registry.labs.dae.mn/ollama:latest
docker pull registry.labs.dae.mn/open-webui:main
docker pull registry.labs.dae.mn/chroma:latest
docker pull registry.labs.dae.mn/python:3.11-slim


# Retag to standard names for use in docker-compose
docker tag registry.labs.dae.mn/curl:latest curlimages/curl:latest
docker tag registry.labs.dae.mn/ollama:latest ollama/ollama:latest
docker tag registry.labs.dae.mn/open-webui:main ghcr.io/open-webui/open-webui:main
docker tag registry.labs.dae.mn/chroma:latest chromadb/chroma:latest
docker tag registry.labs.dae.mn/python:3.11-slim python:3.11-slim
```

> This approach works with Docker Desktop, Rancher Desktop, Podman, and other Docker runtimes.

#### Download models

The facilitator will provide the local network address for the model files:

```shell
docker run --rm -v $(pwd):/data/models curlimages/curl -o /data/model/tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf https://files.labs.dae.mn/tinyllama-1.1b-chat-v1.0.Q4_K_M.gguf
```

```shell
docker run --rm -v $(pwd):/data/models curlimages/curl -o /data/model/nomic-embed-text.tar.gz https://files.labs.dae.mn/nomic-embed-text.tar.gz
```

#### Download sample documents

The facilitator will provide the local network address for the sample documents:

```shell
docker run --rm -v $(pwd):/data/sample-docs curlimages/curl -o /data/sample-docs/01-welcome.md https://files.labs.dae.mn/01-welcome.md
```

```shell
docker run --rm -v $(pwd):/data/sample-docs curlimages/curl -o /data/sample-docs/02-security-policy.md https://files.labs.dae.mn/02-security-policy.md
```

```shell
docker run --rm -v $(pwd):/data/sample-docs curlimages/curl -o /data/sample-docs/03-faq.md https://files.labs.dae.mn/03-faq.md
```

</details>

> [!NOTE]  
> **Fallback for low-resource devices:** If your laptop cannot run the local AI model due to insufficient RAM/GPU, you can connect to the instructor's running instance instead. The instructor will provide their local network IP address - you'll use this in Section 1 when configuring the connection.

---

## What is RAG?

RAG (Retrieval-Augmented Generation) gives your AI access to your own documents:

```text
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Your      │    │  Generate   │    │    LLM      │
│  Documents  │───▶│  Embeddings │───▶│  (Ollama)   │
└─────────────┘    └─────────────┘    └─────────────┘
                          ▲
                          │
                   ┌──────┴───────┐
                   │   ChromaDB   │
                   │(Vector Store)│
                   └──────────────┘
```

**Why RAG?**

- Your AI can cite specific documents
- No need to retrain models with your data
- Works completely offline
- No data leaves your machine

---

## 1. Setup & start services

**Goal:** Start the services from the previous workshop and explore our documents.

### Create the Docker Compose file

Create a `docker-compose.yaml` file in the root of your project and add the following:

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
  open_webui:
    image: ghcr.io/open-webui/open-webui:main
    depends_on:
      ollama:
        condition: service_healthy
    ports:
      - 8080:8080
    environment:
      OLLAMA_BASE_URL: http://ollama:11434
    volumes:
      - open_webui_data:/app/backend/data
volumes:
  ollama_data:
  open_webui_data:
```

### Start the services

Start the Docker services:

```shell
docker compose up
```

> [!NOTE]
> You should see the `ollama` service starts up followed by the `open-webui` service.

Verify all services are running:

```shell
docker compose ps
```

> [!TIP]
> In Visual Studio Code, you can open a new terminal via Terminal → Split Terminal or the + button to run this command while `docker compose up` runs in the current terminal.

### Final check

Open your browser and navigate to the following:

```text
http://localhost:8080
```

> [!NOTE]
> You should see the Open WebUI interface with "Select a model" dropdown (empty for now).

### Create your account

On first access, Open WebUI requires you to create an admin account.

Enter any name, email, and password to create your account.

> [!NOTE]
> This account is local to your instance - no external verification is needed.

---

## 2. Add ChromaDB & Python

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
  open_webui:
    image: ghcr.io/open-webui/open-webui:main
    depends_on:
      ollama:
        condition: service_healthy
    ports:
      - 8080:8080
    environment:
      OLLAMA_BASE_URL: http://ollama:11434
    volumes:
      - open_webui_data:/app/backend/data
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
    profiles:
      - python
    volumes:
      - .:/app
      - python_packages:/usr/local/lib/python3.11/site-packages
    working_dir: /app
volumes:
  ollama_data:
  open_webui_data:
  chroma_data:
  python_packages:
```

> [!NOTE]
> **Exit your container by pressing Ctrl+C on your keyboard.**

### Start the new services

```shell
docker compose up
```

Verify all services are running:

```shell
docker compose ps
```

You should see `ollama`, `open-webui`, and `chromadb` running.

### Install Python dependencies

Create a new file `src/requirements.txt`:

```text
chromadb>=0.4.22
langchain>=0.1.0
langchain-text-splitters>=0.0.1
requests>=2.31.0
```

Install the dependencies:

```shell
docker compose run --rm python pip install -r src/requirements.txt
```

---

## 3. Document ingestion

**Goal:** Load and chunk your documents into smaller pieces.

### Create the config file

Create a new file `src/config.py`:

```python
from pathlib import Path

DATA_DIR = Path(__file__).parent.parent / "data" / "sample-docs"
EMBEDDING_MODEL = "nomic-embed-text"
GENERATION_MODEL = "tinyllama:latest"
OLLAMA_BASE_URL = "http://ollama:11434"
CHROMA_HOST = "chromadb"
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

```shell
docker compose run --rm python python src/ingest.py
```

You should see:

```text
Loading documents...
Loaded 3 documents
Chunking documents...
Created X chunks
```

> [!TIP]
> The script uses `RecursiveCharacterTextSplitter` with 500 character chunks and 50 character overlap. This preserves context across chunk boundaries.

---

> [!IMPORTANT]
> Let's take a break.

---

## 4. Generate embeddings

**Goal:** Convert text chunks into vector embeddings and store in ChromaDB.

### Import the models

If you downloaded the models from the instructor's local server...

Import the LLM model:

```shell
docker compose exec ollama ollama import --name llama3.2:3b /root/workshop/data/model/llama3.2-3b.tar.gz
```

Import the embedding model:

```shell
docker compose exec ollama ollama import --name nomic-embed-text /root/workshop/data/model/nomic-embed-text.tar.gz
```

Verify they're available:

```shell
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
        json={"model": config.EMBEDDING_MODEL, "prompt": text},
    )
    response.raise_for_status()
    return response.json()["embedding"]


def store_embeddings(chunks):
    client = chromadb.HttpClient(host=config.CHROMA_HOST, port=8000)

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
        ids=ids, documents=documents, metadatas=metadatas, embeddings=embeddings
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

```shell
docker compose run --rm python python src/embed.py
```

> [!WARNING]
> First run may take several minutes - it needs to download the embedding model and process all chunks.

You should see:

```text
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

## 5. Query the knowledge base

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
        json={"model": config.EMBEDDING_MODEL, "prompt": text},
    )
    response.raise_for_status()
    return response.json()["embedding"]


def query_chroma(query_text, n_results=3):
    client = chromadb.HttpClient(host=config.CHROMA_HOST, port=8000)

    collection = client.get_collection(name=config.COLLECTION_NAME)

    query_embedding = get_embedding(query_text)

    results = collection.query(query_embeddings=[query_embedding], n_results=n_results)

    return results


def generate_response(query, context):
    prompt = f"""Context information:
{context}

Question: {query}

Instructions: Answer the question based on the context above. If the context doesn't contain relevant information, say so. Cite your sources using [source filename]."""

    response = requests.post(
        f"{config.OLLAMA_BASE_URL}/api/generate",
        json={"model": config.GENERATION_MODEL, "prompt": prompt, "stream": False},
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

```shell
docker compose run --rm python python src/query.py
```

Try these questions:

```text
What are the hardware requirements?
```

<!--  -->

```text
How does RAG work?
```

<!--  -->

```text
What security practices are recommended?
```

> [!TIP]
> The query engine:
>
> 1. Converts your question to an embedding
> 2. Searches ChromaDB for similar document chunks
> 3. Passes the retrieved context + question to Ollama
> 4. Returns an answer with source citations

### Exit the query engine

Type `quit` or `exit` to return to your terminal.

---

## 6. Integration with Open WebUI

**Goal:** Make Open WebUI use your RAG setup.

Now that you have a working RAG pipeline, let's connect it to Open WebUI so you can use RAG directly from the chat interface.

### Simple Approach: RAG Bridge Service

The easiest way is to create a small Python service that sits between Open WebUI and Ollama:

1. **Create a bridge script** (`rag_bridge.py`) that:
   - Listens for queries from Open WebUI
   - Adds RAG context from ChromaDB
   - Forwards to Ollama
   - Returns responses with source citations

2. **Update your setup** to use this bridge instead of direct Ollama

### Step 1: Create the RAG Bridge

Create `src/rag_bridge.py`:

```python
import requests
import chromadb
from flask import Flask, request, jsonify

app = Flask(__name__)

# Use the same config as your query.py
CHROMA_HOST = "chromadb"
COLLECTION_NAME = "workshop-docs"
OLLAMA_URL = "http://ollama:11434"

def get_embedding(text):
    """Get embedding from Ollama"""
    response = requests.post(
        f"{OLLAMA_URL}/api/embeddings",
        json={"model": "nomic-embed-text", "prompt": text},
    )
    return response.json()["embedding"]

def query_chroma(query_text):
    """Query ChromaDB for relevant documents"""
    client = chromadb.HttpClient(host=CHROMA_HOST, port=8000)
    collection = client.get_collection(name=COLLECTION_NAME)

    query_embedding = get_embedding(query_text)
    results = collection.query(query_embeddings=[query_embedding], n_results=3)

    return results

@app.route('/api/generate', methods=['POST'])
def generate():
    """Handle Open WebUI queries with RAG"""
    data = request.json
    query = data.get('prompt', '')

    # Get relevant documents from ChromaDB
    results = query_chroma(query)

    if results['documents']:
        # Build context from retrieved documents
        context = "\n\n".join(results['documents'][0])
        sources = list(set([m['source'] for m in results['metadatas'][0]]))

        # Create RAG-enhanced prompt
        rag_prompt = f"""Context from documents:
{context}

Question: {query}

Answer based on the context above. Cite sources with [source: filename]."""

        # Forward to Ollama
        response = requests.post(
            f"{OLLAMA_URL}/api/generate",
            json={"model": "tinyllama:latest", "prompt": rag_prompt, "stream": False}
        )

        result = response.json()
        # Add source citations
        result['response'] += f"\n\nSources: {', '.join(sources)}"
        return jsonify(result)

    # No relevant documents found
    return jsonify({
        "response": "I couldn't find relevant information in my knowledge base.",
        "sources": []
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### Step 2: Update Requirements

Add Flask to your `src/requirements.txt`:

```text
chromadb>=0.4.22
langchain>=0.1.0
langchain-text-splitters>=0.0.1
requests>=2.31.0
flask>=3.0.0
```

Install it:

```shell
docker compose run --rm python pip install -r src/requirements.txt
```

### Step 3: Update Docker Compose

Add the RAG bridge service to your `docker-compose.yaml`:

```yaml
rag-bridge:
  image: python:3.11-slim
  ports:
    - 5000:5000
  depends_on:
    - ollama
    - chromadb
  volumes:
    - .:/app
  working_dir: /app
  command: python src/rag_bridge.py
```

Then update Open WebUI to use the bridge:

```yaml
open-webui:
  environment:
    OLLAMA_BASE_URL: http://rag-bridge:5000 # Changed from ollama:11434
```

### Step 4: Restart and Test

Restart your services:

```shell
docker compose up
```

> [!NOTE]
> **Exit your container by pressing Ctrl+C on your keyboard.**

Test the bridge directly:

```shell
curl -X POST http://localhost:5000/api/generate \
  -H "Content-Type: application/json" \
  -d '{
    "model": "tinyllama:latest",
    "prompt": "What are the security policies?",
    "stream": false
  }'
```

### Step 5: Use Open WebUI with RAG

1. Open `http://localhost:8080` in your browser
2. Log in to Open WebUI
3. Select `tinyllama:latest` as your model
4. Ask questions about your documents:
   - "What are the hardware requirements?"
   - "How does RAG work?"
   - "What security practices are recommended?"

You'll see responses with source citations like `[source: 02-security-policy.md]`.

### How It Works

```text
Open WebUI → RAG Bridge → Ollama
              ↓
           ChromaDB
```

1. Your question goes to the RAG bridge
2. Bridge queries ChromaDB for relevant documents
3. Bridge adds document context to the prompt
4. Enhanced prompt goes to Ollama
5. Response comes back with source citations

### Key Benefits

- **Simple**: Just one Python file and docker-compose changes
- **Private**: All processing stays on your machine
- **Transparent**: Source citations show where answers come from
- **Flexible**: Easy to modify or extend

### Troubleshooting

**"No relevant information found"**

- Make sure documents are ingested: `docker compose run --rm python python src/embed.py`
- Check ChromaDB: `curl http://localhost:8000/api/v1/heartbeat`

**Open WebUI shows errors**

- Check bridge logs: `docker compose logs rag-bridge`
- Verify Ollama is running: `docker compose exec ollama ollama list`

**Slow responses**

- Reduce `n_results` in `query_chroma()` (default is 3)
- The first query might be slow while models load

---

## 7. Cleanup

**Goal:** Clean resources and discuss next steps.

First, stop any running containers by pressing **Ctrl+C** in the terminal where `docker compose up` is running.

Run the following command:

```shell
docker compose down -v
```

> [!NOTE]
> This stops all services, removes containers, networks, and volumes.

Run the following command:

```shell
docker compose ps -a
```

> [!NOTE]
> Even though they're not all running, we still have images sitting there doing nothing.

Run the following command:

```shell
docker compose images
```

> [!NOTE]
> We also have these images which are taking up resources on our machine.

Run the following command:

```shell
docker compose down --rmi all
```

> [!NOTE]
> This removes all images used by this project.

---

## What we accomplished

✅ **Ingested** documents and created text chunks  
✅ **Generated** local embeddings with nomic-embed-text  
✅ **Stored** vectors in ChromaDB  
✅ **Queried** the knowledge base with RAG  
✅ **Integrated** RAG with Open WebUI chat interface  
✅ **Built** complete privacy-preserving AI pipeline

---

## When to use RAG

| Use Case              | Recommended Approach |
| --------------------- | -------------------- |
| Company internal docs | Local RAG            |
| Customer support KB   | Local RAG            |
| Codebase search       | Local RAG            |
| General knowledge     | Standard Ollama      |
| Real-time data        | Cloud API            |

---

## 🎉 Congratulations

You've built a complete local RAG system! Your AI can now:

- Access your private documents
- Cite sources for its answers
- Work completely offline
- Keep all data on your machine

This is the foundation for enterprise-grade, privacy-preserving AI applications.
