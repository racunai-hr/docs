# Operational Readiness Review (ORR) — Fine Star M1.8

**Datum pregleda:** 2026-07-02  
**Topologija:** single instance (`finestar_erp`, `racunai_mps`, `racunai_domibus`)  
**Cilj:** go/no-go prije šireg produkcijskog rollouta DIRECT + CIS fiskalizacije

## SLO-ovi (Fine Star, single instance)

| SLO | Target | Mjerenje | Baseline (2026-07-02) |
|-----|--------|----------|------------------------|
| Outbound uspješnost | ≥ 99% / 7 dana | ops dashboard: sent / (sent + failed) | N/A — < 7 dana podataka |
| E2E latencija (admin send) | p95 < 30s | audit `duration_ms` po `correlation_id` | N/A — čeka produkcijski promet |
| Outbox recovery | 100% transient → retry ili dead + alert | outbox status distribucija | ✅ pytest + admin reprocess |
| Inbound processing | ≥ 99.5% push → Expense < 60s | audit inbound koraci | ✅ audit timeline u testu |
| UBL pipeline | 100× < 30s | `test_benchmark` | ✅ 0.03s / 100× |
| Dostupnost MPS/Domibus | synthetic check / 5 min | `/health`, `/rest/application/info` | ✅ MPS; Domibus skip iz mreže |

## Error budget

- **Outbound:** 1% failure = ~7 failed sends / 700 mjesečno → alert + incident runbook ([developer-guide.md](../../integrations/developer-guide.md))
- **Prag alerta:** `INTEGRATION_ALERT_OUTBOUND_FAILED_THRESHOLD` (default 5 / 1h)
- **Trenutno stanje:** CIS S003 (certifikat) i PU AMS 500 — ne troše outbound budget, ali blokiraju puni PTS Verified

## Go / no-go checklist

### Must-pass (blokeri)

| # | Kriterij | Status | Dokaz |
|---|----------|--------|-------|
| 1 | Interoperability matrica: CIS + DIRECT + MPS | ⚠️ djelomično | [interoperability-matrix.md](interoperability-matrix.md) — audit tragovi popunjeni; CIS JIR čeka S003 |
| 2 | Load baseline dokumentiran | ✅ | [load-test-results.md](load-test-results.md) |
| 3 | Chaos drill 1–4 | ⚠️ djelomično | [chaos-drill-runbook.md](chaos-drill-runbook.md) — drill 2,3 + outbox pytest |
| 4 | Outbox retry + dead-letter + admin reprocess | ✅ | `integrations/tests/test_outbox.py` |
| 5 | SLO baseline (7 dana ili soak) | ☐ | 30-day window nakon go |
| 6 | Incident runbook testiran | ✅ | Drill 2 → `fiscal_failed` + audit |
| 7 | Track 0 (cert S003, firewall 443) | ☐ | S003 aktivan na PTS CIS |

### Should-pass (ne blokira M1.8 kod)

| # | Kriterij | Status |
|---|----------|--------|
| 1 | Sentry/APM na Django + MPS | ☐ |
| 2 | Django `/api/ready/` u Docker Compose healthcheck | ✅ |
| 3 | CI workflow `pytest -m "not pts and not pts_integration"` | ✅ postojeći testovi |

### Eksplicitno odgođeno (M2+)

- Multi-instance AS4 (M2.4)
- SUPER uklanjanje (M1.7)
- Novi connectori (MER, Peppol)

## Odluka

| | |
|---|---|
| **Preporuka** | **Conditional go** — nastaviti 30-day stability window s tjednim ops exportom |
| **Blokeri prije produkcije** | Riješiti S003 certifikat; PU admin proširenje svrhe testiranja; pokrenuti Celery worker u stacku |
| **Sljedeći korak** | [stability-window.md](stability-window.md) |

## Prilozi

- Ops CSV export: `/admin/integrations/integrationconfig/ops/export/`
- Audit correlation primjeri: vidi interoperability matricu
- Load rezultati: `load-test-results.md`
- Chaos rezultati: `chaos-drill-runbook.md`

## Potpis

| Uloga | Ime | Datum | Go / No-go |
|-------|-----|-------|------------|
| Fine Star operator | | | |
| racunAI ops | | | |
| Tehnički vlasnik | | | |
