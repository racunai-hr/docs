# Tax Form Registry — racunAI ERP

```text
Status: Living document (SSOT)
Last updated: 2026-07-10
Related: docs/tax/README.md, docs/architecture/DOMAIN_MAP.md
```

**Jedini izvor istine** za sve hrvatske porezne knjige, obrasce za predaju i njihovu zrelost u racunAI-u.

Ažurirati pri zatvaranju Tax milestonea ili dodavanju novog obrasca — **bez novog ADR-a** (konvencija unutar ADR-0010).

**Sprint 3 zatvoren (2026-07-10):** ZP **L2**, PDV 610+/RC/OSS mapiranje — vidi [`ADR-0014`](../architecture/ADR-0014-tax-domain-completion.md). Infrastructure freeze održan tijekom sprinta.

### Dokumentacija Tax domene

| Dokument | Uloga |
|----------|-------|
| [`README.md`](README.md) | Pregled domene + infrastructure freeze |
| [`FORM_REGISTRY.md`](FORM_REGISTRY.md) | SSOT obrazaca + maturity (ovaj dokument) |
| [`EPOREZNA_SUBMISSION_MATRIX.md`](EPOREZNA_SUBMISSION_MATRIX.md) | Kanali predaje (XSD/G2B/Portal) — **inicijalni paket završen** (plan v3) |
| [`FORM_IMPLEMENTATION_CONVENTION.md`](FORM_IMPLEMENTATION_CONVENTION.md) | Layout `tax_forms/<form>/` |
| [`TESTING_GUIDE.md`](TESTING_GUIDE.md) | Obvezni testni paket |
| [`ZP_ARCHITECTURE.md`](ZP_ARCHITECTURE.md) | Obrazac ZP (Sprint 3) |

---

## Legenda

### Maturity (L0–L5)

Vidi [`DOMAIN_MAP.md`](../architecture/DOMAIN_MAP.md).

| Razina | Značenje |
|--------|----------|
| L0 | Planirano — nema implementacije |
| L1 | Stub (`domains/tax/` paket, docs) |
| L2 | Djelomično — generator ili predaja, ne oboje produkcijski |
| L3 | Produkcijski spremno za trenutnu fazu |
| L4+ | Stabilno / enterprise |

### Stupci registryja (implementacija)

| Stupac | Opis |
|--------|------|
| **Source** | Odakle se agregiraju podaci (VATLedger, Builder, ručni unos, …) |
| **Return model** | Django model verzioniranog dokumenta (`VATReturn`, …) |
| **Canonical Artifact** | SSOT po verziji — obično `payload.json` (ADR-0007) |
| **Gen / Parse / Sub** | Generator / parser / submission u ERP-u |

### Stupci registryja (ePorezna — sažetak)

Detalji (Transport, Authentication, Response, Schema Version, …): [`EPOREZNA_SUBMISSION_MATRIX.md`](EPOREZNA_SUBMISSION_MATRIX.md).

| Stupac | Opis |
|--------|------|
| **XSD** | Službena shema u `ePorezna_Schemas.zip` |
| **G2B** | Strojna G2B predaja (✅ / Partial / ❌ / TBD) |
| **Channel** | Sažetak kanala: `XSD+Portal`, `XSD+G2B+Portal`, … |
| **Biz** | Poslovni prioritet racunAI: P0–P3 |
| **Gen** | Generate XML u ERP-u |
| **Sub** | Submit: `Manual` / `G2B` / `N/A` / `Planned` |

### Canonical Artifact Rule

Svaki obrazac za predaju **mora** slijediti isti princip kao PDV (ADR-0007):

1. **`payload.json` je SSOT** po verziji dokumenta
2. **XML je izvedeni artefakt** — generira se iz payloada; ručne izmjene nisu dopuštene
3. **PDF je izvedeni artefakt** — generira se iz payloada
4. **Submission koristi payload hash** — `SubmissionService.create_event()` bilježi SHA-256 kanonskog payloada

Novi obrazac bez `payload.json` kanona zahtijeva obrazloženje u PR-u i ažuriranje ovog registryja.

---

## Knjige (evidencije) — nisu obrasci za predaju

Porezne knjige generiraju se iz poslovnih dokumenata i temeljnica. Ne idu na ePoreznu kao zasebni XML obrasci.

| Knjiga | Biz | Maturity | Source | Return model | Canonical Artifact | Napomena |
|--------|-----|----------|--------|--------------|-------------------|----------|
| I-RA | P0 | L2 | Invoice, JE | — | — | Izlazni PDV; `VATLedgerEntry.LEDGER_I_RA` |
| U-RA | P0 | L2 | Expense, JE | — | — | Ulazni PDV; box 303 izvor; `LEDGER_U_RA` |
| VAT knjiga (agregat) | P0 | L2 | I-RA + U-RA | — | — | `generate_vat_ledger()` → `VATLedgerEntry` |

