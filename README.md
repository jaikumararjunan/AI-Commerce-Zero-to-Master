# AI Commerce Control Center

A study-first, runnable reference architecture that combines:

- **Java 21 + Spring Boot 4.1.x** for deterministic commerce workflows.
- **Python + FastAPI** for ML, DL, GenAI, RAG, semantic search, recommendations, fraud/anomaly detection, forecasting, and agent workflows.
- **LangChain + LangGraph + Ollama** for local LLM/agent experiments.
- **OpenTelemetry + Prometheus + Grafana + MLflow** for application, ML, and agent observability.
- **Legacy and modern e-commerce patterns** side-by-side so you can understand the evolution from monolith/rules/batch systems to event-driven, ML-powered, multimodal, and agentic commerce.

## Mental model

```text
Web / Mobile / Voice / Agent
            |
            v
      Java Commerce API
 Catalog | Cart | Order | Promo
            |
   REST + domain events
            |
            v
      Python AI Service
 Search | RecSys | Fraud | Forecast
 RAG | GenAI | LangGraph Agent
            |
            v
 Data / Model / Vector / Event platforms
            |
            v
 OpenTelemetry -> Prometheus / Grafana / MLflow
```

## Fastest local study path (no Docker required)

1. Start Python AI service:
   ```powershell
   cd ai-python
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1
   pip install -r requirements.txt
   uvicorn app.main:app --reload --port 8000
   ```
2. Start Java commerce service in another terminal:
   ```powershell
   cd backend-java\commerce-core
   mvn spring-boot:run
   ```
3. Run smoke test:
   ```powershell
   .\scripts\smoke-test.ps1
   ```
4. Open:
   - Java health: http://localhost:8080/actuator/health
   - Python docs: http://localhost:8000/docs
   - Java Prometheus: http://localhost:8080/actuator/prometheus
   - Python Prometheus: http://localhost:8000/metrics

## Optional local LLM

```powershell
ollama pull llama3.2:3b
$env:OLLAMA_MODEL='llama3.2:3b'
$env:OLLAMA_BASE_URL='http://127.0.0.1:11434'
```

The `/ai/agent/ask` endpoint works with deterministic fallbacks even when Ollama is unavailable, but a running Ollama model gives the full GenAI experience.

## Documents

See `docs/` for the Word guides:

1. Architecture and Flow Guide
2. Developer Setup and Runbook
3. E-commerce AI Feature Catalog
4. Observability, Monitoring and Production Guide

## Project intent

This is an **educational reference implementation**, not a payment-certified production storefront. Sensitive production flows such as payment authorization, regulated pricing, refunds, account changes, and fraud blocking should use explicit authorization, policy controls, audit trails, and human review where appropriate.

## MCP + LLM + vector database learning layer

New study paths are included under `ai-python/app/llm`, `ai-python/app/vectorstores`, `ai-python/app/mcp`, and `ai-python/app/services/vector_rag.py`.

### 1. Vector math from scratch (no Docker, no LLM)

```powershell
cd ai-python
$env:VECTOR_BACKEND='memory'
$env:EMBEDDING_PROVIDER='hash'
python .\vector_from_scratch_demo.py
```

### 2. Local semantic embeddings with Ollama

```powershell
ollama pull all-minilm
$env:EMBEDDING_PROVIDER='ollama'
$env:OLLAMA_EMBED_MODEL='all-minilm'
$env:VECTOR_BACKEND='memory'
```

### 3. Qdrant / pgvector

```powershell
cd infra
docker compose -f docker-compose-data.yml up -d

# Qdrant
$env:VECTOR_BACKEND='qdrant'
$env:QDRANT_URL='http://127.0.0.1:6333'

# or pgvector
$env:VECTOR_BACKEND='pgvector'
$env:PGVECTOR_DSN='postgresql://commerce:commerce@127.0.0.1:5432/commerce'
```

FastAPI endpoints:
- `POST /ai/vector/index`
- `GET /ai/vector/search?q=return+policy`
- `GET /ai/rag/answer?q=Can+I+return+a+product?`
- `GET /ai/llm/raw?q=Explain+RAG`

### 4. Python MCP v2 server/client

```powershell
.\scripts\start-python-mcp.ps1
# second terminal
.\scripts\test-python-mcp.ps1
```

The Python MCP endpoint is `http://127.0.0.1:8100/mcp` and exposes product search plus policy retrieval/RAG tools.

### 5. Java MCP

`commerce-core` now exposes selected read-only catalog/order operations with Spring AI `@McpTool`. The MCP adapter delegates to the same Java domain services used by REST.

## Additional Word guides

