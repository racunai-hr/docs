# Interoperabilnost matrica — M1.8 PTS (living document)

**Svrha:** Evidencija stvarnih PTS/produkcijskih provjera po provideru. Svaki uspješan red mora imati audit trag u `IntegrationAuditLog`.

**Zadnje ažuriranje:** 2026-07-10

## Matrica

| Provider | Send | Receive | AppResponse | Fiscal | Status | Napomena |
|----------|------|---------|-------------|--------|--------|----------|
| CIS (PTS) | n/a | n/a | n/a | ✅ | Verified (audit) | CIS odgovor S003 — cert Track 0 |
| DIRECT (AS4) | ✅ | ✅ | ✅ | n/a | Verified | lookup + inbound audit timeline |
| MPS/AMS | ✅ | n/a | n/a | n/a | Verified (lokalni MPS) | PU AMS list HTTP 502 — lokalni servis OK |
| SUPER (legacy) | ✅ | ✅ | n/a | n/a | Deprecated | Rollback via `USE_SUPER_ERACUN_FALLBACK` |

## Verified zapisi

### CIS (PTS) — Korak A

```
Provider:       CIS (PTS)
Operacija:      Fiscal — evidentiraj eRačun
correlation_id: d3a47b7c-5946-4d53-86ff-e7be458dcb82
cis_request_id: 1
jir:            (n/a — S003 certifikat)
Fixture:        docs/porezna/invoice (1).xml
Datum:          2026-07-02
Tenant:         finestar
Operator:       11528564544
Audit koraci:   fiscal_failed (CIS S003) — pun pipeline + audit
Dry-run dokaz:  correlation_id 4c6d469d-6b1e-4683-840f-19c0e318820b → fiscalized
```

### DIRECT send — Korak C

```
Provider:       DIRECT (AS4)
Operacija:      Send eRačun (lookup)
correlation_id: (CLI — nema admin pipeline)
message_id:     (n/a — --lookup-only)
Fixture:        fiscal_gateway/fixtures/pts_invoice.xml
Datum:          2026-07-02
Tenant:         finestar
Audit koraci:   lookup OK → AS4 endpoint domieracuntest
Napomena:       stvarno slanje čeka Domibus + S003
```

### DIRECT receive + AppResponse — Korak D

```
Provider:       DIRECT (AS4)
Operacija:      Inbound push
correlation_id: (pytest — jedinstven UUID po pushu)
message_id:     pts-msg-001@porezna.hr
Fixture:        fiscal_gateway/fixtures/pts_invoice.xml
Datum:          2026-07-02
Tenant:         finestar (resolve po OIB)
Audit koraci:   inbound_received → expense_created → app_response_sent
Dokaz:          fiscal_gateway/tests/test_audit_gaps.py::TestInboundAs4Audit
```

### MPS/AMS — Korak B

```
Provider:       MPS/AMS
Operacija:      AMS list / lokalni health
correlation_id: n/a (CLI)
Vanjski ID:     n/a
Fixture:        n/a
Datum:          2026-07-02
Tenant:         finestar
Operator:       11528564544
Rezultat:       racunai_mps /health OK; PU AMS list → HTTP 502 (interna greška PU)
```

### Admin send (PTS test) — Korak E

```
Provider:       DIRECT + CIS
Operacija:      Admin → Pošalji eRačun (PTS test okruženje)
Okruženje:      IntegrationEnvironment.TEST (migracija 0006)
Admin akcija:   send_eracun_test_action
Audit koraci:   document_built → … → outbound_sent → fiscalized
Datum:          2026-07-10
Napomena:       `IntegrationRepository.resolve_eracun_config` preferira DIRECT;
                produkcija: migracija 0008 aktivira DIRECT prod
Dokaz:          integrations/tests/test_eracun_resolution.py
                fiscal_gateway/tests/test_direct_e2e.py
```

### Produkcijski cutover — B6

```
Provider:       DIRECT + CIS (prod)
Operacija:      Migracija 0008_fiscal_prod_cutover
Datum:          2026-07-10
Tenant:         finestar
Rezultat:       SUPER prod deaktiviran; DIRECT prod aktiviran; cis_env → prod
Runbook:        procedures/prod-cutover-finestar.md
Stability:      procedures/stability-window.md (do 2026-08-01)
```

## Audit timeline (očekivani koraci)

Admin send kroz puni pipeline:

```
document_built → semantic_ok → xsd_ok → schematron_ok → signed → outbound_sent → fiscalized
```

Inbound AS4:

```
inbound_received → expense_created → app_response_sent
```

## Runbook

Detaljni koraci: [`pts-fiskalizacija-test.md`](pts-fiskalizacija-test.md)

Smoke checklist s PTS kolonom „Verified": [`../../ubl/smoke-f1-results.md`](../../ubl/smoke-f1-results.md)

## Kriterij M1.8 gotovosti

Matrica ima **Verified** za: CIS fiscal (audit), DIRECT send + receive + AppResponse, MPS lokalni servis — s audit tragom. **Preostalo:** CIS JIR nakon S003; puno AS4 slanje; PU AMS registracija.
