# Context & Governance

## 1. Project Context
- **Goal**: Lightweight LiteLLM proxy wrapper with LangChain agents and Ollama backends.
- **Stack**: Python, Docker Compose, LiteLLM, LangChain, Ollama.
- **Structure**: Flat root for config/infra, `src/` for agent logic.

## 2. Operational Commands
- **Start Stack**: `docker compose up -d`
- **View Logs**: `docker compose logs -f litellm`
- **Init Models**: `docker exec -it tinyllama1 ollama run tinyllama` (repeat for 2, 3)
- **Smoke Test**: `curl http://localhost:4444/models -H "Authorization: Bearer sk-4444"`
- **Chat Test**: `curl http://localhost:4444/v1/chat/completions -H "Content-Type: application/json" -d '{"model": "tinyllama", "messages": [{"role": "user", "content": "hi"}]}'`
- **Linting**: `tox -e ruff`
- **Shutdown**: `docker compose down -v`

## 3. Golden Rules

### Immutable Constraints
- **Line Limit**: This file must stay < 500 lines.
- **Tokens**: No emojis, no secrets, no tables in Context Maps.
- **Auth**: Always use `Authorization: Bearer sk-4444` for local proxy.
- **Ports**: LiteLLM on `4444`.

### Do's & Don'ts
- **DO**: Use `tinyllama{1..3}` naming pattern for services.
- **DO**: Keep YAML indented with 2 spaces, lowercase-hyphenated keys.
- **DO**: Use uppercase for all environment variables.
- **DO**: Run smoke tests after every `docker-compose.yml` change.
- **DON'T**: Hardcode secrets (use `.env` or `litellm_settings.yml`).
- **DON'T**: Modify `litellm_settings.yml` without validating against schema.

## 4. SOLID & Design Principles
- **SRP**: Modules have one reason to change (e.g., separate config from agent logic).
- **OCP**: Extend agent capabilities via new classes/tools, not by modifying core loop.
- **LSP**: LLM provider implementations must be interchangeable.
- **ISP**: Agents should only depend on tools they actually use.
- **DIP**: High-level agents depend on abstract model interfaces, not concrete APIs.
- **DRY**: Define model names and API keys in one place (`litellm_settings.yml`).

## 5. TDD Protocol
1. **Red**: Write a failing test (or smoke test script) for the new feature.
2. **Green**: Implement minimal code/config to pass.
3. **Refactor**: Optimize while keeping tests green.
- **Coverage**: Critical logic in `src/` requires unit tests. Infra requires smoke tests.

## 6. Standards & References
- **Coding**: PEP 8 for Python, YAML 2-space indent.
- **Git**: Conventional Commits (e.g., `feat: add new model`).
- **Docs**: Update `README.md` for runbook changes.

## 7. Context Map
- **[Agent Logic](./src)** — LangChain agent implementations
- **[Model Config](./litellm_settings.yml)** — Routing, rate limits, and credentials
- **[Infrastructure](./docker-compose.yml)** — Service orchestration (LiteLLM + Ollama)
- **[Dependencies](./pyproject.toml)** — Python package requirements