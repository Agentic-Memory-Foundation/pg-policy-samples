# Installing pg_policy (then these samples)

This repository is **samples and docs**, not the extension. Build the kernel from [rahiakil/pg-policy](https://github.com/rahiakil/pg-policy).

## Prerequisites

- PostgreSQL 14 or newer
- Development files for that PostgreSQL (`postgresql-server-dev-*` / Xcode + Postgres.app / Postgres from Homebrew)
- GNU make
- `pg_config` on your `PATH`

## Build and install the extension

```bash
git clone https://github.com/rahiakil/pg-policy.git
cd pg-policy
make
make install
```

Then in each database:

```sql
CREATE EXTENSION pg_policy;
```

## Verify

```sql
SELECT extname, extversion FROM pg_extension WHERE extname = 'pg_policy';
SELECT pg_policy.parse_apl($apl$
permit
  principal agent "a"
  action tool "t"
$apl$);
```

## Run samples

From this repository:

```bash
export DATABASE_URL=postgres:///mydb
psql "$DATABASE_URL" -f examples/01-basic-guardrails.sql
psql "$DATABASE_URL" -f examples/packs/00-baseline.sql
```

## Uninstall

```sql
DROP EXTENSION pg_policy CASCADE;
```

## Packaged installs

PGXN and OS packages are planned; see `docs/roadmap.md` in the extension repo.
