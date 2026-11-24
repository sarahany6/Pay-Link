# Pay-Link
PayLink is a backend digital payment system. Customers and agents can pay utility bills (electricity, water, internet) and recharge mobile phones using a digital wallet.

**Project status:** Phase 1 (requirements, design and DevOps foundation).

## Who Uses It 
- **Customers:** pay bills and recharge phones from their wallet.
- **Agents:** make transactions on behalf of customers.
- **Billers:** the external utility and telecom providers that receive the payments.
The full list of stakeholders and user stories is in [`docs/requirements.md`](docs/requirements.md).
 
## Tech Stack
 
| Part | Technology |
|---|---|
| Backend | Python 3.12, FastAPI, Uvicorn |
| Database | PostgreSQL 16 |
| Containers | Docker, Docker Compose |
| CI | GitHub Actions |
| Lint and tests | flake8, pytest |

## Getting Started
### GitHub Codespaces 
 
1. On the repository page, click **Code → Codespaces → Create codespace on main**.
2. In the terminal, run:
```bash
docker compose up --build
```
 
3. Open the **Ports** tab and open port `8000`. Add `/health` to the URL.
### Verify
 
```bash
curl http://localhost:8000/health
```
 
Expected response:
 
```json
{"status": "ok"}
```
 
Stop the services:
 
```bash
docker compose down
```
 
## Services
 
| Service | Port | Description |
|---|---|---|
| `app` | 8000 | FastAPI application |
| `db` | 5432 | PostgreSQL database |
 
Default development database settings (not for production):
 
| Variable | Value |
|---|---|
| `POSTGRES_USER` | `paylink` |
| `POSTGRES_PASSWORD` | `paylink` |
| `POSTGRES_DB` | `paylink` |
 
## API
 
| Method | Path | Description |
|---|---|---|
| GET | `/health` | Health check, returns `{"status": "ok"}` |
 
FastAPI also serves interactive API docs at `http://localhost:8000/docs`.

 ### Continuous Integration
 
On every push and pull request, GitHub Actions runs:
1. **Lint** with flake8
2. **Test** with pytest
3. **Build** the Docker image, start the stack with Docker Compose, and check `/health`


## Project Structure
 
```
Pay-Link/
├── .github/workflows/
│   └── ci.yml                 # CI pipeline
├── app/                       # FastAPI application
│   └── main.py
├── tests/                     # Automated tests
│   └── test_health.py
├── docs/
│   ├── requirements.md        # Requirements Document
│   └── design/                # Diagram source files (PlantUML / Mermaid)
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
└── requirements-dev.txt
```

## Documentation
 
- **Requirements:** [`docs/requirements.md`](docs/requirements.md)
- **Design diagrams** in [`docs/design/`](docs/design/):
  - Use Case
  - Domain Class
  - Sequence: Pay a Bill
  - Sequence: Refund a Failed Payment
  - Payment State
  - ER
  
## Team
 
| Name | Role |
|---|---|
|Reem Hussein|Requirements & Use Case|
|Malak Ahmed|Requirements & Documentation|
|Rahma Elkholy|Design Diagrams|
|Sara Hany|Payment State & DevOps|