Implementacija: [`accounting/services/vat.py`](../../erp/app/accounting/services/vat.py). Ciljna putanja: `domains/tax/ledger/`.

---

## Obrasci za predaju — P0 (v1.x jezgra)

| Obrazac | Maturity | XSD | G2B | Channel | Source | Return model | Canonical Artifact | Gen | Parse | Sub | Napomena |
|---------|----------|-----|-----|---------|--------|--------------|-------------------|-----|-------|-----|----------|
| **PDV** | L3 | ✅ | ✅ | XSD+G2B+Portal | VATLedger | `VATReturn` | `payload.json` | ✅ | ✅ | Manual | Frozen ADR-0007; [`pdv-obrazac-architecture.md`](../accounting/pdv-obrazac-architecture.md) |
| **PDV-ispravak** | L3 | ✅ | ✅ | XSD+G2B+Portal | VATLedger | `VATReturn` | `payload.json` | ✅ | ✅ | Manual | Nova verzija + `SubmissionService.supersede()` |
| **PDV-S** | L2 | ✅ | Partial | XSD+G2B+Portal | VATLedger | `PDVSReturn` | `payload.json` | ✅ | ✅ | Manual | Inbound EU ledger |
| **ZP** | L2 | ✅ | TBD | XSD+Portal | VATLedger | `ZPReturn` | `payload.json` | ✅ | ✅ | Manual | Sprint 3 ✅ — [`ZP_ARCHITECTURE.md`](ZP_ARCHITECTURE.md), [`ADR-0014`](../architecture/ADR-0014-tax-domain-completion.md) |
| **JOPPD** | L0 | ✅ | ✅ | XSD+G2B+Portal | Builder | `JOPPDReturn` (TBD) | `payload.json` | Planned | — | Planned | Planned candidate v1.5 |

### JOPPD — planned candidate for v1.5

- **Nije** committed deliverable u [`ROADMAP.md`](../architecture/ROADMAP.md) (Faza 3 ostaje planirana lokacija)
- **Gate:** ✅ Sprint 3 završen — procjena opsega sada (vidi [`ADR-0014`](../architecture/ADR-0014-tax-domain-completion.md))
- **Dizajn:** `domains/tax/joppd_builder/` + `domains/tax/forms/joppd/`; HR (`Employee`) samo izvor podataka
- **Submission:** postojeći `SubmissionService` (ADR-0009)

---

## Obrasci — P1 (visoki prioritet)

| Obrazac | Maturity | XSD | G2B | Channel | Source | Return model | Canonical Artifact | Gen | Sub | Napomena |
|---------|----------|-----|-----|---------|--------|--------------|-------------------|-----|-----|----------|
| **PD** | L0 | ✅ | TBD | XSD+Portal | TBD | TBD | `payload.json` | Planned | Planned | Porez na dobit |
| **OSS** | L0 | ✅ | TBD | XSD+Portal | VATLedger | TBD | `payload.json` | Planned | Planned | One Stop Shop (e-trgovina EU) |
| **PDV-K** | L0 | ✅ | TBD | XSD+Portal | VATLedger | TBD | `payload.json` | Planned | Planned | [`pdv-extensions-roadmap.md`](../accounting/pdv-extensions-roadmap.md) §2 |

---

## Obrasci — P2 (ovisno o djelatnosti)

| Obrazac | Maturity | XSD | G2B | Channel | Source | Return model | Canonical Artifact | Gen | Sub | Napomena |
|---------|----------|-----|-----|---------|--------|--------------|-------------------|-----|-----|----------|
| **PPO** | L0 | ✅ | TBD | XSD+Portal | TBD | TBD | `payload.json` | Planned | Planned | Porez na potrošnju |
| **PZ 42/63** | L0 | TBD | TBD | Portal | Finance+Tax | — | — | Planned | Planned | Posebni postupci PDV |
| **PD-IPO** | L0 | ✅ | TBD | XSD+Portal | TBD | TBD | `payload.json` | Planned | Planned | Porez na dobit — IPO |
| **DONH** | L0 | ✅ | TBD | XSD+Portal | TBD | TBD | `payload.json` | Planned | Planned | Porez na dohodak (godišnja) |
| **DOH** | L0 | ✅ | TBD | XSD+Portal | TBD | TBD | `payload.json` | Planned | Planned | Porez na dohodak |
| **INO-DOH** | L0 | ✅ | TBD | XSD+Portal | TBD | TBD | `payload.json` | Planned | Planned | Inozemni dohodak |
| **EPOM** | L0 | ✅ | TBD | XSD+Portal | TBD | TBD | `payload.json` | Planned | Planned | Evidencija prometa gotovinom |
| **PPN** | L0 | ✅ | TBD | XSD+Portal | TBD | TBD | `payload.json` | Planned | Planned | **Porez na promet nekretnina** |
| **OPZ-STAT-1** | L0 | ✅ | TBD | XSD+Portal | TBD | TBD | `payload.json` | Planned | Planned | Statistički obrazac (OPZ) |

