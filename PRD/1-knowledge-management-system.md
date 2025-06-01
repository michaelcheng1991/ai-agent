# Knowledge Management System TDD

## Key Components

- **Document Storage Service**: Handles file uploads and storage of various document formats
- **Embedding Service**: Processes documents and generates vector embeddings for semantic search
- **Vector Database**: Stores document embeddings for efficient semantic retrieval
- **Knowledge API**: Exposes endpoints for CRUD operations on knowledge documents

## Technical Flow Diagram

```mermaid
sequenceDiagram
    participant A as Admin/Instructor
    participant UI as Web UI
    participant API as Knowledge API
    participant DS as Document Storage
    participant ES as Embedding Service
    participant VDB as Vector Database
    
    A->>UI: Upload document(PDF/DOCX/TXT)
    UI->>API: POST /api/knowledge
    API->>DS: Store original document
    DS-->>API: Document stored, return ID
    API->>ES: Process document for embedding
    ES->>ES: Extract text & generate embeddings
    ES->>VDB: Store document embeddings
    VDB-->>ES: Confirm storage
    ES-->>API: Embedding complete
    API-->>UI: Document processed successfully
```

## User Flow Diagram

```mermaid
graph TD
    A[Admin logs in] --> B[Access Knowledge Management]
    B --> C[Upload new document]
    B --> D[Browse existing documents]
    
    C --> G[Preview processed content]
    
    D --> K[Open document details]
    
    K --> M[Replace document]
    K --> N[Delete document]
``` 