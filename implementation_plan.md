# AURA Implementation Plan

This document outlines the technical implementation plan for building the MVP (Minimum Viable Product) of the **Adaptive Unified Routing Architecture (AURA)**.

## Goal Description

Build an intelligent LLM routing engine that minimizes costs by preferencing local inference while ensuring response quality through automated escalation. The MVP will consist of a Python/FastAPI backend and a simple React dashboard to visualize routing metrics.

> [!IMPORTANT]
> ## User Review Required
> Please review the open questions below before we begin coding. The choices here will significantly impact our velocity during the hackathon.

> [!WARNING]
> ## Open Questions
> 1. **Database:** Should we use **SQLite** instead of PostgreSQL for this MVP? SQLite requires zero setup for the hackathon judges, which makes the demo much easier to run locally.
> 2. **Local Model:** Which local model do you plan to use via Ollama? (e.g., `llama3:8b`, `gemma2:9b`, or `phi3:mini`?) 
> 3. **Prioritization:** Should we focus completely on building the **Backend API and Routing Engine** first, and tackle the Frontend Dashboard later, or do you want scaffolding for both simultaneously?

## Proposed Changes

We will create a structured backend and frontend application. The backend will be the core of the routing engine.

### Backend Structure (FastAPI)

#### [NEW] `backend/requirements.txt`
Dependencies: `fastapi`, `uvicorn`, `litellm`, `sqlalchemy`, `pydantic`.

#### [NEW] `backend/main.py`
The FastAPI application entry point. Will expose endpoints:
- `POST /route` - Main endpoint to handle user queries.
- `GET /metrics` - Endpoint to fetch dashboard stats.

#### [NEW] `backend/router/engine.py`
Core routing logic implementing the Multi-Dimensional Routing and Quality-Gated Escalation:
- Initial complexity check.
- Invocation of the local model.
- Invocation of the API models (Cheap -> Powerful) if escalation is needed.

#### [NEW] `backend/evaluators/quality.py`
Implements "LLM-as-a-judge" logic. Evaluates the local/cheap model's response against the user prompt to generate a quality score (0.0 to 1.0).

#### [NEW] `backend/models/database.py`
SQLAlchemy models for tracking metrics (Token usage, Latency, Cost, Escalation status) to populate the dashboard.

### Configuration & Infrastructure

#### [NEW] `docker-compose.yml`
(Optional, if requested) To spin up PostgreSQL (if not using SQLite), Redis, and the backend.

#### [NEW] `.env.example`
Template for required API keys (e.g., `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `OLLAMA_HOST`).

### Frontend Structure (React/Vite)

*(To be implemented after the backend is stable)*
#### [NEW] `frontend/`
A lightweight React app (using Vite and Recharts) to poll the `/metrics` endpoint and display cost savings and routing distribution.

## Verification Plan

### Automated Tests
- Unit tests for the routing engine (`pytest backend/tests/test_router.py`).
- Mock the LiteLLM/Ollama responses to test escalation logic without spending API credits.

### Manual Verification
- Start `ollama` locally.
- Run `uvicorn backend.main:app`.
- Send simple queries via cURL/Postman and observe them being handled by the local model.
- Send complex/reasoning queries and observe the system rejecting the local output and escalating to the powerful API.
- Verify that the `/metrics` endpoint accurately records the fallback.
