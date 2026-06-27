# pytest Quick Reference

The most useful pytest patterns. Copy what you need.

## conftest.py — async FastAPI setup

```python
import pytest
import pytest_asyncio
from httpx import AsyncClient, ASGITransport
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

from myapp.main import app
from myapp.database import Base, get_db

# Session-scoped engine (reuse across all tests)
@pytest_asyncio.fixture(scope="session")
async def async_engine():
    engine = create_async_engine("sqlite+aiosqlite:///:memory:", echo=False)
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield engine
    await engine.dispose()

# Per-test session with rollback (tests never dirty each other)
@pytest_asyncio.fixture
async def db_session(async_engine):
    async_session = sessionmaker(async_engine, class_=AsyncSession, expire_on_commit=False)
    async with async_session() as session:
        async with session.begin_nested():
            yield session
            await session.rollback()

# HTTP client wired to the FastAPI app
@pytest_asyncio.fixture
async def async_client(db_session):
    async def override_get_db():
        yield db_session
    app.dependency_overrides[get_db] = override_get_db
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as c:
        yield c
    app.dependency_overrides.clear()

# Pre-authed client
@pytest_asyncio.fixture
async def authed_client(async_client):
    async_client.headers.update({"Authorization": "Bearer test-token"})
    yield async_client
```

## pyproject.toml — pytest config

```toml
[tool.pytest.ini_options]
asyncio_mode = "auto"   # No need to mark every async test
testpaths = ["tests"]
addopts = "-v --tb=short"

[tool.coverage.run]
source = ["src"]
omit = ["src/migrations/*", "tests/*"]

[tool.coverage.report]
fail_under = 80
```

## Test patterns

```python
# Parametrize
@pytest.mark.parametrize("email,valid", [
    ("user@example.com", True),
    ("not-an-email", False),
    ("", False),
])
def test_email_validation(email, valid):
    assert validate_email(email) == valid

# Expected exception
def test_raises_on_invalid():
    with pytest.raises(ValueError, match="must be positive"):
        create_order(amount=-1)

# Temporary directory
def test_file_creation(tmp_path):
    output = tmp_path / "report.csv"
    generate_report(output)
    assert output.exists()

# Mock an external service
def test_with_mock(mocker):
    mock_send = mocker.patch("myapp.mail.send", return_value=True)
    result = send_welcome_email("user@example.com")
    mock_send.assert_called_once_with("user@example.com", "Welcome!")
    assert result is True
```

## Marks

```python
@pytest.mark.slow
@pytest.mark.integration  # Run with: pytest -m integration
@pytest.mark.skip(reason="WIP")
@pytest.mark.xfail(reason="known bug #123")
```

---

**Full pack** (auth fixtures, DB sessions, HTTP clients, factory_boy factories):  
https://castrocrest.gumroad.com/l/zezgl — Complete Bundle
