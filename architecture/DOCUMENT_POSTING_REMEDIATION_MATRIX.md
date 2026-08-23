# Remediation matrica — document / GL / subledger (Korak 2)

> **Status:** Korak 2a locked; Korak 2b (`asset_purchase` profil) implementiran u kodu + testovima. Bez storna/repostinga na finestar. Nema DB writeova, storna ni repostinga dok se ova matrica ne odobri.
>
> **Ulaz:** inventar `audit_document_gl_subledger_mismatch --tenant finestar` (2026-08-23) — 36 dokumenata, 1 mismatch: expense `T-2026-0009` (`P1+P2+P3+P6`).
>
> **Izvan scopea ovog dokumenta:** PFC integrity scan (zaseban Korak, nakon Hadžić remediation gatea).

---

## 1. Poslovna klasifikacija `T-2026-0009`

| Polje | Vrijednost |
|-------|------------|
| Dokument | Expense `T-2026-0009` (id=16), Automobile Hadžić (partner 17) |
| Iznos | 8.000,00 EUR, PDV 0, status `paid`, `settlement_method=business_account` |
| Poslovni događaj | **Nabava dugotrajne imovine (vozilo) u pripremi** — ne OPEX rashod |
| Dokaz | JE 49 `D 0373 / K 1000`; FixedAsset id=2 (VW T-Cross, `in_preparation`); opis VIN/račun/ZOU |
| Trenutna kategorija | `Ostalo` → `default_account=4120` (**pogrešno** za ovaj događaj) |

**Zaključak:** dokument = **`asset_purchase`** (CIP / imovina u pripremi na 0373), ne default `expense` rashod na 4120.

Napomena: stari playbook [`uvoz-vozila-knjizenje.md`](../accounting/uvoz-vozila-knjizenje.md) faza 1 dopuštao je **jednu** manual JE `D 0373 / K 1000`. To je bio privremeni shortcut. Kanonski put ubuduće ide kroz dokument + AP subledger (dolje).

---

## 2. Posting profil `asset_purchase` (cilj)

Poslovni događaj knjiži se kroz dokument (`post_document`), ne kroz ručnu temeljnicu.

```text
Expense (asset_purchase)
  → post_document('expense_approved')
       D 0373  (imovina u pripremi)     /  P 2201  (dobavljač)
       + SubledgerItem AP (partner = supplier)
  → bankovno zatvaranje / post_document('expense_paid') uz business_account
       D 2201  /  P 1000
       + SubledgerAllocation na istu AP stavku
  → BankTransaction match na payment JE (ne na bypass JE)
```

| Korak | Marker / event | GL | Subledger |
|-------|----------------|----|-----------|
| Obveza | `expense_approved` | D **0373** / P **2201** | kreira `SubledgerItem` (payable, open) |
| Plaćanje | `expense_paid` + `settlement_method=business_account` **ili** bank reconcile open-item | D **2201** / P **1000** | `SubledgerAllocation` |
| (kasnije) | aktivacija OS | D 032001 / P 0373 | FixedAsset — izvan ovog remediationa |

**PDV reverse charge (JE 57):** ostaje **zaseban tax događaj** (`D 14022 / K 24022`). Nije dio AP subledger puta; ne dirati u Hadžić AP remediation osim ako tax review eksplicitno zatraži.

### Implementacija profila (Korak 2b — locked)

Kanonski smjer: **eksplicitni `Expense.posting_profile`** određuje način knjiženja; konto je posljedica profila, ne uzrok.

1. `Expense.posting_profile = asset_purchase` (polje na dokumentu; default `opex`).
2. `PostingRule` za `expense_approved` s `condition.posting_profile = ["asset_purchase"]` → **D 0373 / P 2201** + `SubledgerItem`.
3. `ExpenseCategory.default_account` smije **samo** precizirati debitni konto unutar profila (npr. 0373), **ne** smije sam zaključivati vrstu dokumenta.
4. `expense_paid` + `settlement_method=business_account` → **D 2201 / P 1000** + alokacija (orthogonalno na profil).

**Ne:** `default_account=0373` ⇒ `asset_purchase`.  
**Da:** `posting_profile=asset_purchase` ⇒ debit 0373 (ili category override) + AP 2201.

---

## 3. Remediation matrica (po pattern grupi × poslovnom profilu)

Patterni ostaju **ne-ekskluzivni**; remediation se bira po **kombinaciji + vrsti događaja**, ne po jednom kodu.

### 3.1 Zabranjeno (sve profile)

| Zabranjeno | Zašto |
|------------|--------|
| Reaktivacija `cancelled` SubledgerItem (npr. id=28) | Kartica ≠ GL; dual truth |
| `SubledgerAllocation` na postojeći bypass JE (npr. JE 49) | AP na kartici bez 2201 u GL |
| Samo UI/kartica „popravak” | Krši Finance SSOT |
| Ponovno knjiženje na 4120 za ovaj dokument | Pogrešna poslovna vrsta (već jednom stornirano JE 99/109) |

### 3.2 Matrica

