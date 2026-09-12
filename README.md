# Cloud Resume Challenge – Backend

## Overview
The **cloud-resume-challenge-backend** repository implements the server‑side component of the Cloud Resume Challenge using **Azure Functions** (Python). It provides a simple REST API that:
- Tracks page‑view counts stored in **Azure Table Storage**.
- Secures endpoints with **Azure AD** (MSAL) and **JWT** validation.
- Demonstrates best practices for serverless development, testing, and CI/CD on Azure.

## Quick Start
1. **Clone the repository**
   ```bash
   git clone https://github.com/your‑org/cloud-resume-challenge-backend.git
   cd cloud-resume-challenge-backend
   ```
2. **Create a virtual environment & install dependencies**
   ```bash
   python -m venv .venv
   source .venv/bin/activate   # on Windows: .venv\Scripts\activate
   pip install -r requirements.txt
   ```
3. **Configure local settings** (see *Configuration* below).
4. **Run the function locally**
   ```bash
   func start
   ```
   The API will be available at `http://localhost:7071/api/...`.
5. **Run the test suite**
   ```bash
   pytest
   ```

## Tech Stack
| Layer | Technology |
|-------|------------|
| Runtime | Python 3.11 |
| Serverless platform | Azure Functions |
| Data store | Azure Table Storage |
| Authentication | Azure AD (MSAL) + PyJWT |
| HTTP client | `requests` |
| Testing | `pytest`, `pytest‑mock` |
| CI/CD (suggested) | Azure Pipelines / GitHub Actions |

## Project Structure
```
cloud-resume-challenge-backend/
│
├─ .venv/                     # Virtual environment (not committed)
├─ .funcignore                # Files ignored by Azure Functions Core Tools
├─ host.json                  # Global function app configuration
├─ local.settings.json        # Local dev settings (generated, not committed)
├─ requirements.txt           # Python dependencies
├─ tests/                     # Unit / integration tests
│   └─ ...
├─ .
├─ __init__.py                # Package marker (if needed)
├─ main.py                    # Entry point for Azure Functions (HTTP triggers)
├─ utils/                     # Helper modules (auth, storage, etc.)
│   ├─ auth.py
│   ├─ storage.py
│   └─ ...
└─ README.md                  # This file
```
*Adjust the file names to match the actual layout of your repo.*

## Configuration
The function app expects the following settings (available via **local.settings.json** for local development and **Application Settings** in Azure):

| Setting | Description | Example |
|---------|-------------|---------|
| `AzureWebJobsStorage` | Connection string for the storage account used by Azure Functions runtime. | `DefaultEndpointsProtocol=https;AccountName=...;AccountKey=...;EndpointSuffix=core.windows.net` |
| `TABLE_STORAGE_CONNECTION_STRING` | Connection string for the Table Storage that holds the view counter. | Same format as above |
| `TABLE_NAME` | Name of the Azure Table used for counters. | `PageViews` |
| `CLIENT_ID` | Azure AD Application (client) ID used for token acquisition. | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| `TENANT_ID` | Azure AD tenant ID. | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| `CLIENT_SECRET` | Secret for the Azure AD app (keep this secure!). | `my-secret-value` |
| `ALLOWED_ORIGINS` | Comma‑separated list of origins allowed for CORS. | `https://myresume.com` |

Create a **local.settings.json** file (copy from `local.settings.sample.json` if provided) and populate the values above. Do **not** commit secrets to source control.

## Running the Project
### Locally
```bash
# Activate venv if not already active
source .venv/bin/activate
func start
```
The Functions Core Tools will spin up a local runtime. Use tools like **cURL**, **Postman**, or a browser to hit the endpoints, e.g.:
```bash
curl http://localhost:7071/api/views
```
### Deploying to Azure
1. **Create an Azure Function App** (Python runtime) via the portal or CLI.
2. **Configure the Application Settings** with the same keys listed in *Configuration*.
3. **Deploy** using one of the following methods:
   - **Azure CLI**
     ```bash
     az functionapp deployment source config-zip \
         --resource-group <RG> \
         --name <FUNC_APP_NAME> \
         --src ./functionapp.zip
     ```
   - **GitHub Actions** – add a workflow that runs `pip install -r requirements.txt` and `func azure functionapp publish <FUNC_APP_NAME>`.
4. Verify the endpoint at `https://<FUNC_APP_NAME>.azurewebsites.net/api/...`.

## Key Dependencies
| Package | Version | Purpose |
|---------|---------|---------|
| `azure-functions` | 1.23.0 | Azure Functions runtime bindings for Python |
| `azure-data-tables` | 12.7.0 | Interact with Azure Table Storage |
| `azure-identity` | 1.25.0 | Simplified Azure AD authentication helpers |
| `msal` / `msal-extensions` | 1.33.0 / 1.3.1 | Acquire and cache Azure AD tokens |
| `PyJWT` | 2.10.1 | Decode/validate JWT access tokens |
| `requests` | 2.32.5 | HTTP client for external calls (e.g., health checks) |
| `pytest` & `pytest-mock` | 8.4.2 / 3.15.1 | Unit testing framework |
| `cryptography`, `cffi` | 46.0.1 / 2.0.0 | Underlying crypto primitives used by auth libraries |

## Contributing
Contributions are welcome! Follow these steps:
1. **Fork** the repository and create a feature branch.
2. **Write tests** for any new functionality (place them under `tests/`).
3. Ensure the test suite passes locally:
   ```bash
   pytest
   ```
4. **Commit** with clear messages and **push** to your fork.
5. Open a **Pull Request** targeting the `main` branch. Include a brief description of the change and any relevant issue numbers.

*Please keep the repository free of sensitive information. If you need to add new configuration keys, update the documentation accordingly.*