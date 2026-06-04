# DocMind AI: Agentic RAG Document Intelligence System

DocMind AI is a Retrieval-Augmented Generation (RAG) system designed to perform grounded, multi-turn question answering over document repositories. The system integrates semantic document parsing, local vector databases, conversational state management, and multi-agent coordination loops to deliver verified answers while minimizing large language model (LLM) hallucinations.

---

## Key Capabilities

* **Agentic Query Refinement**: Utilizes an LLM agent to preprocess and rewrite conversational user queries into factual search queries, improving the recall of relevant vector chunks.
* **Semantic Document Search**: Leverages high-performance similarity search over dense vector embeddings using Facebook AI Similarity Search (FAISS).
* **Self-Corrective Output Validation**: Incorporates an optional post-generation validation agent to cross-verify LLM answers against source materials and suppress responses containing ungrounded assertions.
* **Persistent Index Caching**: Caches the generated vector index and chunk map locally to disk, preventing redundant API calls and decreasing initialization times on subsequent boot cycles.
* **Context-Aware Dialogue Memory**: Maintains conversational history across turns, allowing the system to resolve follow-up inquiries that depend on prior interactions.
* **Containerized Deployment**: Includes Docker configurations for immediate containerized deployment with multi-volume bindings for index persistence.

---

## Technical Architecture

The system processes questions through a structured, multi-stage retrieval pipeline:

1. **Ingestion**: The system parses PDF documentation from the target source folder using programmatic extraction libraries.
2. **Chunking**: Text is split into overlapping slices of 400 characters (with a 50-character overlap) to preserve sentence transitions and context across boundaries.
3. **Embedding Generation**: The system calls the Google Gemini Embedding API to generate vector embeddings of the chunked text, utilizing distinct task-specific types (`RETRIEVAL_DOCUMENT` for index blocks and `RETRIEVAL_QUERY` for search queries).
4. **Vector Database Indexing**: The embeddings are loaded into a local FAISS Euclidean distance ($L_2$) index. The compiled index and document chunks are cached to the disk using Python serialization.
5. **Retrieval Coordination**: The rewritten query is used to search the FAISS index for the top matching context blocks.
6. **Prompt Assembly**: The retrieved chunks are formatted alongside a sliding memory window containing the last 3 dialogue turns, injecting strict grounding constraints.
7. **Grounded Generation**: The Gemini Flash model generates the final response. If the source material does not contain the answer, the model is instructed to explicitly report a lack of document-grounded context.
8. **Optional Verification**: A verification agent examines the draft response against the retrieved source context to ensure absolute alignment, overriding any ungrounded response.

---

## Tech Stack

* **Core Language**: Python 3.10+
* **LLM Engine**: Google Gemini (via google-genai SDK)
* **Vector Store**: FAISS (Facebook AI Similarity Search)
* **Text Extraction**: PyPDF
* **Web Server**: Flask, Gunicorn (production WSGI server)
* **Infrastructure**: Docker, Docker Compose, Vercel

---

## Installation and Setup

### Prerequisites
* Python 3.10 or higher
* Docker and Docker Compose (optional, for containerized deployments)
* A Google Gemini API Key

### Configuration
Create a `.env` file in the root of the project and populate it with your API key:

```env
GEMINI_API_KEY=your_gemini_api_key_here
```

---

### Local Installation

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/lumen-byte/DocMind-AI-Agentic-RAG-Document-Intelligence-System.git
   cd DocMind-AI-Agentic-RAG-Document-Intelligence-System
   ```

2. **Establish virtual environment**:
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows, use: venv\Scripts\activate
   ```

3. **Install Dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

4. **Run the Application**:
   ```bash
   python app.py
   ```
   The Flask server runs on port 5001. Open `http://localhost:5001` in your browser.

---

### Containerized Deployment (Docker)

To run the application inside an isolated Docker container with volume mapping for persistent document uploads and cache indexing:

1. **Build and start the container**:
   ```bash
   docker-compose up --build
   ```

2. **Access the Application**:
   Open `http://localhost:5001` in your browser.

3. **Container Architecture**:
   * The container binds `./data` from the host to persist original files.
   * Internal volumes are mounted for `/tmp/docs` and `/tmp/cache` to ensure document indexing and vector stores persist across container reboots.