| Pattern kombinacija | Poslovni profil | Remediation (kanonski) | Ne raditi |
|---------------------|-----------------|------------------------|-----------|
| **P1+P2+P3+P6** (referent: `T-2026-0009`) | **`asset_purchase`** | Vidi §4 | Vidi §3.1 |
| P1 (+/− P4) bez P3, debit je rashod | `opex_expense` (4120) | Identificirati zašto je subledger cancelled; ako je GL obveza živa → rebuild AP + allocation; ako je GL storniran → kanonski repost kroz dokument | Cosmetic reactivate |
| P2+P3 bez bank match | `asset_purchase` ili opex (po kontu) | Storno bypass JE → post_document obveza (+ plaćanje ako paid) | Pretvoriti manual JE u „obvezu” bez 2201 |
| P6 sam | bilo koji s aktivnim AP | Match/alokacija na postojeći open item kroz bank reconcile | Kreirati drugi AP |
| P5 (orphan na **istom** source dokumentu) | — | Premjestiti/poništiti orphan subledger na krivom partneru; rebuild na kanonskom | Brisati bez GL konteksta |
| P7 | — | Zadržati jedan aktivni item; zatvoriti duplikat računovodstveno | Arbitrary keep-newest |
| P8 only | — | Nema remediation | — |

**P5 napomena iz inventara:** SubledgerItem 39 (partner 24) **nije** P5 na expense 16 — pripada `PrivateFundsClaim` → expense 25. Ide u **zaseban PFC scan**, ne u ovu matricu.

---

## 4. Konkretni plan za `T-2026-0009` (nakon locka — još uvijek nije GO za write)

### Stanje danas (read-only)

| Sloj | Stanje |
|------|--------|
| Dokument | `paid`, kategorija Ostalo/4120 |
| Aktivni krivi GL | **JE 49** posted: D 0373 / K 1000 (bypass AP) |
| Tax | JE 57 posted RC (zasebno) |
| Već poništeno | JE 99/100 reversed; storno 108/109; SubledgerItem 28 `cancelled` |
| Banka | BTX 18 matched → JE 49 |
| OS | FixedAsset 2 → `purchase_journal_entry=49` |

### Redoslijed (predloženo)

1. **Zaključaj profil** na dokumentu: kategorija / metadata = `asset_purchase` (0373), ne 4120.
2. **Poništi aktivni bypass:** storno/reverse **JE 49** (i unmatch BTX 18). *Ne* dirati cancelled item 28.
3. **Proizvedi kanonsku obvezu:** `post_document(..., 'expense_approved')` → D 0373 / P 2201 + novi `SubledgerItem`.
4. **Proizvedi kanonsko plaćanje:** `expense_paid` (business_account) **ili** bank open-item reconcile → D 2201 / P 1000 + `SubledgerAllocation`.
5. **Ponovno poveži banku:** BTX 18 → nova payment JE.
6. **FixedAsset:** preusmjeri `purchase_journal_entry` na novu **obvezu** (ili dogovorenu nabavnu JE) — ne ostavljati FK na stornirani JE 49.
7. **Verifikacija:** inventar više ne prijavljuje expense 16; partner 17 kartica pokazuje AP + zatvaranje; saldo 0373/1000/2201 konzistentan; FixedAsset 2 i dalje `in_preparation`.

### Acceptance (prije zatvaranja ticketа)

- [ ] Inventar: expense 16 nije u mismatch records (P8 ili izvan scan statusa prema pravilu)
- [ ] Partner 17 statement: vidi se nabava + bankovno zatvaranje za 8.000 €
- [ ] Nema `SubledgerAllocation` na JE 49
- [ ] SubledgerItem 28 ostaje `cancelled` (nije reaktiviran)
- [ ] JE 57 netaknut osim ako tax review kaže drugačije

---

## 5. Gateovi

| Gate | Što | Status |
|------|-----|--------|
| Korak 1 | Read-only inventar | ✅ gotov |
| **Korak 2a** | Lock matrice + profil `asset_purchase` | ✅ locked |
| Korak 2b | Implementacija profila (`Expense.posting_profile` + rules + tests) | ✅ done |
| Korak 2c | Dry-run remediation plan (`plan_expense_asset_purchase_remediation`, fail-closed, no writes) | ✅ done (BLOCKED: asset_purchase PostingRule not seeded on finestar) |
| Korak 2d | Storno/repost `T-2026-0009` na finestar | eksplicitni GO |
| Kasnije | PFC integrity scan (read-only) | nakon što Hadžić nije blocker |

---

## 6. Odluke koje treba eksplicitno potvrditi na reviewu

1. **`T-2026-0009` = `asset_purchase` (0373), ne OPEX 4120** — DA/NE  
2. **Kanonski put = dokument → 2201 → subledger → banka**, ne `D 0373 / K 1000` — DA/NE  
3. **Zabrana reaktivacije item 28 / allocation na JE 49** — DA/NE  
4. **Profil implementirati prvo preko ExpenseCategory `default_account=0373`** (postojeći hook) — DA/NE / alternativa  
5. **JE 57 (RC) ostaje izvan AP remediation** — DA/NE  
6. **FixedAsset.purchase_journal_entry** nakon remediation pokazuje novu obvezu JE — DA/NE
