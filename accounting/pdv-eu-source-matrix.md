<!-- generated from pdv-eu-source-matrix.yaml — do not edit manually; run: python -m accounting.services.tax_forms.pdv.source_matrix -->

# PDV EU — source matrix

Strojno čitljiv izvor: [`pdv-eu-source-matrix.yaml`](pdv-eu-source-matrix.yaml).
Ovaj dokument se generira iz YAML-a; ne uređivati ručno.

## Metapodaci

| Polje | Vrijednost |
| --- | --- |
| Verzija | `1.2.0` |
| Status | **Locked** |
| Vlasnik | Accounting |
| Zadnji pregled | 2026-07-10 |

## Procesni ugovor (status)

| Status | Značenje | Tko | Što je dopušteno |
| --- | --- | --- | --- |
| **Draft** | Radna verzija | Developer | Slobodne izmjene sadržaja, strukture, `event_id`, box mapiranja |
| **Reviewed** | Pregledano | Developer + računovođa | Manje korekcije (tipfelere, konta, stopa, dedup); nema promjene poslovne logike bez reviewa |
| **Locked** | Poslovna semantika zaključana | Računovođa potvrda | Svaka promjena zahtijeva semver bump, ažuriranje `last_reviewed`, novi review računovođe i regression testove prije mergea koda koji ovisi o promjeni |

**Locked nije samo YAML flag** — kod (`implemented=True`, mapping, ledger) smije ovisiti o Locked matrixu.
PR-B ne mergea se dok matrix nije **Locked**.

### Pravila verzioniranja (`version`)

| Bump | Primjer | Kada |
| --- | --- | --- |
| **major** | `1.0.0` → `2.0.0` | Promjena poslovne semantike postojećeg `event_id`, promjena mapiranja boxova za postojeći događaj, promjena invarijanata koji utječu na izračun |
| **minor** | `1.0.0` → `1.1.0` | Dodavanje novog `event_id` ili novih boxova **bez** promjene postojećih događaja |
| **patch** | `1.0.0` → `1.0.1` | Ispravci opisa, komentara, primjera, dokumentacije — **bez** promjene semantike, mapiranja ili invarijanata |

## Događaji (`event_id`)

Stabilni identitet poslovnog pravila — **ne koristiti box kodove kao identitet događaja**.

### `EU_GOODS_ACQUISITION` — EU stjecanje dobara

- **Izvor:** `manual_journal_entry`
- **Boxovi:** `207`, `307`
- **Primjer:** Automobile Hadžić DE, račun 70025237, 8.000 EUR neto, 0% PDV (isporuka u RH)
- **Dobavljač (ref):** `DE229674882`
- **Stopa PDV:** 25%
- **Konta:**
  - `asset`: `0373`
  - `obveza`: `24022`
  - `pretporez`: `14022`
- **Napomena:** Nabava vozila iz EU. Osnovica i PDV obveza u box 207 (II.7): D 0373 (osnovica), K 24022 (PDV). Pretporez u box 307 (III.7): D 14022. Boxovi 610–615 su VIII.1 ispravak DI (faza B).

### `OSS_ECOMMERCE_SUPPLY` — OSS isporuka e-trgovina EU

- **Izvor:** `invoice_item`
- **Boxovi:** `215`
- **Primjer:** B2C web shop DE, 100 EUR neto + 19% DE PDV
- **Napomena:** InvoiceItem vat_procedure=oss. Strani PDV ne ulazi u box 400 ni 200. Ne mapira se u ZP boxove 101/103 (B2B isključivo).

### `EU_DISTANCE_SALE` — Prodaja na daljinu unutar EU

- **Izvor:** `invoice_item`
- **Boxovi:** `204`
- **Napomena:** InvoiceItem vat_procedure=eu_distance.

### `EU_ELECTRONIC_INTERFACE` — Isporuke putem elektroničkog sučelja

- **Izvor:** `invoice_item`
- **Boxovi:** `214`
- **Napomena:** InvoiceItem vat_procedure=eu_electronic.

### `IOSS_IMPORT` — IOSS uvoz — pretporez u drugoj državi

- **Izvor:** `expense_or_journal`
- **Boxovi:** `308`
- **Konta:**
  - `pretporez`: `14042`
- **Napomena:** Expense vat_procedure=ioss ili JE D 14042. Isključeno iz box 400. Ne preklapa se sa ZP (EU outbound B2B).

## Rasponi konta (EU)

| Ključ | Raspon |
| --- | --- |
| `eu_goods_obveza` | `24020-24022` |
| `eu_goods_pretporez` | `14020-14022` |
| `eu_services_obveza` | `24030-24032` |
| `eu_services_pretporez` | `14030-14032` |
| `ioss_pretporez` | `14040-14042` |

## Invarijanti

| ID | Pravilo |
| --- | --- |
| `I1` | `207.base >= 0` |
| `I2` | `207.vat == round(207.base * 0.25, 2)` |
| `I3` | `207.vat == 307.vat and 207.base == 307.base` |
| `I4` | `400 unchanged when 207/307 net zero (RC neutral)` |

## Dedup pravila

| rule_id | event_id | action | Napomena |
| --- | --- | --- | --- |
| `EU_SUPPLIER_ZERO_VAT_EXCLUDE_303` | `EU_GOODS_ACQUISITION` | `exclude_from_box_303` | Račun EU dobavljača s 0% PDV (isporuka u RH) ne smije ulaziti u box 303 kao domaći pretporez. |
| `JE_EU_VAT_SKIP_EXPENSE_DUP` | `EU_GOODS_ACQUISITION` | `skip_if_expense_source` | Ako isti događaj već proizlazi iz expense modula, preskoči duplikat iz journal entry izvora. |

## Reference

- `docs/accounting/uvoz-vozila-knjizenje.md`
- `erp/app/expenses/data/manual_supplier_map.py`
