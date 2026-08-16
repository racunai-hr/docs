# Architecture Principles — racunAI ERP

Globalna tehnička pravila za sve developere. Detalji po domeni: [`DOMAIN_MAP.md`](DOMAIN_MAP.md). Odlučivanje o promjenama: [`ARCHITECTURE_GOVERNANCE.md`](ARCHITECTURE_GOVERNANCE.md).

---

## Principi

### 1. Business logic u domenama

Poslovna logika živi u `domains/*/services/` (ciljna struktura) ili privremeno u `*/services/` postojećih Django appova. Views, admin i API su thin — delegiraju na servise.

### 2. Django modeli = persistence layer

Modeli drže podatke, relacije i osnovnu validaciju. Ne sadrže poslovne workflowe, agregacije ni integracijske pozive.

### 3. Thin views i admin

View/admin metode: validacija ulaza → poziv servisa → render odgovora. Nema if-lanaca poslovne logike u template kontekstu.

### 4. API-first

Svaka značajna funkcionalnost ima REST endpoint s OpenAPI dokumentacijom. Admin je operativni alat, ne jedini ulaz.

### 5. Event-driven za cross-domain side effects

Side effects između domena idu preko `events/` busa (`publish` / `register_handler`), ne Django signala. Vidi [`EVENTS.md`](EVENTS.md).

### 6. Multi-tenant by default

Svaki upit, servis i event nosi `tenant_id`. `TenantMixin` + tenant-scoped manageri. Nema globalnih upita bez eksplicitnog razloga.

### 7. Audit za poslovne radnje

Knjiženja, PDV predaje, bankovne transakcije i integracijski pozivi ostavljaju audit trag (`AuditLog`, `SubmissionEvent`, lifecycle audit).

### 8. Immutable accounting

Proknjižene temeljnice se ne mijenjaju — samo storno/reversal. `JournalEntry.posted_at` je točka bez povratka.

### 9. Append-only tax evidence

PDV knjiženja, `VATReturn` verzije i `SubmissionEvent` su append-only nakon predaje. Ispravci = nova verzija, ne overwrite.

### 10. No cross-domain imports

Domena A ne importira servise domene B direktno. Komunikacija: eventi, shared helpers ili facade API. Vidi [`DOMAIN_DEPENDENCY_MAP.md`](DOMAIN_DEPENDENCY_MAP.md).

### 11. Backward compatibility

Breaking promjene zahtijevaju ADR, verzioniranje API-ja i migration notes. PDV/banking/submission jezgre imaju poseban freeze (ADR-0007, ADR-0009, banking-v2).

### 12. ADR za arhitekturne promjene

Nova domena, novi Django app, promjena governance pravila ili jezgre zahtijeva ADR prije implementacije.

### 13. Async jobs za spore operacije

PDF generiranje, bank sync, fiskalizacija outbox, PDV agregacija za velike periode — Celery taskovi, ne sync HTTP.

### 14. Idempotent integrations

Connector pozivi i webhook handleri moraju biti idempotentni (correlation_id, payload_hash, dedup ključevi).

### 15. Config over hardcoding

Stope PDV-a, mapping pravila, connector URL-ovi, feature flagovi — u bazi ili settings/env, ne u kodu.

---

## Zamrznute jezgre (ne dirati bez ADR-a)

| Jezgra | ADR / doc |
|--------|-----------|
| PDV pipeline | [`ADR-0007-pdv-module.md`](ADR-0007-pdv-module.md) |
| Submission modul | [`ADR-0009-submission-module-v1.md`](ADR-0009-submission-module-v1.md) |
| Banking v2 | [`banking-v2.md`](banking-v2.md) |
| Dokumentacijska struktura | [`ADR-0011-architecture-freeze-v1.0.md`](ADR-0011-architecture-freeze-v1.0.md) |

---

## Shared vs domain

| Sloj | Sadržaj | Primjer |
|------|---------|---------|
| `shared/` | Pure helpers, bez poslovne logike | `money/`, `vat/` (oznake), `iban/`, `oib/` |
| `domains/*/` | Poslovna logika, use-cases, facades | `finance/services/posting.py` |
| Django apps | Persistence, admin, URL routing | `accounting/models.py` |

**Pravilo:** PDV pipeline (`aggregate_vat_boxes`, `build_pdv_payload`) nije u `shared/vat/` — to je Tax domena.
