# Tax Domain — racunAI ERP

Developer vodič za Tax domenu. Registry svih obrazaca: [`FORM_REGISTRY.md`](FORM_REGISTRY.md) (SSOT).

Arhitektura: ADR-0007 (PDV), ADR-0009 (Submission), ADR-0010 (domene). **Nema zasebnog ADR-a za Tax Forms Engine** — to je implementacijska konvencija.

---

## Infrastructure freeze — Sprint 3 (2026-07-09)

**Tax infrastruktura je zamrznuta** za trajanje Sprinta 3. Sprint 3 je **feature sprint**, ne arhitekturni.

| Zamrznuto (ne dirati) | Dozvoljeno |
|------------------------|------------|
| `domains/tax/` struktura | Implementacija unutar `tax_forms/zp/` |
| `FORM_IMPLEMENTATION_CONVENTION.md` | ZP moduli + testovi |
| `TaxFormEngine` protocol | XSD import, aggregate/render/parse |
| Fixture layout `fixtures/tax/<form>/` | Novi JSON scenariji samo ako test zahtijeva |
| Submission facade (re-export) | `ZPReturn`, `submit.py` |

**Ne uvoditi:** nove konvencije, ADR-0017, premještaj `SubmissionService`, refaktor PDV/PDV-S.

Potvrda uspjeha standardizacije: **ADR-0014** ✅ (Sprint 3 zatvoren 2026-07-10).

Dokumentacija (living, ne infrastruktura):

| Dokument | Uloga |
|----------|-------|
| [`FORM_REGISTRY.md`](FORM_REGISTRY.md) | SSOT obrazaca + maturity |
| [`EPOREZNA_SUBMISSION_MATRIX.md`](EPOREZNA_SUBMISSION_MATRIX.md) | Kanali predaje (XSD/G2B/Portal) — **inicijalni paket završen** (plan v3) |
| [`FORM_IMPLEMENTATION_CONVENTION.md`](FORM_IMPLEMENTATION_CONVENTION.md) | Layout koda |
| [`TESTING_GUIDE.md`](TESTING_GUIDE.md) | Obvezni testni paket po obrascu |
| [`ZP_ARCHITECTURE.md`](ZP_ARCHITECTURE.md) | ZP operativni doc |

### Poslovni prioritet obrazaca (P0–P3)

Vizualni pregled — detalji u [`FORM_REGISTRY.md`](FORM_REGISTRY.md) i [`EPOREZNA_SUBMISSION_MATRIX.md`](EPOREZNA_SUBMISSION_MATRIX.md).

| Prioritet | Obrasci | Zrelost (sažetak) |
|-----------|---------|-------------------|
| **P0** ★★★ | PDV, PDV-ispravak, PDV-S, ZP, JOPPD | PDV L3 · PDV-S L2 · ZP/JOPPD L0 |
| **P1** ★★ | PD, OSS, PDV-K | L0 |
| **P2** ★ | PPO, PZ 42/63, PD-IPO, DONH, DOH, INO-DOH, EPOM, PPN, OPZ-STAT-1 | L0 |
| **P3** | Preknjiženja, ostalo | L0 |

---

## Četiri odgovornosti

Tax domena **nije samo obrasci**. Razdvojene su četiri (plus AI) odgovornosti:

```text
domains/tax/

    ledger/           # I-RA, U-RA, VATLedgerEntry
    forms/            # PDV, PDV-S, ZP, JOPPD, …
    submission/       # facade → SubmissionService (frozen)
    validation/       # XSD, poslovna pravila
    verification/     # diff payload/XML, verify period, integrity
    joppd_builder/    # cross-domain agregacija za JOPPD (stub)
    ai/               # auto-PDV pravila (stub)
```

| Sloj | Pitanje koje rješava |
|------|----------------------|
| **ledger** | Što je u knjigama PDV-a prije obrasca? |
| **forms** | Kako generirati službeni obrazac? |
| **submission** | Kako evidencirati predaju na ePoreznu? |
| **validation** | Je li obrazac formalno ispravan prije predaje? |
| **verification** | Je li ERP payload usklađen s predanim XML-om? |

**Verification vs reconciliation:** u ERP-u „reconciliation" često znači bank/GL usklađenje. Tax sloj koristi **verification** za usporedbu generiranog obrasca s predanim artefaktom.

---

## Submission — samo facade

`domains/tax/submission/` **ne smije** sadržavati vlastitu implementaciju audit logike.

```python
# domains/tax/submission/__init__.py
from accounting.services.submission.service import SubmissionService
```

Jedan servis, jedan audit trag — [`ADR-0009`](../architecture/ADR-0009-submission-module-v1.md).

---

## Implementacijski redoslijed (Sprint 3)

