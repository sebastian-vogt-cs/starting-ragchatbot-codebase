# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Starting the Application
```bash
# Quick start (recommended)
chmod +x run.sh
./run.sh

# Manual start
cd backend && uv run uvicorn app:app --reload --port 8000
```

### Package Management
```bash
# Install dependencies
uv sync

# The project uses uv for Python package management, not pip
```

### Environment Setup
- Create `.env` file in root with `ANTHROPIC_API_KEY=your_key_here`
- Requires Python 3.13+
- Uses ChromaDB for vector storage at `./backend/chroma_db/`

## Architecture Overview

### High-Level Structure
This is a full-stack RAG (Retrieval-Augmented Generation) system with:
- **Backend**: FastAPI server with RAG orchestration (`backend/`)
- **Frontend**: Static HTML/CSS/JS served by FastAPI (`frontend/`)
- **Document Storage**: Course materials in `docs/` folder (txt files)
- **Vector Database**: ChromaDB for semantic search

### Core Components (`backend/`)

**RAG System Flow**: `app.py` → `rag_system.py` → `ai_generator.py` + `vector_store.py`

1. **`rag_system.py`**: Main orchestrator that coordinates all components
2. **`vector_store.py`**: ChromaDB wrapper with two collections:
   - Course metadata (titles, instructors, lessons)  
   - Course content chunks (text snippets with metadata)
3. **`ai_generator.py`**: Anthropic Claude integration with tool calling
4. **`search_tools.py`**: Tool-based search system for Claude to use
5. **`document_processor.py`**: Processes course documents into structured data
6. **`session_manager.py`**: Manages conversation history per session
7. **`models.py`**: Pydantic models for Course, Lesson, CourseChunk

### Key Design Patterns

**Tool-Based Architecture**: The AI uses function calling with `CourseSearchTool` to search the vector database, rather than direct RAG retrieval in the main flow.

**Dual Vector Collections**: 
- Course metadata for high-level queries about courses/instructors
- Content chunks for detailed lesson-specific questions

**Session Management**: Tracks conversation history with configurable memory limit (`MAX_HISTORY=2` exchanges).

### Configuration (`config.py`)
- Uses dataclass with environment variable loading
- Key settings: `CHUNK_SIZE=800`, `CHUNK_OVERLAP=100`, `MAX_RESULTS=5`
- ChromaDB path: `./chroma_db`
- Embedding model: `all-MiniLM-L6-v2`

### API Endpoints
- `POST /api/query`: Main query endpoint with session support
- `GET /api/courses`: Course statistics and catalog info
- Static file serving at `/` for frontend

### Document Processing
- Supports .pdf, .docx, .txt files in `docs/` folder
- Extracts course title, instructor, lessons with automatic parsing
- Creates text chunks with course/lesson metadata for vector storage
- Avoids reprocessing existing courses on startup

### Frontend Architecture
- Single-page application with vanilla JavaScript
- Real-time course statistics display
- Markdown rendering support for AI responses
- Session-based conversation flow
- always use uv to run the server do not use pip directly
- make sure to use uv to manage all dependencies
- use uv to run Python files