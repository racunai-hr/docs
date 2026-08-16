# Events — racunAI ERP

Katalog domenskih događaja. Princip: [`ARCHITECTURE_PRINCIPLES.md`](ARCHITECTURE_PRINCIPLES.md) §5. Ovisnosti: [`DOMAIN_DEPENDENCY_MAP.md`](DOMAIN_DEPENDENCY_MAP.md).

**Pravilo:** Svaki novi event zahtijeva ažuriranje ovog dokumenta pri mergeu (quality gate).

---

## Event Bus

Implementacija: `erp/erp/app/events/`

```python
from events.dispatcher import DomainEvent, publish, register_handler

@dataclass(frozen=True, kw_only=True)
class MyEvent(DomainEvent):
    tenant_id: int
    ...

register_handler(MyEvent, my_handler)
publish(MyEvent(tenant_id=1, ...))
```

Trenutno **in-process** sync dispatcher. Nije distributed message broker.

---

## Katalog događaja

| Event | Publisher | Subscriber | Status | Mehanizam |
|-------|-----------|------------|--------|-----------|
| `PaymentExecuted` | `banking` (`payment_order_lifecycle.py`) | `accounting` (`payment_posting.py`) | **implemented** | Event bus |
| `InvoiceIssued` | `accounting/signals` (post_save Invoice) | posting service (`post_document`) | **implemented** | Django signal |
| `InvoicePaid` | `accounting/signals` (post_save Invoice) | posting service + integration | **implemented** | Django signal |
| `ExpenseApproved` | `accounting/signals` (post_save Expense) | posting service | **partial** | Django signal |
| `ExpensePaid` | `accounting/signals` (post_save Expense) | posting service | **implemented** | Django signal |
| `PaymentPosted` | `accounting/signals` (post_save Payment) | posting service | **implemented** | Django signal |
| `VatLedgerGenerated` | `accounting/vat.py` (`generate_vat_ledger`) | PDV aggregate | **implemented** | Direct call |
| `VatReturnGenerated` | `tax_forms/pdv` | submission service | **implemented** | Direct call |
| `VatSubmitted` | `SubmissionService` | audit | **implemented** | Direct call |
| `PdvSSubmitted` | `SubmissionService` | audit | **implemented** | Direct call |
| `PaymentImported` | `banking` (AIS sync) | matching | **implemented** | Direct call |
| `PaymentMatched` | `banking` (matching service) | posting | **partial** | Direct call |
| `InvoiceCreated` | `invoices` | — | **planned** | Event bus |
| `InvoiceSent` | `integrations` | audit | **partial** | Direct call |
| `AssetActivated` | `assets` | depreciation | **planned** | Event bus |
| `DepreciationCalculated` | `assets` | posting | **planned** | Event bus |

---

## Detalji implementiranih eventa

### PaymentExecuted

```python
@dataclass(frozen=True, kw_only=True)
class PaymentExecuted(DomainEvent):
    tenant_id: int
    payment_order_id: int
    payment_id: int | None
    bank_connection_id: int
    provider_code: str
    amount: Decimal
    currency: str
    execution_date: date
    bank_reference: str
    correlation_id: UUID | None
```

| Polje | Opis |
|-------|------|
| Publisher | `banking/services/payment_order_lifecycle.py` — emitira na prijelaz u `executed` |
| Subscriber | `accounting/services/payment_posting.py` → `handle_payment_executed()` |
| Idempotency | `PaymentOrder.posting_journal_entry` FK — drugi publish ne knjiži ponovo |
| Registracija | `accounting/apps.py` → `register_handler(PaymentExecuted, handle_payment_executed)` |

Vidi [`banking-v2.md`](banking-v2.md).

### InvoiceIssued / InvoicePaid

| Polje | Opis |
|-------|------|
| Publisher | `accounting/signals.py` → `auto_post_invoice` (Django post_save) |
| Trigger | Invoice status → `sent`/`paid`/`overdue` (issued), `paid` (paid) |
| Subscriber | `accounting/services/posting.py` → `post_document(tenant, instance, 'invoice_issued', user)` |
| Napomena | Trenutno Django signal, ne event bus — migracija na event bus planirana |

### ExpenseApproved / ExpensePaid

| Polje | Opis |
|-------|------|
| Publisher | `accounting/signals.py` → `auto_post_expense` |
| Trigger | Expense status → `approved`/`paid` |
| Subscriber | `post_document(tenant, instance, 'expense_approved', user)` |
| Status partial | Nema event bus registracije; nema idempotency guard |

### VatSubmitted / PdvSSubmitted

| Polje | Opis |
|-------|------|
| Publisher | `SubmissionService.create_event()` via `mark_vat_return_submitted()` / `mark_pdv_s_submitted()` |
| Subscriber | Audit (`SubmissionEvent` persistencija) |
| Idempotency | `event_uuid` + `payload_hash` (ADR-0009) |

---

## Planirani eventi

### InvoiceCreated

| Polje | Opis |
|-------|------|
| Publisher | `invoices` — na kreiranje računa |
| Payload | `tenant_id`, `invoice_id`, `partner_id`, `total_amount` |
| Subscriber | TBD (audit, notifikacije) |
| Idempotency | `invoice_id` dedup |

### AssetActivated

| Polje | Opis |
|-------|------|
| Publisher | `domains/assets/` — na aktivaciju imovine |
| Payload | `tenant_id`, `asset_id`, `acquisition_cost`, `useful_life_months` |
| Subscriber | Depreciation scheduler, posting service |
| Idempotency | `asset_id` + status guard |

### DepreciationCalculated

| Polje | Opis |
|-------|------|
| Publisher | `domains/assets/` — periodični Celery task |
| Payload | `tenant_id`, `asset_id`, `period`, `depreciation_amount` |
| Subscriber | Posting service → JournalEntry |
| Idempotency | `asset_id` + `period` unique constraint |

---

## Konvencije

| Pravilo | Opis |
|---------|------|
| Immutable | Event dataclass je `@dataclass(frozen=True, kw_only=True)` |
| Tenant scope | Svaki event nosi `tenant_id` |
| Idempotency | Subscriber mora biti idempotentan (FK guard, dedup ključ) |
| No Django signals za cross-domain | Novi cross-domain side effects → event bus, ne post_save |
| Dokumentacija | Novi event = ažuriranje ovog kataloga + Issue |

### Migracija signala → event bus

Postojeći Django signali (`auto_post_invoice`, `auto_post_expense`, `auto_post_payment`) planirano migrirati na event bus u Sprint 2–3. Do tada ostaju dokumentirani kao „Django signal" mehanizam.
