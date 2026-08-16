# Integracije — developer guide

Vodič za proširenje integracijskog sloja (eRačun, fiskalizacija, AS4).

## Arhitektura

```
Invoice → IntegrationManager → EracunDocumentService (UBL pipeline)
                             → Connector (SUPER / DIRECT / CIS)
                             → IntegrationAuditLog
                             → IntegrationOutboxMessage (retry)
```

## 1. Dodavanje novog providera

1. **Konstante** — `integrations/constants.py`: dodaj `IntegrationProvider` vrijednost
2. **Connector** — implementiraj sučelje u `fiscal_gateway/connector_*.py` ili novi modul
3. **Provider loader** — `integrations/provider_loaders.py`
4. **Profil** — `integrations/provider_profiles.py` (health check, env vars)
5. **Registracija** — `@register` dekorator u `integrations/registry.py`

Primjer: `super_integration/connector_eracun.py`, `fiscal_gateway/connector_eracun.py`

## 2. Audit hook

Koristi `integrations.audit.log_audit_step`:

```python
from integrations.audit import log_audit_step, new_correlation_id
from integrations.models import IntegrationAuditLog

correlation_id = new_correlation_id()
log_audit_step(
    tenant=invoice.tenant,
    step=IntegrationAuditLog.STEP_OUTBOUND_SENT,
    status=IntegrationAuditLog.STATUS_SUCCESS,
    correlation_id=correlation_id,
    invoice=invoice,
    integration_type='eracun',
    provider='direct',
    detail={'message_id': '...'},
)
```

Koraci: `document_built`, `semantic_ok`, `xsd_ok`, `schematron_ok`, `signed`, `outbound_sent`, `fiscalized`, `outbound_failed`, `fiscal_failed`.

Timeline: Admin → `/admin/integrations/integrationconfig/ops/timeline/<correlation_id>/`

## 3. Golden test

Vidi `ubl/tests/golden/README.md`:

1. Dodaj JSON fixture u `ubl/tests/golden/hrcius2025/fixtures/`
2. Generiraj XML referencu
3. Test u `ubl/tests/test_golden.py` (parametrizirano po `profile_version`)

Renderer registry: `ubl/builder/registry.py` — `get_renderer('2025')`.

## 4. PTS runbook

[`docs/fiskalizacija-2.0/procedures/pts-fiskalizacija-test.md`](../fiskalizacija-2.0/procedures/pts-fiskalizacija-test.md)

```bash
pytest fiscal_gateway/tests/test_pts_commands.py -m pts -v
```

Interoperability matrica: [`interoperability-matrix.md`](../fiskalizacija-2.0/procedures/interoperability-matrix.md)

## 5. Interpretacija audit timelinea

| Korak | Značenje |
|-------|----------|
| `semantic_ok` failed | Poslovna validacija (OIB, totali) — terminalno |
| `xsd_ok` failed | XML struktura — terminalno |
| `schematron_ok` failed | HR CIUS pravila — terminalno |
| `schematron_skipped` | PU Schematron ZIP nedostaje — provjeri `ubl/schematron/` |
| `signed` failed | Certifikat / XAdES — provjeri `FiscalTenantConfig` |
| `outbound_failed` | AS4/SUPER transport — može biti retryable |
| `fiscal_failed` | CIS odbio — provjeri `FiscalSubmissionLog` |

Ops pregled: `/admin/integrations/integrationconfig/ops/`

---

# Incident runbook — outbound_failed

## Simptomi

- Admin send prikaže grešku
- Audit log: `outbound_failed`
- `IntegrationOutboxMessage` u statusu `retrying` ili `dead`

## Koraci

1. **Identificiraj correlation_id** — Admin → Integracijski audit logovi
2. **Timeline** — otvori ops timeline link; provjeri je li pipeline stao prije outbounda
3. **Terminal vs transient:**
   - Semantic/XSD/Schematron greška → ispravi podatke na računu, ne retry
   - `As4SendError` / timeout / 5xx → outbox automatski retry (1m → 5m → 15m → 1h)
4. **Domibus** — provjeri message log za `message_id` iz audit detail
5. **Dead letter** — Admin → Integracijski outbox → akcija „Ponovi slanje“
6. **Alerting** — `INTEGRATION_ALERT_OUTBOUND_FAILED_THRESHOLD` (default 5/h) šalje mail + Notification

## CIS fiscal_failed (produkcija)

1. `FiscalSubmissionLog` → `error_code`, `response_xml`
2. Certifikat na pouzdanom popisu (S003)
3. Alert automatski za produkcijski tenant

## Schematron 100% skip

1. Dodaj `HRUBLSchematron*.zip` u `ubl/schematron/`
2. Alert kad svi dokumenti imaju `schematron_skipped`

## Korisne naredbe

```bash
# Outbox retry worker
celery -A config call fiscal_gateway.process_due_outbox_messages

# PTS lookup
python manage.py pts_eracun_lookup --xml fiscal_gateway/fixtures/pts_invoice.xml

# Poll AS4 status
python manage.py shell -c "
from integrations.manager import IntegrationManager
from tenants.models import Tenant
print(IntegrationManager.poll_eracun_outbound(Tenant.objects.get(slug='finestar')))
"
```
