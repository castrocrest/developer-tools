# Claude Code Quick Reference

Useful patterns for working with Claude Code CLI.

## CLAUDE.md template (FastAPI)

Place this in your project root — Claude Code reads it automatically.

```markdown
# Project Context

## Stack
- Python 3.12, FastAPI 0.115, Pydantic v2
- SQLAlchemy 2.0 async with asyncpg, Alembic for migrations
- pytest + pytest-asyncio (asyncio_mode = "auto")
- ruff for lint/format, mypy --strict

## Conventions
- `src/` layout: `routers/`, `models/`, `services/`, `repositories/`, `schemas/`
- Repository pattern: DB queries live in `repositories/`, business logic in `services/`
- FastAPI Annotated dependencies: `Annotated[AsyncSession, Depends(get_db)]`
- Pydantic models: `model_config = ConfigDict(from_attributes=True)`
- Async everywhere: all route handlers, DB calls, and service methods are async def

## Testing
- Use `pytest.mark.asyncio` or configure `asyncio_mode = "auto"` in pyproject.toml
- Fixtures in `conftest.py`: `db_session` (per-test rollback), `async_client` (httpx ASGI), `auth_headers`
- One test class per router file; test module structure mirrors `src/routers/`

## Don\'ts
- No synchronous DB calls (no `session.execute()` without `await`)
- No business logic in routers — delegate to services
- No bare `except Exception` — be specific
- No `select *` — always specify columns
```

## Slash commands (drop in .claude/commands/)

### /standup
```markdown
---
description: Generate standup from recent git activity
---
Run: git log --oneline --since="24 hours ago" --author=$(git config user.email)
Format the output as:
**Yesterday:** [bullet points of commits]
**Today:** [what logically follows from the work]
**Blockers:** [any WIP branches or TODO comments in changed files]
```

### /test-gaps
```markdown
---
description: Find untested code paths
---
1. List all source files in src/ that have corresponding test files missing
2. For each source file, check function coverage by comparing function names in src vs test files
3. Report: files with no tests, functions with no test cases
4. If --write flag: generate test stubs for the top 3 missing test cases
```

---

**Full pack** (7 production slash commands — standup, readme, security-audit, release, test-gaps, db-migrate, explain):  
https://castrocrest.gumroad.com/l/agwofd — $14
