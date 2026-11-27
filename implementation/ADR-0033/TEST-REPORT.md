# ADR-0033: izvještaj o testiranju

Datum: 2026-11-27. Kod: `api` `8c982e8`, `app` `ed56bb2`.

## 1. Okruženje

Namjenskog lokalnog CI-ja za ovaj projekt nema. Korišten je izolirani lokalni ekvivalent: zaseban kontejner `postgis/postgis:16-3.4` na zasebnoj Docker mreži i `erp-django` slika s dodanim `pytest-xdist`. Produkcijska baza i produkcijski `.env` nisu korišteni. Automatizirani testovi ne zovu vanjske API-je.

## 2. Rezultati

| Provjera | Prije (netaknuti `develop`) | Poslije |
|----------|------------------------------|---------|
| `api` cijeli pytest | 1471 prolazi, 19 pada, 19 grešaka, 2 preskočena | **1558 prolazi**, 19 pada, 19 grešaka, 2 preskočena |
| Novi padovi | | **nijedan** (popisi padova uspoređeni po imenu) |
| FX moduli kroz `manage.py test` | | 86/86 |
| OpenAPI `--validate --fail-on-warn` | prolazi | prolazi; razlika samo dodaje |
| Import lint (warning) | 1 upozorenje | isto 1 upozorenje |
| `app` typecheck i `next build` | | prolazi |
| `app` vitest | 442 prolazi, 1 pada | **457 prolazi**, 1 pada |

### Zatečeni padovi (nisu uvedeni ovim radom i nisu istraženi)

- `api`, greške prikupljanja: `accounting/tests.py`, `banking/tests.py`, `super_integration/tests.py` (sudar imena s paketom `tests/`), `fiscal_gateway/tests` i `ubl/tests` (`pytest_plugins` u podređenom `conftest`). Ti moduli se u ovom pokretanju **nisu izvršili**.
- `api`, padovi: `tenants/tests.py` (5), `integrations/tests/test_inbound_eracun_sync.py` (7), `settings/tests.py` (2), `test_subledger_admin.py` (2), `test_zp_admin.py` (1), `test_banking_write_api.py` (1), `test_asset_purchase_remediation_write_path.py` (1). Mogući uzrok je minimalno testno okruženje; nije potvrđeno.
- `app`: `src/lib/dashboard/nav.test.ts`.

Test `lets an owner assign an account per expense line` jednom je pao tijekom razvoja i zatim prošao u više uzastopnih pokretanja; vjerojatno je osjetljiv na vrijeme.

## 3. Acceptance scenariji ADR-0033

`domains/finance/tests/test_fx_acceptance.py`, svih 15 prolazi.

| # | Scenarij | Test |
|--:|----------|------|
| 1 | F261166, 103.500 DKK uz 7,50 = 13.800 EUR | `test_01_f261166_dkk_at_confirmed_7_50` |
| 2 | EUR račun nepromijenjen | `test_02_eur_invoice_is_unchanged` |
| 3 | Bez tečaja: EUR nedostupan, knjiženje blokirano, zbroj nepotpun | `test_03_missing_rate_hides_eur_blocks_posting_and_marks_aggregate` |
| 4 | Predloženi ili izmijenjeni tečaj sljediv | `test_04_proposed_or_changed_rate_is_confirmed_and_traceable` |
| 5 | Nedostupan izvor, dokumentirani unos | `test_05_unavailable_source_still_allows_documented_rate` |
| 6 | Porezni tečaj odvojen od dobavljačeva | `test_06_tax_rate_differs_and_tax_record_never_uses_supplier_rate` |
| 7 | Zaokruživanje i uravnotežena temeljnica | `test_07_line_rounding_keeps_journal_balanced` |
| 8 | Pogrešno knjižen račun: prijava, ispravak stornom | `test_08_wrongly_booked_invoice_is_reported_not_rewritten` |
| 9 | Vikend/praznik | `test_09_weekend_only_allowed_prior_rate_with_visible_date` |
| 10 | Ponovljeni dohvat i izmjena izvora | `test_10_repeated_fetch_and_source_revision_keep_confirmed_history` |
| 11 | Dvije valute, obveza u EUR | `test_11_eur_obligation_with_informative_dkk_creates_no_foreign_payable` |
| 12 | Dvije valute, obveza nejasna | `test_12_two_amounts_require_explicit_obligation_currency` |
| 13 | Djelomična plaćanja | `test_13_partial_payments_allocate_and_realize_difference_once` |
| 14 | Vrednovanje otvorene obveze | `test_14_valuation_is_traceable_and_not_recognised_twice` |
| 15 | Naknada odvojena od razlike | `test_15_bank_fee_is_separate_from_fx_difference` |

