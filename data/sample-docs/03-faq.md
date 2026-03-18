# Frequently Asked Questions

## General

### What are the hardware requirements?

For this workshop, you'll need:

- Laptop with 8GB+ RAM (16GB recommended)
- 10GB free disk space
- Docker Desktop installed and running

### Do I need API keys?

No! This workshop uses completely local, open-source tools. No external API keys are required.

### What if my laptop isn't powerful enough?

If your laptop can't run local models, you can connect to the instructor's instance. The instructor will provide their local network IP address during the workshop.

## Technical

### What's RAG?

RAG stands for Retrieval-Augmented Generation. It's a technique where an AI model can access and cite your own documents when answering questions.

### Why use local embeddings?

Local embeddings keep your data private. Instead of sending documents to OpenAI or other services, you generate embeddings on your own machine.

### What's ChromaDB?

ChromaDB is an open-source vector database designed for AI applications. It stores document embeddings and enables fast similarity search.

### What's nomic-embed-text?

It's an open-source embedding model that runs locally via Ollama. It converts text into mathematical vectors that capture semantic meaning.

## After the Workshop

### How do I continue learning?

- Try different embedding models
- Experiment with different chunking strategies
- Add more documents to your knowledge base
- Explore integrating with other AI tools

### Where can I get help?

- Join our community channels
- Check the workshop repository for updates
- Ask questions during Q&A

### Can I use my own documents?

Absolutely! Just place your Markdown files in the `data/sample-docs` folder and re-run the ingestion script.