---

## Obrasci — P3 (niži prioritet)

| Obrazac | Maturity | XSD | G2B | Channel | Source | Return model | Canonical Artifact | Gen | Sub | Napomena |
|---------|----------|-----|-----|---------|--------|--------------|-------------------|-----|-----|----------|
| **Preknjiženja** | L0 | TBD | TBD | Portal | Finance JE | — | — | Planned | Planned | Cross-cutting Finance + Tax |
| PP-MI-PO | L0 | TBD | TBD | TBD | TBD | TBD | `payload.json` | Planned | Planned | |
| PD-NN | L0 | TBD | TBD | TBD | TBD | TBD | `payload.json` | Planned | Planned | |
| DPD | L0 | TBD | TBD | TBD | TBD | TBD | `payload.json` | Planned | Planned | |
| Izjava o neaktivnosti | L0 | TBD | TBD | TBD | TBD | TBD | `payload.json` | Planned | Planned | |
| MGP-PD1 | L0 | TBD | TBD | TBD | TBD | TBD | `payload.json` | Planned | Planned | |
| SR | L0 | TBD | TBD | TBD | TBD | TBD | `payload.json` | Planned | Planned | |
| TZ1 | L0 | TBD | TBD | TBD | TBD | TBD | `payload.json` | Planned | Planned | |
| PD-PO | L0 | TBD | TBD | TBD | TBD | TBD | `payload.json` | Planned | Planned | |
| INO-IZJAVA | L0 | TBD | TBD | TBD | TBD | TBD | `payload.json` | Planned | Planned | Inozemna izjava |

---

## Cross-cutting (nije obrazac u `forms/`)

| Funkcionalnost | Biz | Domena | Napomena |
|----------------|-----|--------|----------|
| e-trgovina | P1 | Tax + Integration | OSS/IOSS u PDV; eventualno ZP |
| Preknjiženja | P3 | Finance | JE workflow — vidi P3 tablicu |
| PZ 42 / PZ 63 | P2 | Finance + Tax | Knjiženje + PDV mapiranje (610+ kontekst) |

---

## Životni ciklus (svi obrasci)

Isti ciklus bez obzira na obrazac — implementacija preko **TaxFormEngine** konvencije ([`README.md`](README.md)):

```text
1. generate()        — agregacija izvora → payload (jednom)
2. validate()        — XSD + poslovna pravila
3. render_xml/pdf()  — izvedeni artefakti iz payloada
4. pregled           — admin / API read-only
5. verzioniranje     — Return model po razdoblju (1:N)
6. submit()          — SubmissionService.create_event()
7. audit             — payload_hash + SubmissionEvent
8. history()         — lanac eventa, supersede za ispravak
9. ispravak          — nova verzija dokumenta + novi submission event
```

---

## Mapiranje na kod (trenutno stanje)

| Sloj | Putanja (privremeno) | Cilj |
|------|----------------------|------|
| Ledger | `accounting/services/vat.py` | `domains/tax/ledger/` |
| Forms | `accounting/services/tax_forms/` (`pdv/`, `pdv_s/`, **`zp/`**) | `domains/tax/forms/` |
| Submission | `accounting/services/submission/` | `domains/tax/submission/` (re-export) |
| Validation | `tax_forms/*/validation.py` | `domains/tax/validation/` |
| Verification | `pdv/diff.py`, `verify_pdv_period`; **`zp/verify.py`**, `verify_zp_period` | `domains/tax/verification/` |

---

## Reference

- [Tax domain README](README.md)
- [ePorezna Submission Matrix](EPOREZNA_SUBMISSION_MATRIX.md)
- [ADR-0007 PDV](../architecture/ADR-0007-pdv-module.md)
- [ADR-0009 Submission](../architecture/ADR-0009-submission-module-v1.md)
- [PDV extensions roadmap](../accounting/pdv-extensions-roadmap.md)
