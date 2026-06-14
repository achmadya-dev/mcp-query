# mcp-query

Local dev workspace for the five MCP query server packages. This repo uses **git submodules** — each `mcp-*-query/` folder tracks its own GitHub remote and npm publish cycle.

## Layout

```
mcp-query/
├── .gitmodules
├── docker-compose.yml   # local MySQL, Postgres, MSSQL, SQLite
├── .cursor/mcp.json     # npx MCP config (uses .env)
├── data/schema.sql      # SQLite seed schema
├── mcp-excel-query/     → submodule: achmadya-dev/mcp-excel-query
├── mcp-mysql-query/     → submodule: achmadya-dev/mcp-mysql-query
├── mcp-postgres-query/  → submodule: achmadya-dev/mcp-postgres-query
├── mcp-mssql-query/     → submodule: achmadya-dev/mcp-mssql-query
└── mcp-sqlite-query/    → submodule: achmadya-dev/mcp-sqlite-query
```

Shared runtime: [mcp-core](https://github.com/achmadya-dev/mcp-core).

## Clone

```bash
git clone --recurse-submodules git@github.com:achmadya-dev/mcp-query.git
cd mcp-query
cp .env.example .env
docker compose up -d
```

If you already cloned without submodules:

```bash
git submodule update --init --recursive
```

## Sync submodules with remote

Update all packages to latest `main`:

```bash
git submodule update --remote --merge
```

Or work inside one package as usual:

```bash
cd mcp-excel-query
git pull origin main
git checkout -b fix/my-change
# commit, push, open PR on that repo
```

Commit pointer updates in the parent when you want to pin new versions:

```bash
cd ..   # back to mcp-query root
git add mcp-excel-query
git commit -m "chore: bump mcp-excel-query submodule"
```

## Develop

Each package is developed in its own repo. Open the folder you need (or clone it directly from GitHub):

```bash
cd mcp-excel-query   # or any sibling repo
pnpm install
pnpm run build
pnpm test
```

Point Cursor at the built entrypoint (`dist/index.js`) — see each repo's README for MCP config and env vars.

### Pull request workflow

```bash
git checkout -b fix/my-change
# edit, build, test
git add -A && git commit -m "fix: …"
git push -u origin fix/my-change
gh pr create --base main
```

CI runs `build` + `test` on every PR to `main`. Merge when checks pass.

### Release (Changesets)

For changes that should ship to npm, add a changeset before merging:

```bash
pnpm changeset   # patch / minor / major + short description
git add .changeset/*.md && git commit -m "chore: add changeset"
```

After merge to `main`:

1. The **Publish** workflow opens or updates a **Version packages** PR (`changeset-release/main`).
2. CI runs on that branch automatically (`build-and-test`); merge when checks pass.
3. Merge that PR → version bump, `CHANGELOG.md`, and publish to npm.

If CI did not run, re-trigger from **Actions → CI → Run workflow** (`workflow_dispatch`).

Install published packages with `npx -y @achmadya-dev/mcp-<name>-query`.

## Local databases (Docker)

Start MySQL, PostgreSQL, SQL Server, and SQLite for MCP testing:

```bash
docker compose up -d
# credentials in .env (see .cursor/mcp.json for npx MCP setup)
```

SQLite is initialized from `data/schema.sql` into `data/dev.db`. MCP config lives in `.cursor/mcp.json` (npx + `envFile`).

To remove broken root-owned folders from an old compose mount (optional):

```bash
sudo rm -rf docker/sqlite/data docker/sqlite/init.sql
```
