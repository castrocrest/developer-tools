# GitHub Actions Quick Reference

Copy these workflow blocks into your `.github/workflows/` directory.

## Python CI with ruff + pytest

```yaml
# .github/workflows/python-ci.yml
name: Python CI
on: [push, pull_request]
jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip
      - run: pip install -e ".[dev]"
      - run: ruff check .
      - run: ruff format --check .
      - run: mypy src/
      - run: pytest --cov --cov-report=xml
```

## TypeScript CI with pnpm + vitest

```yaml
# .github/workflows/typescript-ci.yml
name: TypeScript CI
on: [push, pull_request]
jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: pnpm
      - run: pnpm install --frozen-lockfile
      - run: pnpm tsc --noEmit
      - run: pnpm lint
      - run: pnpm test --coverage
```

## Docker publish to ghcr.io

```yaml
# .github/workflows/docker-publish.yml
name: Docker
on:
  push:
    branches: [main]
    tags: ["v*"]
permissions:
  contents: read
  packages: write
jobs:
  push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v6
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

**Full pack** (8 production workflows including security scan, auto-release, Fly.io deploy, Dependabot config):  
https://castrocrest.gumroad.com/l/fbfgwt — $12
