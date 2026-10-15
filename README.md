::: {align="center"}
# UpTrack

### Containerized Website Uptime Monitoring & Incident Intelligence

**Monitor availability. Inspect performance. Orchestrate every service
with Docker.**

A multi-service website monitoring platform with a React dashboard,
FastAPI backend, and PostgreSQL database---packaged and orchestrated
with Docker Compose.
:::

------------------------------------------------------------------------

## Why UpTrack?

UpTrack brings website monitoring data into one dashboard. Register
websites, trigger on-demand checks, inspect check history, and review
uptime and response-time statistics through a FastAPI-powered API.

The project also demonstrates a practical containerized application
architecture: each major service runs independently, services
communicate over a Compose network, Nginx routes API traffic to the
backend, and PostgreSQL data is persisted using a named volume.

> **Deployment status:** The current Docker Compose configuration runs
> the application locally. It is a foundation for cloud deployment, not
> a claim that the project is already hosted on a public cloud.

## Docker-First Architecture

``` mermaid
flowchart TB
    U["User / Browser"]
    subgraph COMPOSE["Docker Compose Application"]
        subgraph FE["frontend container"]
            R["React + TypeScript build"]
            N["Nginx web server"]
            R --> N
        end
        subgraph BE["backend container"]
            API["FastAPI / Uvicorn"]
            MON["Website checks, history and statistics"]
            API --> MON
        end
        subgraph DB["db container"]
            PG[("PostgreSQL")]
        end
        VOL[("Named volume: uptrack_data")]
        N -- "/api/* proxy" --> API
        MON <--> PG
        PG --- VOL
    end
    U -->|"HTTP :5173"| N
    API -. "host port :8000" .- U
```

### What Docker does in this project

  -----------------------------------------------------------------------
  Docker component                    Role in UpTrack
  ----------------------------------- -----------------------------------
  `frontend/Dockerfile`               Builds the React frontend and
                                      packages the production assets into
                                      an Nginx image.

  `backend/Dockerfile`                Packages the FastAPI application
                                      and Python dependencies into a
                                      backend image.

  `docker-compose.yml`                Defines the frontend, backend, and
                                      PostgreSQL services; configures
                                      ports, environment variables,
                                      dependencies, and networking.

  Compose network                     Lets containers reach each other by
                                      service name, such as the backend
                                      connecting to `db:5432`.

  `uptrack_data` volume               Persists PostgreSQL files
                                      independently of the database
                                      container lifecycle.

  Nginx configuration                 Serves the frontend and forwards
                                      `/api/` requests to `backend:8000`.

  Database health check               Helps Compose wait until PostgreSQL
                                      is ready before starting the
                                      backend.
  -----------------------------------------------------------------------

### Request flow

1.  The user opens the dashboard at `http://localhost:5173`.
2.  Nginx serves the built React application.
3.  The frontend sends API calls under `/api/`.
4.  Nginx forwards those requests to the FastAPI backend container.
5.  FastAPI runs the relevant monitoring or statistics logic and
    reads/writes PostgreSQL data.
6.  PostgreSQL stores application records in the `uptrack_data` volume.

### Why separate containers?

-   **Isolation:** each service has its own runtime and dependencies.
-   **Reproducibility:** images describe how the services are packaged.
-   **Service discovery:** Compose provides an internal network and
    service-name DNS.
-   **Simpler startup:** one Compose command brings up the application
    stack.
-   **Persistent data:** the database volume is not tied to a disposable
    container.
-   **Portability:** the same containerized design can be deployed to a
    suitable server or cloud environment after production configuration
    and security review.

## Current Capabilities

-   Register, list, view, and remove monitored websites
-   Trigger a manual website check
-   Retrieve uptime and response-time statistics
-   Inspect check history and incident records
-   Store application data in PostgreSQL
-   Run the frontend, backend, and database through Docker Compose

*These capabilities reflect the project's exposed backend endpoints and
architecture. The exact UI behavior depends on the current frontend
implementation.*

## Technology Stack

  Layer                           Technologies
  ------------------------------- ---------------------------------------
  Frontend                        React, TypeScript, Vite, Tailwind CSS
  Production frontend server      Nginx
  Backend API                     Python, FastAPI, Uvicorn
  Database access                 SQLAlchemy
  Database                        PostgreSQL
  Containerization                Docker
  Multi-container orchestration   Docker Compose

## Run the Complete Stack

### Prerequisites

-   Docker Desktop installed and running
-   Git installed

### 1. Clone the repository

``` bash
git clone <YOUR-GITHUB-REPOSITORY-URL>
cd <REPOSITORY-FOLDER>
```

