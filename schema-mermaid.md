# Architecture - Book-By-Click

## Schema Global

```mermaid
flowchart TB
    subgraph CLIENT["Client"]
        A[("Navigateur")]
    end

    subgraph FRONTEND["Frontend"]
        B["React App - Port 5173"]
    end

    subgraph BACKEND["Backend"]
        C["Flask API - Port 5000"]
    end

    subgraph DOCKER["Docker Services"]
        D[("PostgreSQL 15 - Port 5432")]
        E["Mail Server - Port 25"]
    end

    A -->|"HTTP"| B
    B -->|"REST API"| C
    C -->|"SQL"| D
    C -->|"SMTP"| E

    style A fill:#6366f1,stroke:#4f46e5,color:#fff
    style B fill:#06b6d4,stroke:#0891b2,color:#fff
    style C fill:#8b5cf6,stroke:#7c3aed,color:#fff
    style D fill:#3b82f6,stroke:#2563eb,color:#fff
    style E fill:#10b981,stroke:#059669,color:#fff

    style CLIENT fill:#1e1b4b,stroke:#4f46e5,color:#c7d2fe
    style FRONTEND fill:#164e63,stroke:#0891b2,color:#a5f3fc
    style BACKEND fill:#2e1065,stroke:#7c3aed,color:#ddd6fe
    style DOCKER fill:#1e3a5f,stroke:#2563eb,color:#bfdbfe
```

## Stack

**Frontend:** React 19, Vite, React Router, TailwindCSS, Recharts

**Backend:** Flask, SQLAlchemy, Flask-CORS

**Database:** PostgreSQL 15

**Services Docker:** PostgreSQL (5432), Mail Server (25)

## Communication

API REST en JSON sur `http://localhost:5000`
