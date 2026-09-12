# Cloud Resume Challenge – Full‑Stack (Backend + Frontend)

## Overview
This repository contains the **complete solution** for the Cloud Resume Challenge:
- A **backend** implemented as Azure Functions (Python) that stores and returns page‑view counts in Azure Table Storage and secures the API with Azure AD/JWT.
- A **frontend** – a static site served locally for development – with a Cypress test suite that validates the end‑to‑end flow.
- A GitHub Actions workflow that runs the Cypress tests, then builds and deploys the static site to Azure Blob Storage (static website) and purges the CDN.

The two parts are independent but designed to work together when the frontend points to the backend API.

---

## Quick Start
### 1. Clone the repository
```bash
git clone https://github.com/your-org/cloud-resume-challenge.git
cd cloud-resume-challenge
```

### 2. Set up the **backend** (Azure Functions)
```bash
# Create and activate a Python virtual environment
python -m venv .venv
# Linux/macOS
source .venv/bin/activate
# Windows PowerShell
# .venv\Scripts\Activate.ps1

# Install Python dependencies (assumes a requirements.txt is present)
pip install -r requirements.txt
```

### 3. Set up the **frontend** (Cypress tests)
```bash
# Install Node.js dependencies (Cypress, type definitions, etc.)
npm ci   # runs `npm ci` using the root package.json
```

### 4. Configure local settings
- **Backend** – copy `local.settings.sample.json` (if it exists) to `local.settings.json` and fill in the required Azure values (see *Configuration* below).
- **Cypress** – the workflow sets `CYPRESS_BASE_URL` automatically. For local runs you can export it manually:
```bash
export CYPRESS_BASE_URL=http://localhost:5000   # macOS/Linux
set CYPRESS_BASE_URL=http://localhost:5000      # Windows CMD
```

### 5. Run the application locally
```bash
# 1️⃣ Start the Azure Functions backend
func start &

# 2️⃣ Serve the static frontend (Python simple HTTP server)
cd frontend
python -m http.server 5000 &

# 3️⃣ Execute the Cypress end‑to‑end tests
npm run cypress:run
```
The Cypress suite will hit `http://localhost:5000` (the static site) which in turn calls the backend API.

---

## Tech Stack
| Layer | Technology |
|-------|------------|
| **Backend Runtime** | Python 3.11, Azure Functions |
| **Backend Data Store** | Azure Table Storage |
| **Backend Auth** | Azure AD (MSAL) + PyJWT |
| **Backend Testing** | pytest, pytest‑mock |
| **Frontend** | Static HTML/JS (served by Python HTTP server) |
| **E2E Tests** | Cypress 15.x |
| **CI/CD** | GitHub Actions (OIDC to Azure) |
| **Deployment Target** | Azure Blob Storage static website + Azure CDN |

---

## Project Structure
```
cloud-resume-challenge/
│
├─ .github/                     # GitHub Actions workflows
│   └─ workflows/
│       └─ main.yml            # CI – Cypress + Azure deploy
│
├─ backend/                     # Azure Functions source (original repo root)
│   ├─ host.json
│   ├─ local.settings.json      # **DO NOT COMMIT** – contains secrets
│   ├─ requirements.txt
│   ├─ utils/                   # auth, storage helpers
│   ├─ main.py                  # HTTP triggers
│   └─ tests/                   # pytest suite
│
├─ frontend/                    # Static site files (HTML, CSS, JS)
│   ├─ index.html
│   ├─ script.js
│   └─ ...
│
├─ package.json                 # Root npm config – Cypress dev dependency
├─ frontend/package.json        # Frontend‑specific npm config (type definitions)
└─ README.md                    # **This file**
```
*The backend files shown above are representative – adjust the list to match the actual layout of your repo.*

---

## Configuration
### Backend (Azure Functions)
| Setting | Description | Example |
|---------|-------------|---------|
| `AzureWebJobsStorage` | Storage account used by the Functions runtime. | `DefaultEndpointsProtocol=https;AccountName=...;AccountKey=...;EndpointSuffix=core.windows.net` |
| `TABLE_STORAGE_CONNECTION_STRING` | Connection string for the Table Storage that holds the view counter. | Same format as above |
| `TABLE_NAME` | Name of the Azure Table used for counters. | `PageViews` |
| `CLIENT_ID` | Azure AD Application (client) ID. | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| `TENANT_ID` | Azure AD tenant ID. | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| `CLIENT_SECRET` | Secret for the Azure AD app (keep secret!). | `my-secret-value` |
| `ALLOWED_ORIGINS` | Comma‑separated list of origins allowed for CORS. | `https://myresume.com` |

Create a **local.settings.json** file (copy from a sample if provided) and populate the values. Do **not** commit this file.

### Cypress / Frontend
- `CYPRESS_BASE_URL` – Base URL of the site under test (defaults to `http://localhost:5000`).
- No additional secrets are required for the test suite.

---

## Running the Project
### Locally
1. **Backend** – start Azure Functions:
   ```bash
   func start
   ```
2. **Frontend** – serve the static files:
   ```bash
   cd frontend
   python -m http.server 5000
   ```
3. **Cypress** – run the end‑to‑end tests:
   ```bash
   npm run cypress:run
   ```
   The tests will use the `CYPRESS_BASE_URL` environment variable to reach the site.

### GitHub Actions (CI)
The workflow defined in `.github/workflows/main.yml` performs the following steps on every push to `main`:
1. **Checkout** the repository.
2. **Set up Node.js** (v20) and install npm dependencies.
3. **Start a temporary Python HTTP server** on port 5000 (serves `frontend/`).
4. **Run Cypress** (`npm run cypress:run`).
5. **If tests pass**, build a `dist/` folder, copy the static site into it, and upload the content to an Azure Storage account (`visitorcounterstgaccdev`).
6. **Purge the Azure CDN** to make the new version live.
7. **Logout** from Azure.

The workflow uses **OpenID Connect** (OIDC) to obtain an Azure AD token without storing service‑principal secrets.

---

## Key Dependencies
### Backend (Python)
| Package | Version | Purpose |
|---------|---------|---------|
| `azure-functions` | 1.23.0 | Azure Functions runtime bindings |
| `azure-data-tables` | 12.7.0 | Table Storage client |
| `azure-identity` | 1.25.0 | Azure AD authentication helpers |
| `msal` | 1.33.0 | Acquire Azure AD tokens |
| `PyJWT` | 2.10.1 | JWT validation |
| `requests` | 2.32.5 | HTTP client |
| `pytest` | 8.4.2 | Test runner |
| `pytest-mock` | 3.15.1 | Mocking utilities |
| `cryptography` | 46.0.1 | Crypto primitives used by auth libraries |

### Frontend (Node)
| Package | Version | Purpose |
|---------|---------|---------|
| `cypress` | ^15.3.0 | End‑to‑end testing framework |
| `@types/node` | ^24.5.2 | TypeScript type definitions for Node (dev only) |

---

## Contributing
Contributions are welcome! Please follow these steps:
1. **Fork** the repository and create a feature branch.
2. **Implement** your change. If you add new functionality, include appropriate unit tests (`backend/tests/`) **and** Cypress tests (`cypress/e2e/`).
3. **Run the full test suite** locally:
   ```bash
   # Backend unit tests
   pytest

   # Cypress end‑to‑end tests
   npm run cypress:run
   ```
4. **Commit** with a clear message and push to your fork.
5. Open a **Pull Request** targeting the `main` branch. Provide a concise description of the change and reference any related issues.

*Remember to keep secrets out of the repository. If you introduce new configuration keys, update the *Configuration* section of this README accordingly.*
