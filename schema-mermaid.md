# Architecture - Book-By-Click

## Schema Global

```mermaid
flowchart TB
    subgraph CLIENT["Client"]
        A[("Navigateur")]
    end

    subgraph FRONTEND["Frontend - Docker"]
        B["React 19 / Vite\nPort 5173"]
    end

    subgraph BACKEND["Backend - Docker"]
        C["Flask API\nPort 5000"]
        S["Swagger UI\n/swagger"]
    end

    subgraph DOCKER["Docker Services"]
        D[("PostgreSQL 15\nPort 5433 → 5432")]
    end

    A -->|"HTTP"| B
    B -->|"REST API /api/*"| C
    C -->|"SQLAlchemy"| D
    C --- S

    style A fill:#6366f1,stroke:#4f46e5,color:#fff
    style B fill:#06b6d4,stroke:#0891b2,color:#fff
    style C fill:#8b5cf6,stroke:#7c3aed,color:#fff
    style S fill:#a78bfa,stroke:#7c3aed,color:#fff
    style D fill:#3b82f6,stroke:#2563eb,color:#fff

    style CLIENT fill:#1e1b4b,stroke:#4f46e5,color:#c7d2fe
    style FRONTEND fill:#164e63,stroke:#0891b2,color:#a5f3fc
    style BACKEND fill:#2e1065,stroke:#7c3aed,color:#ddd6fe
    style DOCKER fill:#1e3a5f,stroke:#2563eb,color:#bfdbfe
```

## Stack

**Frontend:** React 19, Vite (rolldown), React Router 7, TailwindCSS 4, Styled Components, Recharts, react-big-calendar, Lucide Icons, Heroicons

**Backend:** Flask, Flask-SQLAlchemy, Flask-CORS, Flask-Swagger-UI, python-dotenv

**Database:** PostgreSQL 15

**Services Docker:** PostgreSQL (5433), Backend Flask (5000), Frontend Vite (5173)

## Communication

API REST en JSON sur `http://localhost:5000/api/*`
