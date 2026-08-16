# Storno u računovodstvenim izvještajima

Računovodstveni sloj koristi **dva različita pogleda** na iste temeljnice:

| Pogled | Svrha | Tipični izvještaji |
|--------|-------|-------------------|
| **NET** | Trenutno važeće poslovno stanje | bruto bilanca, kumulativna bruto bilanca, bilanca, RDG, Excel izvozi |
| **AUDIT** | Potpuna povijest knjiženja | dnevnik knjiženja, buduća kartica konta, audit export |

Implementacija: `erp/app/accounting/reporting/query.py` (`ReportMode`, `reporting_lines_qs`, `reporting_entries_qs`).

Vidi i: [manual-journal-bank-matching.md](manual-journal-bank-matching.md) za ručne temeljnice i usklađivanje s bankovnim transakcijama.

**Ne vraćati se na jedinstveni filter `status='posted'`** — to u financijskim izvještajima ostavlja artefakte storno parova na sintetičkim kontima.

---

## 1. Model storna

Storno se izvršava metodom `JournalEntry.reverse()` (`accounting/models.py`).

| Zapis | Status | `reversed_entry` | Uloga |
|-------|--------|------------------|-------|
| Originalna temeljnica | `reversed` | `NULL` | Povijesni zapis — više ne ulazi u neto agregaciju |
| Storno temeljnica | `posted` | → original | Obrnute stavke (D↔K); isključena iz neto agregacije |

Tok:

1. Korisnik stornira knjiženu temeljnicu (`status='posted'`).
2. Original prelazi u `status='reversed'`.
3. Kreira se nova temeljnica s brojem `{original}-ST`, statusom `posted` i FK `reversed_entry` na original.
4. Stavke storna imaju zamijenjene iznose duguje/potražuje.

`draft` temeljnice ne ulaze ni u jedan izvještaj.

---

## 2. NET izvještaji

Koriste `ReportMode.NET`.

**Filter na razini temeljnice:**

```text
status = 'posted'
AND reversed_entry IS NULL
```

Time se isključuju:

- originali čiji je status `reversed`
- storno temeljnice (`reversed_entry IS NOT NULL`)

Rezultat je **trenutno važeće poslovno stanje** — ono što korisnik očekuje od bruto bilance, bilance i RDG-a.

Izvještaji koji prolaze kroz ovaj filter (`accounting/services/reports.py`):

- `trial_balance` / `trial_balance_cumulative`
- `balance_sheet` (bilanca)
- `income_statement` (RDG)
- `export_trial_balance_xlsx`, `export_bilanca_xlsx`, `export_rdg_xlsx`

---

## 3. AUDIT izvještaji

Koriste `ReportMode.AUDIT`.

**Filter:**

```text
status IN ('posted', 'reversed')
```

Uključuje aktivne temeljnice, stornirane originale i storno temeljnice. Svaki zapis ima eksplicitnu audit oznaku (`EntryAuditKind`):

| Oznaka | Uvjet | Značenje |
|--------|-------|----------|
| **Aktivno** | `posted`, bez `reversed_entry` | Važeće knjiženje |
| **Stornirano (original)** | `status='reversed'` | Original koji je poništen stornom |
| **Storno** | `posted` + `reversed_entry IS NOT NULL` | Storno temeljnica koja poništava original |

Dnevnik knjiženja (`journal_report`, Excel export) koristi audit pogled i prikazuje stupac **Tip** s ovim oznakama.

Audit izvještaj **namjerno ne skriva** storno parove — svrha mu je revizijski trag, ne poslovno stanje.

---

## 4. Poznato ponašanje

### Datum storna

Storno temeljnica dobiva `entry_date = timezone.now().date()` — datum **izvršenja storna**, ne datum originalne temeljnice.

Primjer:

- original knjižen u lipnju,
- storno izvršen u srpnju,

→ u lipanjskom mjesečnom dnevniku vidljiv je samo stornirani original; storno temeljnica pojavljuje se u srpanjskom dnevniku.

To je legitimno za revizijski trag, ali korisnik može biti iznenađen ako očekuje oba zapisa u istom mjesečnom izvještaju. **Namjerno nije mijenjano** u implementaciji neto/audit pogleda — promjena datuma storna zahtijevala bi zaseban dizajnski odluku.

### Što nije u opsegu ovog dokumenta

- **Orphan storno** (podatkovni problem bez valjanog para) — rješava se zasebnim maintenance taskom, ne reporting logikom.
- **PDV knjiga, kartica konta** — mogu kasnije usvojiti isti `ReportMode` pattern; trenutno nisu refaktorirani u ovom ticketu.

---

## Referenca u kodu

```
erp/app/accounting/reporting/
  __init__.py
  query.py          # ReportMode, EntryAuditKind, reporting_*_qs

erp/app/accounting/services/reports.py   # financijski izvještaji (NET)
erp/app/accounting/views.py              # journal_export (AUDIT + Tip)
erp/app/accounting/tests.py              # ReportingStornoTests
```
