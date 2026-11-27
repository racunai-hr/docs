<!-- generated from pdv-eu-source-matrix.yaml — do not edit manually; run: python -m accounting.services.tax_forms.pdv.source_matrix -->

# PDV EU — source matrix

Strojno čitljiv izvor: [`pdv-eu-source-matrix.yaml`](pdv-eu-source-matrix.yaml).
Ovaj dokument se generira iz YAML-a; ne uređivati ručno.

## Metapodaci

| Polje | Vrijednost |
| --- | --- |
| Verzija | `2.0.0` |
| Status | **Locked** |
| Vlasnik | Accounting |
| Zadnji pregled | 2026-11-27 |

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

- **Izvor:** `tax_fact_or_journal`
- **Boxovi:** `205`, `206`, `207`, `305`, `306`, `307`
- **Primjer:** Sintetički dobavljač DE, 8.000 EUR neto, stjecanje po stopi 25 %
- **Stopa PDV:** 25%
- **Konta:**
  - `asset`: `0373`
  - `obveza`: `24020-24022`
  - `pretporez`: `14020-14022`
- **Napomena:** Izvor je potvrđena porezna činjenica (vrsta isporuke, porezni datum, postupak, stopa) ili temeljnica na EU kontima. Stopa određuje polje: 5 % II.5/III.5, 13 % II.6/III.6, 25 % II.7/III.7. Konto 0373 bez EU retka u istoj temeljnici nije dokaz stjecanja i traži pregled. Mapiranje usklađeno sa službenim opisom elemenata XML sheme PDV v11.0 (2026-11-27) uz autorizaciju vlasnika; zasebna potvrda računovođe još nije evidentirana.

### `EU_SERVICES_RECEIVED` — Primljene usluge iz EU

- **Izvor:** `tax_fact_or_journal`
- **Boxovi:** `208`, `209`, `210`, `308`, `309`, `310`
- **Stopa PDV:** 25%
- **Konta:**
  - `obveza`: `24030-24032`
  - `pretporez`: `14030-14032`
- **Napomena:** 5 % II.8/III.8, 13 % II.9/III.9, 25 % II.10/III.10. Porezni datum je dan obavljanja usluge.

### `DOMESTIC_REVERSE_CHARGE` — Tuzemni prijenos porezne obveze (primljene isporuke)

- **Izvor:** `manual_journal_entry`
- **Boxovi:** `204`, `304`
- **Stopa PDV:** 25%
- **Konta:**
  - `obveza`: `2401`
  - `pretporez`: `1401`
- **Napomena:** II.4 i III.4. Ne ulazi u PDV-S. OSS, prodaja na daljinu, elektroničko sučelje i IOSS nemaju automatsko polje na Obrascu PDV i traže računovodstveni pregled.

## Rasponi konta (EU)

| Ključ | Raspon |
| --- | --- |
| `domestic_reverse_charge_obveza` | `2401-24011` |
| `domestic_reverse_charge_pretporez` | `1401-14011` |
| `eu_goods_obveza` | `24020-24022` |
| `eu_goods_pretporez` | `14020-14022` |
| `eu_services_obveza` | `24030-24032` |
| `eu_services_pretporez` | `14030-14032` |

## Invarijanti

| ID | Pravilo |
| --- | --- |
| `I1` | `207.base >= 0` |
| `I2` | `207.vat == round(207.base * 0.25, 2); 206: 0.13; 205: 0.05` |
| `I3` | `207.vat == 307.vat and 207.base == 307.base` |
| `I4` | `400 unchanged when 207/307 net zero (RC neutral)` |

## Dedup pravila

| rule_id | event_id | action | Napomena |
| --- | --- | --- | --- |
| `EU_SUPPLIER_ZERO_VAT_EXCLUDE_303` | `EU_GOODS_ACQUISITION` | `exclude_from_box_303` | Račun EU dobavljača s 0% PDV (isporuka u RH) ne smije ulaziti u box 303 kao domaći pretporez. |
| `JE_EU_VAT_SKIP_EXPENSE_DUP` | `EU_GOODS_ACQUISITION` | `skip_if_expense_source` | Ako isti događaj već proizlazi iz expense modula, preskoči duplikat iz journal entry izvora. |

## Reference

- `docs/accounting/uvoz-vozila-knjizenje.md`
- `docs/tax/pdv-s/IMPLEMENTATION.md`