Scenariji 9, 13, 14 i 15 provjeravaju pravila uz **testnu** konfiguraciju (dopušten raniji tečaj, odobrena konta). U zadanoj konfiguraciji oboje je zaključano, što je također testirano.

## 4. Pokrivenost po traženim vrstama

| Vrsta | Gdje | Broj novih |
|-------|------|-----------:|
| Parseri, povijest tečajeva, raspodjela centi | `accounting/tests/test_fx_rates.py` | 16 |
| Validacije, API, ovlasti, tenant izolacija, bez mreže, N+1, izvoz | `test_fx_confirmation_api.py` | 20 |
| Plaćanja, banka, storno, vrednovanje, zaključana razdoblja, audit | `test_fx_settlement.py` | 23 |
| Porezni preračun, RC, starija knjiženja, ostali strani izvori | `test_fx_tax_and_legacy.py` | 10 |
| Acceptance | `test_fx_acceptance.py` | 15 |
| Migracije (samo aditivne) | `test_fx_migrations.py` | 2 |
| Strani račun bez tečaja ne knjiži (postojeći modul) | `test_vehicle_inventory_cost.py` | 1 |
| Frontend | `DualAmount`, `ExpenseFxPanel`, `IncomingExpenseDetail` | 15 |

Regresija EUR računa: scenarij 2 i cijela postojeća suita, uključujući PDV, saldakonto i bankovne testove.

### Izmijenjeni postojeći testovi

Prilagođeni su jer ADR mijenja ponašanje: dva testa stranog PDV-a na zalihi vozila i jedan test pregleda knjiženja sada prvo potvrđuju tečaj; test KPI-ja potvrđuje tečaj umjesto oslanjanja na `Expense.fx_rate`; test „sama valuta ne otvara strani PDV” dodatno očekuje da pregleda nema prije potvrde.

## 5. Provjere izvan automatiziranih testova

- Migracije naprijed, natrag i ponovno naprijed nad postojećim retkom saldakonta u izoliranoj bazi: iznosi i status ostali isti, novi stupci `EUR` / `NULL`.
- Jedan ručni dohvat s javnih adresa u izoliranu bazu: ECB 29 valuta (dnevni i 90 dana), HNB 13 valuta; ponovljeni dohvat bez novih redaka; DKK 2. 11. 2026. ECB 7,473700, HNB 7,473500.

## 6. Nije provjereno

- Vizualni pregled sučelja u pregledniku i end-to-end tok sa živim API-jem.
- Istodobni zahtjevi za plaćanje u više dretvi (zaključavanja postoje, test ne).
- Nove migracije nad kopijom produkcijske baze i trajanje deploya.
- `audit_fx_documents` nad produkcijskim podacima (tablice tamo ne postoje).
- Dohvat pune ECB povijesti (`--mode history`).

## 7. GitHub CI

Sva pokretanja za commitove iz ovog izvještaja su uspješna.

| Repozitorij | Workflow | Događaj | Rezultat |
|-------------|----------|---------|----------|
| `api` | OpenAPI contract (`schema`) | [push](https://github.com/racunai-hr/api/actions/runs/46015137302) | uspješno |
| `api` | OpenAPI contract (`schema`) | [pull_request](https://github.com/racunai-hr/api/actions/runs/46015182351) | uspješno |
| `app` | OpenAPI TypeScript types (`types`) | [push](https://github.com/racunai-hr/app/actions/runs/46015143491) | uspješno |
| `app` | OpenAPI TypeScript types (`types`) | [pull_request](https://github.com/racunai-hr/app/actions/runs/46015185147) | uspješno |

GitHub CI repozitorija `api` provjerava samo OpenAPI ugovor i njegov inventar. Poslovni testovi izvršeni su lokalno (§2).
