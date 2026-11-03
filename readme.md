# NOTE

- This document is provided as a **starter template only**.
- Customers must review, update, and validate this content to ensure it meets their **functional, security, compliance, and operational requirements** before deployment.

# CrewAI Python Template
![Python](https://img.shields.io/badge/Python-black?logo=python)
![FastAPI](https://img.shields.io/badge/FastAPI-black?logo=fastapi)
![Uvicorn](https://img.shields.io/badge/Uvicorn-black?logo=uvicorn)
![CrewAI](https://img.shields.io/badge/CrewAI-black?logo=crewai)
![uv](https://img.shields.io/badge/uv-black?logo=astral)
![Docker](https://img.shields.io/badge/Docker-black?logo=docker)
![Helm](https://img.shields.io/badge/Helm-black?logo=helm)

## Overview
A two-agent system to generate high-quality, original, beginner-friendly blog posts on-demand from any user-provided topic. The Researcher agent extracts accurate, concise key points and factual notes that cover the beginner scope of a 600-800 word post. The Writer agent transforms those research_notes into a clear, engaging blog post with an introduction, key sections, and a conclusion, all delivered in a single synchronous execution flow.

## Agents
- **Researcher** (Content Research and Fact Extraction) — Extracts accurate, concise, and original key points and factual notes from the supplied topic, validates coverage for a beginner-friendly 600-800 word blog post, and produces `research_notes` for downstream use.
- **Writer** (Blog Post Composition and Structuring) — Transforms the Researcher's `research_notes` into an original 600-800 word blog post with an introduction, key sections, and conclusion, all written in clear, engaging language for general readers, and produces the final `blog_post` text.

## Topology
- Researcher → Writer (sync) — Pass `research_notes` containing key points, factual notes, and coverage status from Researcher to Writer for blog post composition.

## Assumptions
- The user provides a clear and sufficiently descriptive topic input for research.
- No external data fetching or live research is performed; all content is generated from the input and internal knowledge.
- The system runs on-demand and produces the blog post in a single synchronous execution flow.
- Originality is ensured by internal content generation logic without direct copying from external sources.

## Known Risks
- Insufficient or ambiguous topic input may lead to incomplete or low-quality research notes and blog posts.
- Lack of external verification may cause factual inaccuracies or outdated information in the blog post.
- The system may produce content that is too generic or lacks depth if the topic scope is not well captured.
- No explicit content moderation or prohibited content filtering may allow unintended content if input is malicious.

## 1. Project Overview
This repository hosts a CrewAI-powered Python service that orchestrates a Researcher → Writer workflow to turn a user-provided topic into a beginner-friendly blog post. FastAPI exposes a `/health` endpoint and an `/ask` endpoint that accepts a topic request, triggers the crew, and returns the resulting `blog_post` text as the final deliverable.

### Template Origin
This project was initialized using **crewai**.

```bash
crewai create crew latest-ai-development
```

## 2. Tech Stack

### Detected Stack

| Technology / Framework | Detected Version |
| --- | --- |
| Python | `>=3.10,<3.14` |
| CrewAI (`crewai[azure-ai-inference,google-genai,tools]`) | `1.10.1` |
| FastAPI | `>=0.128.0` |
| Uvicorn | `>=0.40.0` |
| APScheduler | `>=3.11.2` |
| boto3 | `>=1.42.47` |
| azure-identity | `>=1.25.1` |
| azure-keyvault-secrets | `>=4.10.0` |
| OpenAI SDK | `>=1.108.1` |
| LiteLLM | `>=1.75.3` |
| fastapi-sso | `>=0.17.0` |
| email-validator | `>=2.3.0` |
| Hatchling (build backend) | Not specified |
| Docker (runtime image) | `python:3.13-slim` |
| Docker (builder image) | `ghcr.io/astral-sh/uv:python3.13-bookworm` |
| Helm Chart | `version: 0.1.0` (chart), `appVersion: "1.0"` |

- **Package manager detected:** `uv`
- **Lockfile(s) found:** `uv.lock`
- **Runtime version hints found:** `requires-python = ">=3.10,<3.14"` in `pyproject.toml`

## 3. Project Structure Explanation

```bash
.
├── src/
│   └── latest_ai_development/
│       ├── config/
│       │   ├── agents.yaml            # Agent definitions (roles/goals/backstories that match Researcher + Writer)
│       │   └── tasks.yaml             # Task definitions (researcher_task + writer_task)
│       ├── tools/
│       │   ├── __init__.py            # Tools package marker
│       │   └── custom_tool.py         # Example CrewAI BaseTool template
│       ├── __init__.py                # Package marker for latest_ai_development
│       ├── crew.py                    # CrewAI orchestration wiring Researcher → Writer and LLM parameters
│       ├── main.py                    # FastAPI app + CLI-style entry functions (run/train/replay/test)
│       └── secrets_manager.py         # AWS Secrets Manager / Azure Key Vault helper
├── knowledge/
│   └── user_preference.txt            # Example “knowledge” seed content used by the project
├── helm_chart/                        # Kubernetes deployment templates (Helm)
│   ├── Chart.yaml                     # Chart metadata (name is placeholder)
│   ├── values.yaml                    # Deployment values (placeholders for image/ports/context)
│   └── templates/                     # K8s manifests (Deployment/Service/Ingress/etc.)
├── Dockerfile                         # Container build/runtime definition (uv-based build)
├── Jenkinsfile                        # Jenkins pipeline entrypoint
├── Jenkinsfile.ci                     # CI pipeline definition (build/test/scan/publish flow)
├── Jenkinsfile.deploy                 # Deploy/testing pipeline definition
├── pyproject.toml                     # Python project metadata + dependencies + scripts
├── uv.lock                            # uv dependency lockfile
├── .dockerignore                      # Docker context exclusions
├── .gitignore                         # Git ignore rules
├── AGENTS.md                          # Repo documentation describing the multi-agent setup
```

## 4. Key config files

- **`pyproject.toml`** — Declares project metadata, Python version constraints, dependencies, and entry scripts; incorrect edits can break installs, dependency resolution, or console commands.
- **`uv.lock`** — Frozen dependency graph for `uv`; changing/removing it can make environments non-reproducible or break `uv sync --frozen`.

## 5. Key Files Explained

- **`src/latest_ai_development/main.py`**
  - What it does: Defines the FastAPI app, `/health` and `/ask` endpoints, and several CLI-style functions (`run`, `train`, `replay`, `test`, `run_with_trigger`) that kick off the crew with `topic` inputs.
  - If modified incorrectly: API routes can break, server startup can fail, or the Crew execution entrypoints can stop working.

- **`src/latest_ai_development/crew.py`**
  - What it does: Declares the CrewAI `CrewBase` class, wires Researcher and Writer agents/tasks from YAML, and configures the LLM parameters used by each agent.
  - If modified incorrectly: Agent/task registration can break, YAML mappings can fail, or the orchestration process (sequential Researcher → Writer) can change unexpectedly.

- **`src/latest_ai_development/config/agents.yaml`** and **`src/latest_ai_development/config/tasks.yaml`**
  - What they do: Define agent personas/config and task definitions expected by the CrewAI decorators in `crew.py`, ensuring Researcher produces `research_notes` and Writer consumes them to emit `blog_post`.
  - If modified incorrectly: Missing keys or mismatched names can cause runtime errors when building agents/tasks.

- **`src/latest_ai_development/secrets_manager.py`**
  - What it does: Provides a thin wrapper around AWS Secrets Manager and Azure Key Vault retrieval.
  - If modified incorrectly: Secret fetching can fail at runtime, causing downstream auth/config failures.

- **`src/latest_ai_development/tools/custom_tool.py`**
  - What it does: A template for extending CrewAI tools using a Pydantic schema.
  - If modified incorrectly: Tool argument schemas or runtime tool execution may fail.

- **`Dockerfile`**
  - What it does: Builds dependencies using `uv`, copies a prebuilt virtualenv into a slim runtime image, and starts the app via Python module execution.
  - If modified incorrectly: Image may not build, dependencies may not exist at runtime, or the container may fail to start.

- **`helm_chart/Chart.yaml`** 
  — Helm chart metadata; wrong values can break chart packaging/installation.
- **`helm_chart/values.yaml`** 
  — Runtime configuration defaults (image/tag/ports/context placeholders); incorrect changes can break deployments.
- **`helm_chart/templates/*`** 
  — Kubernetes manifests; incorrect edits can prevent pods/services/ingress from creating correctly.
- **`Jenkinsfile` / `Jenkinsfile.ci` / `Jenkinsfile.deploy`** 
  — CI/CD pipeline definitions; incorrect edits can break builds, scans, publishing, or deploy flows.
- **`AGENTS.md`** 
  — Project documentation; stale edits can mislead onboarding, but won’t change runtime behavior.

## 6. Setup & Installation

Based on the available code, a typical local setup looks like:

**Prerequisites**
- Python in the supported range: `>=3.10,<3.14` (also see Docker images using Python 3.13).
- `uv` (detected via `uv.lock`).

**Install dependencies**
```bash
uv sync
```

**Start Server**
```bash
PYTHONPATH=src uv run python -m latest_ai_development.main
```

**Secrets Configuration**
- Set `CLOUD_PROVIDER=AWS` to retrieve secrets from AWS Secrets Manager.
- Set `CLOUD_PROVIDER=AZURE` to retrieve secrets from Azure Key Vault.
- For the Amazon Bedrock provider, set  `API_KEY_SECRET` to the secret name that stores the API key.
- `BEDROCKAI_API_KEY_SECRET` is supported only as a backward-compatible fallback when `API_KEY_SECRET` is not set.
- For the OpenAI provider, set `API_KEY_SECRET` to the secret name that stores the API key.
- `OPENAI_API_KEY_SECRET` is supported only as a backward-compatible fallback when `API_KEY_SECRET` is not set.
- For the Anthropic provider, set `API_KEY_SECRET` to the secret name that stores the Anthropic API key.
- `ANTHROPICAI_API_KEY_SECRET` is supported only as a backward-compatible fallback when `API_KEY_SECRET` is not set.
- For the Azure OpenAI provider, set `API_KEY_SECRET` to the secret name that stores the Azure OpenAI API key.
- `AZUREAI_OPENAI_API_KEY_SECRET` is supported only as a backward-compatible fallback when `API_KEY_SECRET` is not set.
- For the Microsoft Foundry AI provider, set `API_KEY_SECRET` to the secret name that stores the Microsoft Foundry AI API key.
- `MICROSOFTFOUNDRYAI_API_KEY_SECRET` is supported only as a backward-compatible fallback when `API_KEY_SECRET` is not set.
- For Microsoft Foundry AI, set `MICROSOFTFOUNDRYAI_ENDPOINT` to the Microsoft Foundry AI operation endpoint. Eg. https://<resource-name>resource.services.ai.azure.com/openai
- For the Gemini provider, set `API_KEY_SECRET` to the secret name that stores the Gemini API key.
- `GEMINIAI_API_KEY_SECRET` is supported only as a backward-compatible fallback when `API_KEY_SECRET` is not set.
- For the Ollama provider, set `OLLAMA_BASE_URL` to the Ollama API base URL.
- When using AWS Secrets Manager, configure `AWS_REGION` and standard AWS credentials for `boto3`.
- When using Azure, configure `AZURE_KEY_VAULT_URL` and an Azure identity supported by `DefaultAzureCredential`.

## 7. Development Guidelines

- **Keep orchestration in `crew.py`, not in `main.py`.** The code comments in `main.py` indicate it’s intended as a lightweight runner and API wrapper.
- **Prefer YAML changes for behavior tweaks.** Agent roles/goals and task definitions live under `src/latest_ai_development/config/*.yaml`. Keep code changes for wiring/implementation, not for content.
- **Follow the “src layout” convention consistently.** Imports and runtime execution rely on `PYTHONPATH=src` (mirroring the container’s `PYTHONPATH` approach).
- **Be careful with generated artifacts.** The Writer task returns `blog_post` text output, so if you change task keys or expected output formats, consumers may break.
- **Treat deployment/CI templates as controlled assets.** Helm and Jenkins files appear to be standardized templates; changes can break build/deploy automation even if application code is correct.
- **Avoid committing runtime artifacts.** The presence of `__pycache__/` and `*.pyc` suggests local artifacts may be present; ensure these are ignored/cleaned in your workflow.

## 8. Security & Networking (HTTP / HTTPS)

- HTTP support is available out of the box.
- For secure deployments, customers are expected to enable and configure TLS/HTTPS after creating the template repository under Helm charts.
- While HTTP is supported, we strongly recommend using HTTPS for all production deployments.
