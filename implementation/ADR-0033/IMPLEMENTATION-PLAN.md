# ADR-0033: analiza i plan implementacije

Status ADR-a: **Proposed** (nepromijenjen). Datum: 2026-11-27.
Odluka: [`ADR-0033`](../../architecture/ADR-0033-incoming-invoice-foreign-currency.md), commit `3b0392fe599bb2ccc39cb9b4e5cff32067bb1a08`.
Ovaj dokument je izvedbeni popis, nije drugi ADR. Povezano: [računovodstvena provjera](ACCOUNTING-REVIEW.md), [testovi](TEST-REPORT.md), [završni izvještaj](FINAL-REPORT.md).

## 1. Polazno stanje

| Repozitorij | Polazište | Napomena |
|-------------|-----------|----------|
| `api` | `develop` `92640ce` | `main` je po sadržaju jednak |
| `app` | `develop` `308712d` | jednako `main` |
| `docs` | `main` `3b0392f` | `develop` nema dva commita s ADR-0033 |
| `stack` | nije mijenjan | infrastrukturne izmjene nisu potrebne |

Razvoj je rađen u zasebnim git worktreeima, odvojeno od pokrenutog sustava.

## 2. Nalazi analize

### Putanje koje su mogle strani iznos proknjižiti ili prikazati kao EUR

| Mjesto | Ponašanje prije izmjene |
|--------|-------------------------|
| `accounting/services/fx.py::in_book_eur` | bez tečaja vraća nepretvoreni broj |
| `posting.py::_get_amount`, stavke, RC PDV | koristi `in_book_eur` |
| `subledger.py::_document_total` | isto; saldakonto u stranim jedinicama |
| `vat.py::generate_vat_ledger` | `expense.amount` ide u PDV knjigu bez obzira na valutu |
| `tax_shadow/adapters.py` | valuta se prenosi, klasifikator je ne provjerava |
| `bank_settlement.py`, `banking/book.py` | bankovni iznos bez `amount_eur` knjiži se kao EUR |
| privatna sredstva, kaucije, predujmovi, kapitalizacija vozila | iznos se uzima kao EUR |
| KPI liste dokumenata | zbroj kaucija bez preračuna |
| frontend | izvorna valuta kao glavni iznos; EUR temeljnice u pregledu označene stranom valutom |

### Tečajevi

Nije postojao dohvat ni povijest. Postojao je samo zakonski tečaj za HRK i ručni unos u pregledu stranog PDV-a (`Expense.fx_rate`, bez zapisa tko je i kada potvrdio).

### Postojeći zapisi

Postojeći strani zapisi pregledani su samo za čitanje. Detalji produkcijskih podataka namjerno nisu u ovom javnom repozitoriju. Za sustavni pregled nakon uvođenja služi naredba `audit_fx_documents`.

## 3. Dizajn

| Pojam ADR-a | Gdje je |
|-------------|---------|
| Izvorna valuta i iznos | postojeća `Expense.currency`, `amount`, `tax_amount` (značenje nepromijenjeno) |
| Valuta i iznos obveze, informativni iznos | `ExpenseFxConfirmation` (`obligation_*`, `informative_*`) |
| Knjigovodstveni i porezni tečaj | dvije odvojene potvrde (`purpose = accounting / tax`) |
| EUR vrijednost | snapshot `eur_gross / eur_vat / eur_net` na potvrdi |
| Valuta stvarnog plaćanja | `FxPayableSettlement.payment_currency` |
| Povijest tečajeva | `ExchangeRate` (globalno, verzionirano), `ExchangeRateFetch` |
| Devizni saldakonto | `SubledgerItem.currency`, `original_amount_foreign`, `open_amount_foreign`; postojeći iznosi ostaju EUR |

Jedina točka preračuna je `accounting/services/fx_conversion.py`. Knjiženje, saldakonto, PDV, read model i banka čitaju je i staju kad preračun nedostaje ili je zastario (iznos ili valuta promijenjeni nakon potvrde).

Potvrda je nepromjenjiva. Izmjena stvara novu verziju i moguća je samo dok nema knjiženja (porezna: dok nema poreznih zapisa ni RC redaka). Nakon toga ide storno i ponovno knjiženje.

## 4. Faze

| Faza | Isporučeno | Zaključano ili izostavljeno |
|------|-----------|------------------------------|
| FX-01 tečajevi | ECB i HNB, verzije, dnevni Celery zadatak s ponavljanjem, naredbe `fetch_fx_rates`, `ensure_fx_beat_schedule`; ručni i dokumentirani tečaj na potvrdi | raniji tečaj za vikend/praznik isključen (`FX_PRIOR_RATE_DAYS = 0`) |
| FX-02 model | potvrde, valuta obveze, validacije, revizijski trag | EUR dokument sa stranom obvezom nije podržan |
| FX-03 sučelje | EUR primarno, panel tečaja, plaćanje u EUR, oznaka nepotpunog zbroja | storno plaćanja, vrednovanje, plaćanje u stranoj valuti i vezanje bankovne stavke samo kroz API; konta samo u Django adminu |
| FX-04 knjiženje | zaštite u svim nalazima iz §2, zatvorena fiskalna razdoblja, kompatibilan API | postojeće temeljnice nisu mijenjane |
| FX-05 plaćanja | djelomična i višestruka plaćanja, razlike, naknade, vrednovanje, usklađenje saldakonta i GK | knjiženje razlika traži odobrena konta; devizni predujmovi nisu podržani |

## 5. Redoslijed uvođenja (nije izvršen, traži zasebnu autorizaciju)

1. Architecture Review i odluke iz [ACCOUNTING-REVIEW](ACCOUNTING-REVIEW.md).
2. RFC i Issue za novi vanjski izvor (ECB, HNB) prema `ARCHITECTURE_GOVERNANCE.md`.
3. Merge `api`, zatim `app`.
4. Deploy: migracije `accounting.0045` i `expenses.0028` (aditivne).
5. `fetch_fx_rates --source ECB --mode history`, `fetch_fx_rates --source HNB --mode date --date …` za potrebne dane, `ensure_fx_beat_schedule`.
6. `audit_fx_documents` (read-only) i pregled nalaza.
7. Odobrenje konta u adminu tek nakon računovodstvene odluke.
8. Račun 547: zaseban postupak uz autorizaciju.

Povratak: kod se vraća na prethodnu verziju; nove tablice i stupci mogu ostati. Unmigracija je provjerena, ali briše potvrde i devizna plaćanja ako postoje.
