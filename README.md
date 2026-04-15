# 📦 Module 12 — Calculator App

A FastAPI-based calculator application with PostgreSQL persistence, JWT authentication, Docker containerization, and a full CI/CD pipeline. This module builds on the polymorphic SQLAlchemy models and Pydantic schemas from Module 11 by adding real database routes, authentication, and production-ready deployment.

---

## Table of Contents

1. [What Was Built](#1-what-was-built)
2. [Project Structure Changes](#2-project-structure-changes)
3. [How to Run Locally](#3-how-to-run-locally)
4. [Running the Tests](#4-running-the-tests)
5. [API Endpoints](#5-api-endpoints)
6. [How the Calculation Model Works](#6-how-the-calculation-model-works)
7. [How the Pydantic Schemas Work](#7-how-the-pydantic-schemas-work)
8. [Factory Pattern Explained](#8-factory-pattern-explained)
9. [CI/CD Pipeline](#9-cicd-pipeline)
10. [Docker Hub](#10-docker-hub)

---

## 1. What Was Built

Module 12 extends Module 11 by wiring up the existing polymorphic models and Pydantic schemas to a fully working FastAPI application with:

- **FastAPI routes** for full CRUD operations on calculations
- **PostgreSQL integration** via SQLAlchemy ORM (replacing the in-memory approach)
- **JWT Authentication** with access tokens and refresh tokens
- **Bcrypt password hashing** for secure user credentials
- **Docker + Docker Compose** for containerized local development
- **pgAdmin** for a visual database management interface
- **GitHub Actions CI/CD** for automated testing and Docker Hub deployment
- **Health check endpoint** for container orchestration readiness
- **Separate test database** (`fastapi_test_db`) isolated from the main database
- **Code coverage reporting** via pytest-cov

---

## 2. Project Structure Changes

```
module12-calculator-app/
├── .github/
│   └── workflows/
│       └── ci.yml              # GitHub Actions CI/CD pipeline
├── .vscode/                    # VS Code editor settings
├── app/
│   ├── core/
│   │   └── config.py           # Pydantic settings (env vars)
│   ├── models/
│   │   ├── user.py             # User SQLAlchemy model
│   │   └── calculation.py      # Polymorphic calculation models
│   ├── schemas/
│   │   └── calculation.py      # Pydantic request/response schemas
│   ├── routers/                # FastAPI route handlers
│   ├── database.py             # SQLAlchemy engine & session
│   ├── database_init.py        # DB table creation on startup
│   └── main.py                 # FastAPI app entry point
├── templates/                  # HTML templates
├── tests/                      # Pytest test suite
├── .gitignore
├── Dockerfile                  # Container image definition
├── docker-compose.yml          # Multi-service orchestration
├── init-db.sh                  # Creates fastapi_test_db on first run
├── pytest.ini                  # Pytest configuration & coverage settings
└── requirements.txt            # Python dependencies
```

**Key additions compared to Module 11:**

| File / Folder | What Changed |
|---|---|
| `app/routers/` | New — FastAPI route handlers for calculations and users |
| `app/database_init.py` | New — initializes tables before app starts |
| `Dockerfile` | New — containerizes the FastAPI app |
| `docker-compose.yml` | New — orchestrates web, db, and pgAdmin services |
| `init-db.sh` | New — auto-creates the test database in PostgreSQL |
| `.github/workflows/ci.yml` | New — CI/CD pipeline for tests and Docker Hub push |

---

## 3. How to Run Locally

### Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running
- [Git](https://git-scm.com/downloads) installed

### Step 1 — Clone the Repository

```bash
git clone git@github.com:tl392/module12-calculator-app.git
cd module12-calculator-app
```

### Step 2 — Start All Services

```bash
docker compose up --build
```

This command will:
1. Build the FastAPI application image
2. Start the PostgreSQL database
3. Run `init-db.sh` to create the `fastapi_test_db` test database
4. Initialize the application tables via `app.database_init`
5. Start the FastAPI server with 4 uvicorn workers
6. Start pgAdmin for database management

### Step 3 — Access the Application

| Service | URL | Notes |
|---|---|---|
| API | http://localhost:8000 | FastAPI application |
| Swagger Docs | http://localhost:8000/docs | Interactive API documentation |
| Health Check | http://localhost:8000/health | Container health status |
| pgAdmin | http://localhost:5050 | Email: `admin@example.com` / Password: `admin` |

### Step 4 — Stop All Services

```bash
docker compose down
```

To also remove the database volume (full reset):

```bash
docker compose down -v
```

### Environment Variables

The following variables are pre-configured in `docker-compose.yml` for local development:

| Variable | Value |
|---|---|
| `DATABASE_URL` | `postgresql://postgres:postgres@db:5432/fastapi_db` |
| `TEST_DATABASE_URL` | `postgresql://postgres:postgres@db:5432/fastapi_test_db` |
| `JWT_SECRET_KEY` | `super-secret-key-for-jwt-min-32-chars` |
| `JWT_REFRESH_SECRET_KEY` | `super-refresh-secret-key-min-32-chars` |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | `30` |
| `REFRESH_TOKEN_EXPIRE_DAYS` | `7` |
| `BCRYPT_ROUNDS` | `12` |

> ⚠️ Change the `JWT_SECRET_KEY` and `JWT_REFRESH_SECRET_KEY` values before deploying anywhere outside your local machine.

---

## 4. Running the Tests

Tests are configured in `pytest.ini` and use a separate `fastapi_test_db` database to avoid touching production data.

### Run All Tests

```bash
pytest
```

### Run with Verbose Output

```bash
pytest -v
```

### Run by Marker

```bash
pytest -m fast          # Run only fast tests
pytest -m "not slow"    # Skip slow tests
pytest -m e2e           # Run end-to-end tests only
```

### Coverage Reports

Coverage is automatically generated on every `pytest` run:

- **Terminal report** — shows missing lines in the console
- **HTML report** — saved to `htmlcov/index.html`, open in a browser for full detail

```bash
# Open the HTML coverage report (Mac)
open htmlcov/index.html

# Open the HTML coverage report (Windows)
start htmlcov/index.html
```

### Test Markers Reference

| Marker | Description |
|---|---|
| `slow` | Long-running or integration-heavy tests |
| `fast` | Quick unit tests with no external dependencies |
| `e2e` | Full end-to-end tests through the API |

---

## 5. API Endpoints

All endpoints are available via the interactive Swagger UI at **http://localhost:8000/docs**.

### Auth Endpoints

| Method | Path | Description |
|---|---|---|
| `POST` | `/users/register` | Register a new user |
| `POST` | `/users/login` | Login and receive access + refresh tokens |
| `POST` | `/users/token` | Get access token |


### Calculation Endpoints

| Method | Path | Description |
|---|---|---|
| `POST` | `/calculations/` | Create a new calculation |
| `GET` | `/calculations/` | List all calculations for the current user |
| `GET` | `/calculations/{id}` | Get a specific calculation by ID |
| `PUT` | `/calculations/{id}` | Update a calculation |
| `DELETE` | `/calculations/{id}` | Delete a calculation |

### System Endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/health` | Health check for container orchestration |

### Example Request — Create a Calculation

```json
POST /calculations/
Authorization: Bearer <your_access_token>

{
  "type": "addition",
  "inputs": [10, 20, 30]
}
```

**Response:**

```json
{
  "id": 1,
  "type": "addition",
  "inputs": [10, 20, 30],
  "result": 60.0,
  "user_id": 1,
  "created_at": "2024-01-01T00:00:00"
}
```

---

## 6. How the Calculation Model Works

The calculation model uses **SQLAlchemy single-table polymorphic inheritance**. All four calculation types (`addition`, `subtraction`, `multiplication`, `division`) are stored in a single `calculations` table, and a `type` discriminator column tells SQLAlchemy which Python subclass to use when querying.

### Inheritance Hierarchy

```
Calculation (Base)
├── Addition
├── Subtraction
├── Multiplication
└── Division
```

### How It Works in Practice

```python
# All stored in one table — SQLAlchemy returns the correct subclass
calc = session.query(Calculation).filter_by(id=1).first()

# isinstance() works correctly
assert isinstance(calc, Addition)   # True if type == 'addition'

# Each subclass implements get_result() differently
assert calc.get_result() == 60.0    # Addition: sum of inputs
```

### The `type` Discriminator Column

```
id | type           | inputs      | user_id
---|----------------|-------------|--------
1  | addition       | [10, 20]    | 1
2  | division       | [100, 4]    | 1
3  | multiplication | [3, 4, 5]   | 2
```

When SQLAlchemy loads row `id=2`, it automatically returns a `Division` object because `type = 'division'`.

### `get_result()` Behavior by Type

| Type | Inputs | Result | Logic |
|---|---|---|---|
| `addition` | `[1, 2, 3]` | `6` | Sum all inputs |
| `subtraction` | `[10, 3, 2]` | `5` | Start with first, subtract rest |
| `multiplication` | `[2, 3, 4]` | `24` | Multiply all inputs |
| `division` | `[100, 4]` | `25` | Start with first, divide by rest |

---

## 7. How the Pydantic Schemas Work

Pydantic schemas serve as the **data validation layer** between the HTTP request/response and the database models. They live in `app/schemas/calculation.py`.

### Schema Hierarchy

```
CalculationBase          ← common fields (type, inputs)
├── CalculationCreate    ← used for POST requests (adds user_id)
├── CalculationUpdate    ← used for PUT requests (all fields optional)
└── CalculationResponse  ← used for GET responses (adds id, result, timestamps)
```

### CalculationType Enum

```python
class CalculationType(str, Enum):
    addition       = "addition"
    subtraction    = "subtraction"
    multiplication = "multiplication"
    division       = "division"
```

Using an Enum ensures only valid operation names are accepted — any other string is rejected with a clear error.

### Key Validators

**Minimum inputs check:**
```python
# At least 2 numbers required for any operation
if len(inputs) < 2:
    raise ValueError("Must provide at least 2 inputs")
```

**Division by zero prevention (LBYL — Look Before You Leap):**
```python
# Check BEFORE the operation runs
if type == CalculationType.DIVISION:
    if any(x == 0 for x in inputs[1:]):
        raise ValueError("Cannot divide by zero")
```

**Type normalization:**
```python
# Accepts "Addition" or "ADDITION" — normalizes to "addition"
@field_validator("type")
def normalize_type(cls, v):
    return v.lower()
```

---

## 8. Factory Pattern Explained

The **Factory Pattern** is used to abstract the creation of the correct `Calculation` subclass. Instead of manually instantiating `Addition(...)` or `Division(...)`, callers use a single `Calculation.create()` class method.

### Without Factory (Bad)

```python
# Caller must know which class to use — tightly coupled
if operation == "addition":
    calc = Addition(user_id=1, inputs=[1, 2])
elif operation == "division":
    calc = Division(user_id=1, inputs=[10, 2])
# ... and so on for every type
```

### With Factory (Good)

```python
# Caller just provides the type string — loosely coupled
calc = Calculation.create("addition", user_id=1, inputs=[1, 2])
assert isinstance(calc, Addition)   # Correct subclass returned automatically
assert calc.get_result() == 3
```

### Why It Matters

| Benefit | Explanation |
|---|---|
| **Loose coupling** | Route handlers don't need to import every subclass |
| **Single responsibility** | Object creation logic lives in one place |
| **Extensibility** | Adding a new type (e.g., `modulo`) only requires a new subclass — routes stay unchanged |
| **Testability** | Factory behavior can be tested independently from the routes |

---

## 9. CI/CD Pipeline

The project uses **GitHub Actions** to automate testing and deployment on every push to `main`.

### Pipeline Stages

![alt text](image.png)

### What Triggers the Pipeline

- Every push to the `main` branch
- Every pull request targeting `main`

### Secrets Required

The following secrets must be set in the GitHub repository under **Settings → Secrets and variables → Actions**:

| Secret | Description |
|---|---|
| `DOCKER_USERNAME` | Your Docker Hub username |
| `DOCKER_PASSWORD` | Your Docker Hub password or access token |

### Viewing Pipeline Results

Go to the **Actions** tab of the repository to see the status of each run, view logs, and inspect test output.

---

## 10. Docker Hub

The CI/CD pipeline automatically builds and pushes the Docker image to Docker Hub after every successful test run on `main`.

### Pulling the Image

```bash
docker pull ltaravindh392/module12-calculator-app:latest
```

### Running the Image Standalone

```bash
docker run -p 8000:8000 \
  -e DATABASE_URL=postgresql://postgres:postgres@host.docker.internal:5432/fastapi_db \
  -e JWT_SECRET_KEY=your-secret-key-here \
  ltaravindh392/module12-calculator-app:latest
```

### Image Details

| Detail | Value |
|---|---|
| Base Image | `python:3.10-slim` |
| Exposed Port | `8000` |
| Server | `uvicorn` with 4 workers |
| Run as | Non-root user (`appuser`) |
| Health Check | `GET /health` every 30 seconds |

### Security Notes

- The container runs as a **non-root user** (`appuser`) for improved security
- System packages are upgraded during the build to reduce vulnerabilities
- The health check allows orchestrators (e.g., Kubernetes, ECS) to detect when the app is ready to receive traffic

---

## 11. Screenshots

![alt text](screenshots/image.png)
![alt text](screenshots/image-3.png)
![alt text](screenshots/image-4.png)
![alt text](screenshots/image-5.png)
![alt text](screenshots/image-6.png)
![alt text](screenshots/image-7.png)
![alt text](screenshots/image-8.png)
![alt text](screenshots/image-9.png)
![alt text](screenshots/image-10.png)
![alt text](screenshots/image-11.png)
![alt text](screenshots/image-12.png)
![alt text](screenshots/image-13.png)
![alt text](screenshots/image-14.png)
![alt text](screenshots/image-18.png)
![alt text](screenshots/image-17.png)
![alt text](screenshots/image-16.png)
![alt text](screenshots/image-19.png)
## 📎 Quick Links

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [SQLAlchemy ORM](https://docs.sqlalchemy.org/)
- [Pydantic V2 Docs](https://docs.pydantic.dev/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [pgAdmin](https://www.pgadmin.org/)
- [Docker Hub](https://hub.docker.com/)