Replace the placeholders with your repository URL and the folder Git
creates.

### 2. Build and start the services

Run from the directory containing `docker-compose.yml`:

``` bash
docker compose up --build -d
```

Compose builds the frontend and backend images, creates the configured
network and volume if needed, waits for the database health check, and
starts the services.

### 3. Verify service status

``` bash
docker compose ps
```

### 4. Open the application

  Service                             URL
  ----------------------------------- ------------------------------
  **UpTrack dashboard**               http://localhost:5173
  FastAPI interactive documentation   http://localhost:8000/docs
  Backend health endpoint             http://localhost:8000/health

## Docker Operations

``` bash
# Build images and start all services in the background
docker compose up --build -d

# Show service status and port mappings
docker compose ps

# Follow backend logs
docker compose logs -f backend

# Follow frontend logs
docker compose logs -f frontend

# Follow database logs
docker compose logs -f db

# Rebuild and recreate services after code changes
docker compose up --build -d

# Stop and remove containers and the Compose network
docker compose down
```

**Important --- database persistence:** The database uses the named
volume `uptrack_data`. The command `docker compose down` preserves that
volume. Avoid `docker compose down -v` unless you deliberately want to
remove the volume and its stored database data.

## API Endpoints

  --------------------------------------------------------------------------------------------
  Method                  Endpoint                                     Purpose
  ----------------------- -------------------------------------------- -----------------------
  `GET`                   `/health`                                    Check API/database
                                                                       health

  `GET`                   `/websites/`                                 List websites

  `POST`                  `/websites/`                                 Add a website

  `GET`                   `/websites/{id}`                             Retrieve website
                                                                       details

  `POST`                  `/websites/{id}/check`                       Trigger a manual check

  `GET`                   `/websites/{id}/checks`                      Retrieve check history

  `GET`                   `/websites/{id}/incidents`                   Retrieve incident
                                                                       records

  `DELETE`                `/websites/{id}`                             Delete a website

  `GET`                   `/stats/overview`                            Retrieve overview
                                                                       statistics

  `GET`                   `/stats/uptime?hours=24`                     Retrieve uptime
                                                                       statistics

  `GET`                   `/stats/response-times?hours=24&limit=100`   Retrieve response-time
                                                                       data
  --------------------------------------------------------------------------------------------

Explore the API at `http://localhost:8000/docs` while the backend is
running.

## Frontend Development Without the Frontend Container

Use this mode for rapid UI changes and Vite hot reload. Keep the backend
reachable on port `8000`.

``` bash
cd frontend
npm install
npm run dev
```

Open the URL Vite prints. The development proxy forwards `/api` to
`http://127.0.0.1:8000` by default. If the Docker frontend is already
using port `5173`, Vite may choose another port.

To create a frontend build:

``` bash
npm run build
```

## Backend Development Without the Backend Container

Use this mode when debugging FastAPI directly on your machine. Avoid
running the local backend and Docker backend on port `8000`
simultaneously.

``` powershell
cd backend
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m uvicorn main:app --reload --port 8000
```

Configure `DATABASE_URL` for your database location: inside Compose,
PostgreSQL is reachable as `db:5432`; from the host, the provided port
mapping exposes it at `localhost:5433`.

## Cloud Computing Connection

The current implementation demonstrates containerization, multi-service
orchestration, service networking, health-based startup ordering, and
persistent storage **on a local machine**. To make it a cloud-hosted
deployment, the stack could be deployed to a cloud VM or container
platform and hardened with:

-   HTTPS and a domain name
-   Secure secrets and environment configuration
-   Managed PostgreSQL and backups
-   Centralized logs and operational monitoring
-   Resource limits, recovery strategies, and scaling evaluation

These are possible next steps, not features claimed to be already
deployed.

## Security Notes

-   Never commit credentials, access tokens, API keys, or private `.env`
    files.
-   Replace development database credentials before any public
    deployment.
-   Keep `.venv`, `node_modules`, and build artifacts out of Git.
-   Before exposing the application publicly, review authentication,
    authorization, input validation, HTTPS, database access, and secret
    handling.

## Troubleshooting

-   **Compose cannot find its configuration:** run commands from the
    directory containing `docker-compose.yml`.
-   **`package.json` not found:** run npm commands inside `frontend/`.
-   **`vite` not recognized:** run `npm install` inside `frontend/`.
-   **`No module named uvicorn`:** install `backend/requirements.txt` in
    the intended Python environment.
-   **Port already in use:** avoid running the same service in Docker
    and locally at the same time.

## Contributors

Rajapurohitham Sahana
Rosanne Chugh
