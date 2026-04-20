# Constitutional Decision Plane (CDP) Prototype

A runnable prototype for an **agentic AI decision control plane** with constitutional-style checks and balances:
**proposal → challenge → adjudication → legitimacy → execution → record**.

This starter repo gives you:

- **FastAPI** service for decision lifecycle APIs
- **PostgreSQL** schema for proposals, challenges, adjudications, legitimacy, executions, and event history
- **Containerized local runtime** via Docker Compose or Podman Compose
- **Terraform scaffold** for AWS deployment (VPC, Postgres, ECS/Fargate)
- **Seed data + smoke tests**
- **Simple adjudication worker** you can replace with actual model/policy logic

## Architecture

- `api`: HTTP control plane
- `worker`: background adjudication loop
- `postgres`: system of record
- `terraform/`: cloud scaffold for productionization

## Quick start

### 1) Local runtime
```bash
cp .env.example .env
docker compose up --build
```

Or with Podman:
```bash
cp .env.example .env
podman-compose up --build
```

### 2) Initialize database
The database auto-runs `sql/001_schema.sql` on first boot.

### 3) Open the API
- API docs: `http://localhost:8000/docs`
- Health: `http://localhost:8000/health`

### 4) Try the flow
```bash
curl -X POST http://localhost:8000/proposals \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Suspend claim for fraud review",
    "domain": "cms",
    "proposer": "rules-engine",
    "policy_basis": "high_risk_score > 0.92",
    "payload": {"claim_id": "CLM-1001", "risk_score": 0.96}
  }'
```

Then challenge it:
```bash
curl -X POST http://localhost:8000/challenges \
  -H "Content-Type: application/json" \
  -d '{
    "proposal_id": 1,
    "challenger": "appeals-bot",
    "challenge_type": "fairness",
    "reason": "Sparse provider history may bias the score",
    "evidence": {"provider_tenure_days": 9}
  }'
```

Adjudicate:
```bash
curl -X POST http://localhost:8000/adjudications \
  -H "Content-Type: application/json" \
  -d '{
    "proposal_id": 1,
    "adjudicator": "judge-service",
    "decision": "modify",
    "rationale": "Flag for manual review instead of automatic suspension",
    "conditions": {"route_to": "human_review"}
  }'
```

Legitimize:
```bash
curl -X POST http://localhost:8000/legitimations \
  -H "Content-Type: application/json" \
  -d '{
    "proposal_id": 1,
    "legitimizer": "policy-gate",
    "status": "approved",
    "basis": "Due process and challenge window satisfied"
  }'
```

Execute:
```bash
curl -X POST http://localhost:8000/executions \
  -H "Content-Type: application/json" \
  -d '{
    "proposal_id": 1,
    "executor": "orchestrator",
    "execution_target": "human_review_queue",
    "execution_payload": {"claim_id": "CLM-1001"}
  }'
```

## Core design ideas

- **Proposal is not authority**
- **Challenge is first-class**
- **Adjudication resolves conflict**
- **Legitimacy gates execution**
- **Everything emits events**

## Local commands

```bash
make up
make down
make logs
make test
make seed
```

## Repo layout

```text
src/app/          FastAPI app
src/worker/       Simple worker loop
sql/              Postgres schema and seed
infra/terraform/  AWS deployment scaffold
scripts/          Local helper scripts
tests/            Smoke tests
```

## Notes

This is intentionally a **starter control plane**, not a finished product. The policy engine, authn/authz, signatures, attestation, and model routing are left cleanly separable so you can evolve them without ripping the core apart.