Vidi [`FORM_IMPLEMENTATION_CONVENTION.md`](FORM_IMPLEMENTATION_CONVENTION.md).

1. Import službene ZP XSD → `accounting/schemas/zp/`
2. `tax_forms/zp/` — payload → aggregate → build → render → parse
3. validation → verify
4. Snapshot testovi (fixtures)
5. Submission (`ZPReturn` + `SubmissionService`)
6. Cross-form testovi (PDV 101/103)

**Sprint 3 završen (2026-07-10):** ZP lifecycle + PDV 610+/RC/OSS — vidi [`ADR-0014`](../architecture/ADR-0014-tax-domain-completion.md).

---

## Standardni layout novih obrazaca

Svaki novi obrazac:

```text
accounting/tests/fixtures/tax/<form>/
accounting/services/tax_forms/<form>/
    aggregate.py | payload.py | build.py | render.py | parse.py
    validation.py | verify.py
```

PDV ostaje frozen (legacy `diff.py` / `integrity.py`). Detalji: [`FORM_IMPLEMENTATION_CONVENTION.md`](FORM_IMPLEMENTATION_CONVENTION.md).

---

## TaxFormEngine — konvencija (Protocol)

Novi obrazac implementira zajednički ugovor u `domains/tax/forms/protocol.py`. **Nije** novi arhitekturni servis.

```python
class TaxFormEngine(Protocol):
    form_code: str

    def generate(self, period) -> TaxFormDocument: ...
    def validate(self, document) -> ValidationResult: ...
    def render_xml(self, document) -> bytes: ...
    def render_pdf(self, document) -> bytes: ...
    def parse(self, xml_bytes: bytes) -> TaxFormDocument: ...
```

`submit()` i `history()` **delegiraju** na `SubmissionService` — nikad vlastiti audit model.

Referentna implementacija: [`accounting/services/tax_forms/pdv/`](../accounting/pdv-obrazac-architecture.md).

---

## Canonical Artifact Rule

Svaki obrazac za predaju nasljeđuje PDV princip (vidi [`FORM_REGISTRY.md`](FORM_REGISTRY.md)):

- `payload.json` = SSOT po verziji
- XML i PDF = izvedeni artefakti
- Submission = `payload_hash` iz kanonskog payloada

---

## JOPPD — builder pattern (planned candidate v1.5)

JOPPD je **Tax obrazac**; HR je samo jedan izvor podataka.

```text
domains/hr/                  → Employee (izvor)
domains/tax/forms/joppd/     → payload, render, parse
domains/tax/joppd_builder/   → agregacija DTO iz HR + Finance + Manual
```

Status: L0 stub. Potvrda scopea nakon Sprint 3 gate — vidi FORM_REGISTRY.

---

## Migracija koda

Inkrementalno (ADR-0010):

1. Novi kod → `domains/tax/*`
2. Postojeći PDV pipeline ostaje u `accounting/services/tax_forms/pdv/` (frozen)
3. Facade re-exporti dok traje migracija

---

### Operativni dokumenti po obrascu

Svaki novi obrazac dobiva isti tip dokumentacije (ne ADR):

| Obrazac | Dokument |
|---------|----------|
| PDV | [`pdv-obrazac-architecture.md`](../accounting/pdv-obrazac-architecture.md) (frozen) |
| ZP | [`ZP_ARCHITECTURE.md`](ZP_ARCHITECTURE.md) |
| JOPPD | TBD (`JOPPD_ARCHITECTURE.md`) |

Layout i redoslijed implementacije: [`FORM_IMPLEMENTATION_CONVENTION.md`](FORM_IMPLEMENTATION_CONVENTION.md).

Test fixtures (prije implementacije): `accounting/tests/fixtures/tax/{form}/` — vidi [`tax/zp/README.md`](../../erp/app/accounting/tests/fixtures/tax/zp/README.md).

---

## Linkovi

- [FORM_REGISTRY.md](FORM_REGISTRY.md) — SSOT svih obrazaca
- [EPOREZNA_SUBMISSION_MATRIX.md](EPOREZNA_SUBMISSION_MATRIX.md) — XSD/G2B/Portal matrica + G2B research backlog
- [FORM_IMPLEMENTATION_CONVENTION.md](FORM_IMPLEMENTATION_CONVENTION.md) — standardni layout obrazaca
- [TESTING_GUIDE.md](TESTING_GUIDE.md) — obvezni testni paket po obrascu
- [ZP_ARCHITECTURE.md](ZP_ARCHITECTURE.md) — Sprint 3 (feature) operativni doc
- [DOMAIN_MAP.md](../architecture/DOMAIN_MAP.md) — maturity po domeni
- [pdv-obrazac-architecture.md](../accounting/pdv-obrazac-architecture.md) — PDV frozen pipeline