5. `05_MCP_LLM_Vector_DB_RAG_From_Scratch.docx`
6. `06_AI_Skills_Learning_Curve_and_Roadmap.docx`
7. `07_Java_Python_MCP_Vector_RAG_Code_Walkthrough.docx`
8. `08_Payment_Gateway_Checkout_and_Financial_Flows.docx`
9. `09_Historical_Data_Lake_Warehouse_Lakehouse_and_Future_AI.docx`
10. `10_Amazon_Type_Ecommerce_Micro_Macro_and_AI_Native_Architecture.docx`
11. `11_Environment_CICD_SRE_Performance_and_Observability.docx`
12. `12_Learner_Tracks_Fresher_3_5_7_and_Principal.docx`
13. `13_Hands_On_Labs_Payments_Data_Marketing_and_Production.docx`
14. `14_Mock_Data_Generator_From_Ground_to_Architect.docx`

## Release 0.2 - payments, historical data, warehouse, marketing, environments and learner portal

### Payment lab (safe mock gateway)

Java now includes a provider-neutral payment port plus a mock gateway. The lab demonstrates payment-intent state, idempotent creation, authentication/action states, capture, refund and webhook de-duplication. It deliberately accepts only a provider `paymentMethodRef`; never put PAN/CVV into this training service.

Useful endpoints:
- `POST /api/payments/intents` with `Idempotency-Key`
- `POST /api/payments/{paymentId}/confirm`
- `POST /api/payments/{paymentId}/capture`
- `POST /api/payments/{paymentId}/refunds`
- `POST /api/payments/webhooks/mock`

### Historical lake + warehouse lab

```powershell
cd data-platform\pipelines
python .\run_pipeline.py
```

This creates a local Bronze -> Silver -> SQLite warehouse -> future-feature flow using only Python standard library. It is intentionally simple enough to inspect every record. The docs map it to production CDC/Kafka, object storage/Parquet/Iceberg, warehouse/lakehouse, governance and feature-store patterns.

### Marketing intelligence

FastAPI now exposes:
- `POST /ai/marketing/decision`
- `POST /ai/marketing/campaign-suggestions`

The deterministic learning implementation teaches segmentation, next-best-action, consent, frequency caps and experimentation before you replace the rules with propensity/uplift models.

### Environment promotion

See `environments/` and `infra/k8s/overlays/` for Dev -> QA -> UAT -> Pre-Prod -> Prod examples. The principle is one immutable artifact promoted through stages, with environment-specific configuration, secrets and scale.

### Parallel learner site

```powershell
cd learning-site
python -m http.server 9090
```

Open `http://localhost:9090`. It provides separate learning emphasis for Freshers, 3y, 5y, 7y and 7+ / architect audiences.

## Release 0.3 - cross-service mock data generator

A deterministic synthetic-data subsystem now lives in `mock-data/`.

```powershell
.\scripts\generate-mock-data.ps1 -Profile tiny -Scenario normal
```

It generates one consistent learning world across identity, catalog, pricing, cart, order, payment, inventory, fulfilment, shipping, returns, reviews, support, marketing, search, ML/DL, RAG/vector, agents, events and observability.

Profiles: `tiny`, `small`, `medium`, `large`, `xlarge`.

Failure/traffic scenarios: `normal`, `peak-sale`, `fraud-attack`, `inventory-shortage`, `payment-degradation`, `returns-spike`, `marketing-campaign`, `mixed-chaos`.

Start with:
- `mock-data/README.md`
- `mock-data/SCHEMA.md`
- `mock-data/output/latest/manifest.json`
- `mock-data/output/latest/dataset_stats.json`

To feed the historical-data lab:

```powershell
.\scripts\generate-mock-data.ps1 -Profile small -Scenario normal -SyncDataPlatform
.\scripts\run-data-platform.ps1
```

To exercise the running Java + Python APIs with generated customer IDs:

```powershell
.\scripts\exercise-mock-services.ps1 -Count 3
```

For real-time event/observability practice:

```powershell
cd mock-data
python .\stream_mock_events.py --interval 0.5
```

## Docker v1.2 port selection

Use automatic one-click launchers for zero questions, or interactive launchers when you want to select every host port:

- `START_LOCAL_DOCKER.bat` — automatic core ports
- `START_FULL_LOCAL_DOCKER.bat` — automatic full-stack ports
- `START_LOCAL_DOCKER_INTERACTIVE.bat` — manually approve/change core ports
- `START_FULL_LOCAL_DOCKER_INTERACTIVE.bat` — manually approve/change all full-stack ports
- `SHOW_SELECTED_PORTS.bat` — display `.env.local`

The launcher validates occupied and duplicate ports before Docker starts. Container-internal ports never change.

---

## v1.3 Docker startup fix

If an earlier build failed with `No qualifying bean of type RestClient.Builder`, v1.3 adds an explicit Java `HttpClientConfig` bean plus a regression test. See `FIX_NOTES_v1.3.md`.


## Customer-facing application

The Docker bundle now contains a separate real commerce UI under `commerce-app/`.

After Docker startup, use the **APPLICATION** URL printed by the launcher (preferred `http://localhost:9200`). This is different from the learning portal. Double-click `OPEN_APPLICATION.bat` at any time to open the actual selected application port from `.env.local`.

Application features: catalog, cart, mock checkout/payment, AI search, recommendations, AI copilot and service status. See `APPLICATION_GUIDE.md`.
"# AI-Commerce-Zero-to-Master" 
