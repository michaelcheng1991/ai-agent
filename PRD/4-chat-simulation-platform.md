# Chat Simulation Platform TDD

## Key Components

- **Session Manager**: Handles creation and management of simulation sessions
- **Audio Interface**: Provides speech-to-text and text-to-speech capabilities
- **Chat Service**: Manages real-time messaging between trainee and AI persona
- **Conversation Logger**: Records and stores conversation history
- **Scenario Manager**: Controls progression of financial scenarios during simulation
- **Session API**: Exposes endpoints for managing simulation sessions

## Technical Flow Diagram

```mermaid
sequenceDiagram
    participant T as Trainee
    participant UI as Web UI
    participant API as Session API
    participant LLM as Persona LLM
    participant STT as Speech-to-Text
    participant TTS as Text-to-Speech
    participant DB as Session Database
    
    T->>UI: Start training session
    UI->>API: POST /api/sessions/start
    API->>DB: Create new session
    DB-->>API: Return session ID
    API-->>UI: Session started
    
    T->>UI: Speak into microphone
    UI->>STT: Process audio
    STT-->>UI: Return transcribed text
    UI->>API: POST /api/sessions/:id/message
    API->>LLM: Process trainee message
    LLM->>LLM: Generate persona response
    LLM-->>API: Return response
    API->>DB: Log conversation
    DB-->>API: Confirm logging
    API-->>UI: Return persona response
    UI->>TTS: Convert text to speech
    TTS-->>UI: Return audio
    UI-->>T: Play persona response audio
    
    T->>UI: End session
    UI->>API: POST /api/sessions/:id/end
    API->>DB: Mark session complete
    DB-->>API: Confirm completion
    API-->>UI: Session ended, redirect to evaluation
```

## User Flow Diagram

```mermaid
graph TD
    A[Trainee logs in] --> B[Select training scenario]
    B --> C[Choose AI persona]
    B --> D[Continue saved session]
    
    C --> E[Review scenario details]
    E --> F[Begin simulation]
    
    F --> G[Engage in voice conversation]
    G --> H[Take session notes]
    G --> I[See conversation history]
    
    G --> J[End simulation]
    
    J --> K[Review conversation history]
    J --> L[Wait for evaluation results]
    
    L --> M[View detailed feedback]
    M --> N[Identify improvement areas]
    
    D --> F
``` 