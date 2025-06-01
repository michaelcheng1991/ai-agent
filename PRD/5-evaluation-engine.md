# Evaluation Engine TDD

## Key Components

- **Conversation Analyzer**: Processes conversation transcripts for analysis
- **Knowledge Application Checker**: Verifies if trainee applied relevant knowledge
- **Solution Appropriateness Evaluator**: Assesses if recommended solutions match client needs
- **Scoring Engine**: Calculates scores based on defined evaluation criteria
- **Feedback Generator**: Creates detailed, actionable feedback for trainees
- **Evaluation API**: Exposes endpoints for evaluation requests and results

## Technical Flow Diagram

```mermaid
sequenceDiagram
    participant S as Session End
    participant API as Evaluation API
    participant CA as Conversation Analyzer
    participant KA as Knowledge Application
    participant SA as Solution Appropriateness
    participant LLM as LLM Service
    participant KDB as Knowledge Database
    participant DB as Evaluation Database
    
    S->>API: POST /api/evaluations
    API->>CA: Process conversation transcript
    CA->>CA: Extract key discussion points
    CA-->>API: Return conversation analysis
    
    API->>KDB: Retrieve relevant knowledge
    KDB-->>API: Return knowledge references
    
    API->>KA: Assess knowledge application
    KA->>KA: Compare trainee statements with knowledge
    KA-->>API: Return knowledge score
    
    API->>SA: Assess solution appropriateness
    SA->>SA: Compare solutions to client needs
    SA-->>API: Return solution score
    
    API->>LLM: Generate detailed feedback
    LLM->>LLM: Analyze strengths and weaknesses
    LLM-->>API: Return feedback text
    
    API->>DB: Store evaluation results
    DB-->>API: Confirm storage
    API-->>S: Return complete evaluation
```

## User Flow Diagram

```mermaid
graph TD
    A[Session ends] --> B[Automatic evaluation begins]
    B --> C[System analyzes conversation]
    C --> D[System checks knowledge application]
    C --> E[System assesses solution appropriateness]
    
    D --> F[Generate knowledge accuracy score]
    E --> G[Generate solution fit score]
    
    F --> H[Calculate overall performance score]
    G --> H
    
    H --> I[Generate detailed feedback]
    I --> J[Identify improvement areas]
    I --> K[Highlight successful approaches]
    
    J --> L[Create action items for trainee]
    
    L --> M[Trainee reviews evaluation]
    K --> M
    
    M --> N[Instructor reviews evaluation]
    N --> O[Instructor adds comments]
    
    M --> P[Trainee views progress over time]
``` 