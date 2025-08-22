# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Environment Setup
```bash
# Install uv package manager (if not installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies
uv sync

# Set up environment variables
cp .env.example .env
# Edit .env with your ANTHROPIC_API_KEY
```

### Running the Application
```bash
# Quick start (recommended)
chmod +x run.sh
./run.sh

# Manual start
cd backend
uv run uvicorn app:app --reload --port 8000

# With increased timeout for large dependencies
export UV_HTTP_TIMEOUT=300
uv run uvicorn app:app --host 0.0.0.0 --port 8000 --reload
```

### Application Access
- Web Interface: `http://localhost:8000`
- API Documentation: `http://localhost:8000/docs`
- For WSL users: Also try `http://127.0.0.1:8000` or the WSL IP address

## Architecture Overview

This is a **Retrieval-Augmented Generation (RAG) system** for querying course materials. The architecture follows a tool-based approach where Claude AI decides when to search the knowledge base.

### Core Components

**RAG System (`rag_system.py`)** - Main orchestrator that coordinates all components:
- Initializes document processor, vector store, AI generator, session manager
- Manages the query flow from user input to AI response
- Handles tool registration and execution

**Document Processing Pipeline**:
- `document_processor.py` - Parses structured course documents, extracts metadata, chunks content
- `models.py` - Defines Course, Lesson, and CourseChunk data structures
- Course documents follow format: Title → Link → Instructor → Lessons with content

**Vector Storage (`vector_store.py`)**:
- Uses ChromaDB with sentence transformers for embeddings
- Two collections: `course_catalog` (metadata) and `course_content` (chunks)
- Supports semantic search with course/lesson filtering
- Embedding model: "all-MiniLM-L6-v2"

**AI Integration (`ai_generator.py`)**:
- Anthropic Claude API integration with tool support
- System prompt optimized for educational content
- Model: "claude-sonnet-4-20250514"
- Temperature: 0 for consistent responses

**Tool System (`search_tools.py`)**:
- `CourseSearchTool` - Semantic search with smart course name matching
- `ToolManager` - Registers and executes tools for Claude
- Claude autonomously decides when to search vs. answer from knowledge

**Session Management (`session_manager.py`)**:
- Maintains conversation history (configurable max history)
- Provides context for follow-up questions

### Key Configuration (`config.py`)
- `CHUNK_SIZE`: 800 characters per text chunk
- `CHUNK_OVERLAP`: 100 characters overlap between chunks  
- `MAX_RESULTS`: 5 search results returned
- `MAX_HISTORY`: 2 conversation messages remembered
- `CHROMA_PATH`: "./chroma_db" for vector storage

### Document Format
Course files in `docs/` should follow this structure:
```
Course Title: [title]
Course Link: [url]
Course Instructor: [instructor]

Lesson 0: [lesson title]
Lesson Link: [lesson url]
[lesson content...]

Lesson 1: [lesson title]
[lesson content...]
```

### Query Flow
1. Frontend sends query via POST `/api/query`
2. RAG system gets conversation history and creates prompt
3. Claude API called with search tools available
4. Claude decides whether to search course content
5. If search needed: CourseSearchTool queries vector store
6. Claude synthesizes results into natural language response
7. Response returned with sources and session management

### Frontend Structure
- `frontend/index.html` - Single-page web interface
- `frontend/script.js` - Handles chat functionality and API calls
- `frontend/style.css` - UI styling
- Uses marked.js for markdown rendering in responses

## Common Issues

### Dependency Installation
- Large ML dependencies (PyTorch ~783MB, CUDA libraries ~2GB) may timeout
- Use `export UV_HTTP_TIMEOUT=300` to increase timeout
- First install can take 10+ minutes due to package sizes

### WSL/Windows Users
- Use Git Bash for running shell commands
- If localhost doesn't work, try `127.0.0.1:8000` or WSL IP address
- Check Windows firewall if connections fail

### Environment Variables
- Must have valid `ANTHROPIC_API_KEY` in `.env` file
- Application will fail to start without proper API key
- Use `.env.example` as template

## Important Notes

- Course documents are automatically loaded from `docs/` on startup
- ChromaDB persists vector embeddings between runs
- Session IDs maintain conversation context
- Tool-based architecture allows Claude to search only when needed
- System optimized for educational content and course Q&A