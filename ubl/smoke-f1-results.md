# Faza 0 + Faza 3 — Smoke test checklist

**Datum:** 2026-07-02  
**Okolina:** Docker (`finestar_erp`), test DB `test_finestar_erp_db`

## Faza 1 — Integrations (automatski)

| Korak | Test / provjera | Rezultat |
|-------|-----------------|----------|
| Connector registry | `integrations.tests.test_registry` — SUPER eRačun registriran | ✅ OK |
| Connector create | `test_create_super_connector` | ✅ OK |
| Manager orchestrator | `send_eracun` + fiscal skip/call | ✅ OK |
| Seed migration | `test_seed_finestar_super_integration` | ✅ OK |
| Repository | aktivna konfiguracija po tenantu | ✅ OK |

```bash
docker compose exec -T django python manage.py test integrations --keepdb -v2
```

## Faza 3 — Production hardening + DIRECT (automatski)

| Korak | Test / provjera | Rezultat |
|-------|-----------------|----------|
| UblError hijerarhija | `ubl.tests.test_errors` | ✅ CI |
| Admin validation UI | `invoices.tests.test_admin_eracun` — SemanticValidationError poruke | ✅ CI |
| Audit trail | `integrations.tests.test_audit` — failed semantic prije connectora | ✅ CI |
| Schematron HR CIUS | `ubl.tests.test_golden` — finestar + negativni test | ✅ CI |
| Golden master + benchmark | `ubl.tests.test_golden`, `ubl.tests.test_benchmark` | ✅ CI |
| DIRECT connector | `integrations.tests.test_direct_e2e` | ✅ CI |
| AS4 inbound → Expense | `fiscal_gateway.tests.test_inbound_expense` | ✅ CI |

```bash
docker compose exec -T django pytest ubl/tests integrations/tests fiscal_gateway/tests -m "not pts and not pts_integration" -v
```

## Faza 3 — Manual smoke (test okruženje)

| Korak | Akcija | Očekivani rezultat | Status | PTS Verified |
|-------|--------|-------------------|--------|--------------|
| SUPER test send | Admin → Računi → Pošalji eRačun (test tenant) | `SuperDocumentLink` + `IntegrationAuditLog` outbound_sent | ☐ | n/a |
| CIS dry_run | Tenant s CIS config → `fiscalize(dry_run=True)` ili admin test cert | `FiscalSubmissionLog` success + audit fiscalized | ✅ | ✅ |
| Celery sync | `python manage.py sync_super --tenant finestar` | Bez greške, inbound/outbound poll | ☐ | n/a |
| Validation error UI | Namjerno kriv OIB na računu → admin send | Admin prikaže `SemanticValidationError` poruke | ☐ | n/a |
| DIRECT AS4 (PTS) | `pts_eracun_lookup` + `pts_eracun_send --lookup-only` | MPS lookup OK | ✅ | ✅ |
| MPS AMS | `pts_mps_ams --action list` | Lista registriranih OIB-ova | ⚠️ PU AMS 502 | ⚠️ |
| DIRECT receive | Inbound push na AS4 endpoint | Expense + AppResponse + audit | ✅ | ✅ |
| Ops dashboard | `/admin/integrations/integrationconfig/ops/` | Metrike + CSV export | ☐ | n/a |

**PTS Verified:** označiti ✓ kad je korak potvrđen u stvarnom PTS okruženju s audit tragom — vidi [interoperability-matrix.md](../fiskalizacija-2.0/procedures/interoperability-matrix.md).

## PTS interoperability (ručno, `@pytest.mark.pts`)

```bash
# Mock testovi u CI-ju ne zahtijevaju PTS okruženje:
pytest fiscal_gateway/tests/test_pts_commands.py -m pts -v

# E2E checklist (registracija commanda + runbook format):
pytest fiscal_gateway/tests/test_pts_commands.py::TestPtsRunbookChecklist -m pts -v

# Protiv stvarnog PTS okruženja (@pytest.mark.pts_integration):
pytest fiscal_gateway/tests/test_pts_commands.py -m pts_integration -v
```

## Napomene

- **MPS servis:** FastAPI u `services/mps/`, Django koristi `MpsClient` (`fiscal_gateway/client/mps_client.py`) i `MPS_SERVICE_URL`.
- **Schematron:** Vendored subset u `ubl/schematron/hrcius2025.sch`; puni PU ZIP zamjenjuje sadržaj mape kad postane dostupan.
- **UBL potpis:** Aktivira se automatski za DIRECT provider (`IntegrationManager.send_eracun` → `sign=True`).

## Zaključak

Faza 1 integracijski sloj i Faza 3 hardening (errors, audit, schematron, golden, DIRECT connector) pokriveni unit/E2E testovima. Manual smoke checklist za test okruženje gore.
