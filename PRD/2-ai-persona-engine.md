# AI Persona Engine TDD

## Goal
The AI Persona Engine is designed to simulate client status and goals, such that their problems and desires are exposed.

## Key Components

- **Persona Generator**: Creates realistic financial customer personas using LLM with Consultation Data and Knowledge Base
- **Persona Configuration Manager**: UI for customizing persona attributes and financial scenarios
- **Financial Scenario Generator**: Creates realistic financial situations for personas
- **Persona Database**: Stores created personas for reuse in training sessions
- **Persona API**: Exposes endpoints for CRUD operations on personas

## Technical Flow Diagram

```mermaid
sequenceDiagram
    participant I as Instructor
    participant UI as Web UI
    participant API as Persona API
    participant LLM as LLM Service
    participant DB as Persona Database
    
    I->>UI: Access persona creation
    UI->>API: GET /api/personas/templates
    API-->>UI: Return persona templates
    
    I->>UI: Configure persona attributes
    I->>UI: Set financial situation
    UI->>API: POST /api/personas
    API->>LLM: Generate persona
    LLM-->>API: Return generated persona
    API->>DB: Store persona
    DB-->>API: Confirm storage
    API-->>UI: Persona created
    
    I->>UI: Edit existing persona
    UI->>API: GET /api/personas/:id
    API->>DB: Retrieve persona
    DB-->>API: Return persona data
    API-->>UI: Display persona for editing
    I->>UI: Modify persona attributes
    UI->>API: PUT /api/personas/:id
    API->>DB: Update persona
    DB-->>API: Confirm update
    API-->>UI: Persona updated
```

## User Flow Diagram

```mermaid
graph TD
    A[Instructor logs in] --> B[Access Persona Management]
    B --> C[Browse existing personas]
    B --> D[Create new persona]
    
    C --> E[Select persona to edit]
    C --> F[Select persona for simulation]
    
    D --> G[Start from template]
    D --> H[Start from scratch]
    
    G --> I[Configure persona attributes]
    H --> I
    
    I --> J[Set demographic details]
    I --> K[Define financial situation]
    
    J --> M[Preview persona]
    K --> M
    
    M --> N[Save persona]
    
    E --> I
    F --> P[Launch training simulation]
```