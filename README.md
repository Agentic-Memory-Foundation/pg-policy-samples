# pg-policy-samples

Runnable SQL samples, domain packs, PEP middleware, and user documentation for [`pg_policy`](https://github.com/rahiakil/pg-policy).

This repository is the artifact companion to the working paper [Policy Beside the Data](https://github.com/Agentic-Memory-Foundation/agentic-policy). The extension kernel lives in [rahiakil/pg-policy](https://github.com/rahiakil/pg-policy). Clone this tree for packs and examples:

```bash
git clone git@github.com:Agentic-Memory-Foundation/pg-policy-samples.git
```

HTTPS: https://github.com/Agentic-Memory-Foundation/pg-policy-samples

## Prerequisites

Install the extension first:

```bash
git clone https://github.com/rahiakil/pg-policy.git
cd pg-policy
make install
psql -d mydb -c "CREATE EXTENSION pg_policy;"
```

PostgreSQL 14+ and a PGXS toolchain (`pg_config` on `PATH`). Details: [INSTALL.md](INSTALL.md).

## Layout

| Path | Role |
| --- | --- |
| [`examples/01-basic-guardrails.sql`](examples/01-basic-guardrails.sql) | Deny DDL, permit SELECT |
| [`examples/02-agent-session-limits.sql`](examples/02-agent-session-limits.sql) | Temporal export budget |
| [`examples/03-guidance-policies.sql`](examples/03-guidance-policies.sql) | Soft obligations (`max_rows`, `prefer_tool`) |
| [`examples/04-rls-complement.sql`](examples/04-rls-complement.sql) | Indexed RLS beside APL |
| [`examples/05-mcp-tool-pack.sql`](examples/05-mcp-tool-pack.sql) | MCP tool-shaped policies |
| [`examples/packs/`](examples/packs/) | Baseline + domain packs (analytics, support, fintech, healthcare, devops, multi-agent) |
| [`examples/integrations/evaluate_middleware.py`](examples/integrations/evaluate_middleware.py) | Python PEP: same-connection `evaluate`, fail-closed sentinels |
| [`doc/language.md`](doc/language.md) | APL syntax |
| [`doc/packs.md`](doc/packs.md) | Pack catalog |
| [`docs/onboarding/`](docs/onboarding/) | 30-minute shadow-then-enforce path |
| [`docs/usecases/`](docs/usecases/) | Who / RLS gap / pack / success |

## Quick start

```bash
export DATABASE_URL=postgres:///mydb
psql "$DATABASE_URL" -f examples/01-basic-guardrails.sql
psql "$DATABASE_URL" -f examples/packs/00-baseline.sql
psql "$DATABASE_URL" -f examples/packs/analytics.sql
```

Load **baseline first**, then one domain pack. Packs do not flip `enforcement_mode`. Start in `log_only`, watch `pg_policy.decision_log`, then `enforce`.

```sql
SELECT pg_policy.set_setting('enforcement_mode', 'log_only');
```

## PEP contract

If a pack mentions `unset` or `approved=false`, middleware must send those sentinels when the real value is missing. Copy [`examples/integrations/evaluate_middleware.py`](examples/integrations/evaluate_middleware.py):

```python
from evaluate_middleware import evaluate, apply_sql_obligations, PolicyDenied, open_session

open_session(conn, thread_id, "langgraph:analytics", {"acting_for": user_id})
decision = evaluate(
    conn,
    agent_id="langgraph:analytics",
    tool="execute_sql",
    context={"statement_type": kind, "tenant_id": tenant, "acting_for": user_id},
    session_id=thread_id,
)
sql = apply_sql_obligations(sql, decision)
```

Adapters for MCP, LangGraph, and raw SQL: [`docs/onboarding/integrations.md`](docs/onboarding/integrations.md).

## Paper mapping

Appendix A of the working paper reprints excerpts from this tree:

| Paper listing | File |
| --- | --- |
| Basic guardrails | `examples/01-basic-guardrails.sql` |
| RLS complement | `examples/04-rls-complement.sql` |
| Baseline pack | `examples/packs/00-baseline.sql` |
| PEP middleware | `examples/integrations/evaluate_middleware.py` |

Catalog, `parse_apl`, and `evaluate` live in the extension (`sql/pg_policy--0.1.0.sql`). Matcher oracle: [`agentic-policy/experiments/policy_engine.py`](https://github.com/Agentic-Memory-Foundation/agentic-policy).

## Related

- Extension: https://github.com/rahiakil/pg-policy
- Working paper: https://github.com/Agentic-Memory-Foundation/agentic-policy

License: [PostgreSQL License](LICENSE).
