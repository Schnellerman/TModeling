```mermaid
graph TD
    %% Определение доверительных зон через подграфы
    
    subgraph ZONE_0["[ TRUST ZONE 0: UNTRUSTED / PUBLIC ]"]
        style ZONE_0 fill:#ffcccc,stroke:#333,stroke-width:2px
        User(("👤 User Workstation <br> (Browser / SPA React)"))
    end

    subgraph ZONE_1["[ TRUST ZONE 1: DMZ / EDGE ]"]
        style ZONE_1 fill:#fff2cc,stroke:#333,stroke-width:2px
        Nginx["🌐 Nginx Inverse Proxy <br> (SSL Termination / Rate Limit)"]
    end

    subgraph ZONE_2["[ TRUST ZONE 2: SECURE APPLICATION CONTOUR ]"]
        style ZONE_2 fill:#d9ead3,stroke:#333,stroke-width:2px
        GoBackend["🐹 Backend (Go / Golang) <br> (Business Logic & Auth)"]
        AppDB[("💾 Application DB <br> (PostgreSQL / MySQL)")]
        
        %% Компоненты AI, внедряемые в контур
        GoOrchestrator["🤖 AI Orchestrator (Go Module/Service) <br> (Prompt Eng & Guardrails)"]
        VectorDB[("🗄️ Vector DB <br> (Qdrant / Pinecone)")]
    end

    subgraph ZONE_3["[ TRUST ZONE 3: EXTERNAL AI PROVIDER ]"]
        style ZONE_3 fill:#f4cccc,stroke:#333,stroke-width:2px     
        ExternalLLM["🧠 External LLM API <br> (OpenAI Azure / Anthropic)"]
    end

    %% Потоки данных и связи (Data Flow)
    User -- "1. HTTPS (TLS 1.3)" --> Nginx
    Nginx -- "2. HTTP (Internal Net)" --> GoBackend
    GoBackend -- "3. SQL Queries" --> AppDB
    
    %% Интеграция AI
    GoBackend -- "4. Internal Call" --> GoOrchestrator
    GoOrchestrator -- "5. gRPC / Semantic Search" --> VectorDB
    GoOrchestrator -- "6. HTTPS + API Key (Outbound)" --> ExternalLLM
    GoOrchestrator -- "7. Tool Execution (Verify Identity)" --> GoBackend
```
