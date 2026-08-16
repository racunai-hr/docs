# Load test rezultati — M1.8 Fine Star

**Datum:** 2026-07-02  
**Okolina:** Docker staging (`finestar_erp`, `racunai_mps`), single instance  
**Alati:** pytest benchmark/soak, ručno mjerenje latencije, k6 skripte u `tests/load/`

## Sažetak

| Scenarij | Profil | p50 | p95 | Max RPS / throughput | Greške | Status |
|----------|--------|-----|-----|----------------------|--------|--------|
| MPS `GET /health` | 20× uzorka | 0.5 ms | 0.7 ms | ~2000 req/s (teoretski) | 0% | ✅ |
| Django `GET /api/ready/` | 20× uzorka | 5.7 ms | 53 ms | ~175 req/s | 0% | ✅ |
| AS4 inbound `POST /api/fiscal/as4/inbound/` | k6 smoke (skripta) | — | — | vidi k6 | — | 📋 skripta spremna |
| UBL pipeline | 100× benchmark | 0.3 ms/doc | — | **3365 docs/sec** | 0% | ✅ |
| UBL soak | 100× (default) / 1000× (`UBL_BENCHMARK_ITERATIONS`) | — | — | < 30s / 100× | 0% | ✅ |

## MPS health

```bash
# k6 (smoke 1 VU, 1 min)
k6 run -e MPS_BASE_URL=http://racunai_mps:8000 erp/tests/load/k6_mps_health.js

# Ručno
docker compose exec django curl -s http://racunai_mps:8000/health
```

**Mjerenje 2026-07-02:** p50 0.5 ms, p95 0.7 ms, max 2.6 ms (20 uzoraka iz Django kontejnera).

## Django readiness

Endpoint: `GET /api/ready/` — provjera PostgreSQL + Celery brokera.

```bash
k6 run -e DJANGO_BASE_URL=http://127.0.0.1:8000 erp/tests/load/k6_django_ready.js
docker compose exec django python -c "import urllib.request; print(urllib.request.urlopen('http://127.0.0.1:8000/api/ready/').read())"
```

**Mjerenje 2026-07-02:** p50 5.7 ms, p95 53 ms (cold start max 1150 ms na prvom pozivu nakon restarta).

## AS4 inbound

```bash
k6 run -e DJANGO_BASE_URL=http://127.0.0.1:8000 erp/tests/load/k6_as4_inbound.js
```

Fixture: `tests/load/fixtures/as4_inbound_submit.xml` (PTS invoice u SOAP omotu).

**Napomena:** puni inbound load zahtijeva mock Domibus WS ili test PT; k6 mjeri HTTP sloj (parse + handler). Za E2E s Domibusom koristiti PTS runbook Korak D.

## UBL pipeline

```bash
# Regression (100×, < 30s)
docker compose exec django pytest ubl/tests/test_benchmark.py::test_invoice_pipeline_benchmark -m benchmark -v

# Soak (1000×, < 300s)
docker compose exec django env UBL_BENCHMARK_ITERATIONS=1000 \
  pytest ubl/tests/test_benchmark.py::test_invoice_pipeline_soak -m soak -v
```

**Mjerenje 2026-07-02:** 100× u 0.030 s → **3365 docs/sec**, 0.3 ms/doc.

## Profili (plan)

| Profil | VU | Trajanje | Namjena |
|--------|----|---------|---------|
| smoke | 1 | 1 min | CI / nakon deploya |
| baseline | 5–10 | 10 min | kapacitetni baseline |
| soak | 5 | 4 h | noćni test (ručno zakazati) |

## Ograničenja

- **Namjerno izvan scopea:** flood na CIS/PTS port 8511 — samo runbook scenariji.
- k6 nije instaliran na hostu u trenutku mjerenja; skripte su spremne u `erp/tests/load/`.
- AS4 inbound load ne simulira Domibus push retry/backpressure — za to vidi chaos runbook.
