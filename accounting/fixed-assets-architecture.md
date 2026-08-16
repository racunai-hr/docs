# Osnovna sredstva — arhitektura (issue #6)

> **6.1 BASELINE (zamrznuto 2026-07-03):** model, admin, `create_fixed_asset_from_purchase()`. Daljnje promjene samo bugfix. Sljedeće: **6.3 aktivacija**.

Vidi: [uvoz-vozila-knjizenje.md](uvoz-vozila-knjizenje.md) · [chart-of-accounts-rules.md](chart-of-accounts-rules.md) · GitHub [#5](https://github.com/avrcanio/racunai.hr/issues/5) · [#6](https://github.com/avrcanio/racunai.hr/issues/6)

---

## 6.1 baseline verification (2026-07-03)

Prvi `FixedAsset` u sustavu — **VW T-Cross**, id **2**, tenant `finestar`.

| Provjera | Očekivano | Rezultat |
|----------|-----------|----------|
| Naziv | VW T-Cross | ✅ |
| VIN | `WVGZZZC1ZPY022544` | ✅ |
| Status | `in_preparation` | ✅ |
| Nabavna vrijednost | 8.000 € | ✅ |
| Knjigovodstvena vrijednost | 8.000 € | ✅ |
| `purchase_journal_entry` | #49 (`202605-0011`) | ✅ |
| Filter admin `IN_PREPARATION` | prikazuje T-Cross | ✅ |
| Reverse lookup JE → asset | 1 kartica | ✅ |

Kreirano idempotentno: `create_fixed_asset_from_purchase()` na JE #49.

---

## FixedAsset = računovodstveni centar

Kartica osnovnog sredstva prikazuje nabavnu vrijednost, amortizaciju, knjigovodstvenu vrijednost, sve temeljnice i povijest.

### Polja u bazi (faza 6.1)

| Grupa | Polja |
|-------|-------|
| Identitet | `name`, `inventory_number`, `vin` (unique/tenant), `registration_plate` |
| Status | `in_preparation` \| `active` \| `disposed` |
| Porijeklo | `purchase` \| `opening_balance` |
| Nabava | `purchase_date`, `in_service_date`, `disposed_at`, `acquisition_cost` |
| Računovodstvo | FK: `construction_account`, `asset_account`, `accumulated_depreciation_account`, `depreciation_expense_account` |
| Veze | FK: `purchase_journal_entry`, `activation_journal_entry`, `disposal_journal_entry` |

**Konta = konfiguracija tenant-a** (FK na `ChartOfAccounts`), ne hardkod u kodu.

Vidi: [chart-of-accounts-rules.md](chart-of-accounts-rules.md)

### Izračunata svojstva (ne u bazi)

| Svojstvo | Izvor (faza 6.1) |
|----------|------------------|
| `accumulated_depreciation` | Agregacija iz `DepreciationSchedule` (knjižene temeljnice) |
| `current_book_value` | `acquisition_cost - accumulated_depreciation` |

### Pravilo: temeljnica nabave

Za `origin=purchase`, `purchase_journal_entry` je **obavezan**. Za `opening_balance` — kasnije (issue #1).

Servis `create_fixed_asset_from_purchase()` je **idempotentan**: ponovni poziv za istu temeljnicu vraća postojeću karticu.

---

## Referentni seed: T-Cross (JournalEntry #49)

| Polje | Vrijednost |
|-------|------------|
| Temeljnica | id **49**, broj `202605-0011` |
| Datum | 22.05.2026. |
| Knjiženje | D **0373** / K **1000**, 8.000,00 € |
| VIN | `WVGZZZC1ZPY022544` |
| Status kartice | `in_preparation` |

Fine Star konta:

| Uloga | Šifra |
|-------|-------|
| U pripremi (`construction_account`) | **0373** |
| Aktivirano vozilo (`asset_account`) | **032001** |
| Ispravak vrijednosti | **0393** |
| Trošak amortizacije | **4314** (ili 4332 — TBD) |

---

## Rent-a-car: Vehicle ≠ FixedAsset

```
Vehicle (rent-a-car, operativa)
        │
        │ OneToOne FK
        ▼
FixedAsset (računovodstvo)
```

**Vozilo za najam nije osnovno sredstvo** — koristi osnovno sredstvo.

| FixedAsset | Vehicle (v2+) |
|------------|---------------|
| računovodstvo | registracija |
| amortizacija | kilometraža |
| nabava / aktivacija | status dostupnosti |
| temeljnice (FK) | rezervacije |
| | osiguranje, servis |

Jedan `Vehicle` → jedan `FixedAsset`.

---

## Redoslijed implementacije

| Faza | Opis | Status |
|------|------|--------|
| **6.1** | Model, admin, `create_fixed_asset_from_purchase()` | ✅ **baseline** |
| **6.3** | `can_activate()` + `activate_fixed_asset()` — 0373 → 032001 | ✅ |
| **6.4** | `generate_monthly_depreciation()` + `post_depreciation()` | ✅ |
| — | Rent-a-car `Vehicle` model | nakon OS |

### Faza 6.3 — preduvjeti i atomska aktivacija

**`can_activate()`** (read-only, za admin UX prije klika):

- postoji `purchase_journal_entry` (knjižena)
- `status == in_preparation`
- postavljeni FK konta: `construction_account`, `asset_account`
- zadan `in_service_date` (datum aktivacije)

Vraća `(bool, list[str])` — admin prikazuje npr. „✔ Spremno za aktivaciju” ili „✖ Nedostaje …”.

**`activate_fixed_asset()`** — **jedna transakcija** (`transaction.atomic`):

1. poziva `can_activate()` — ako ne, `ValidationError`
2. generiranje `JournalEntry` (D 032001 / K 0373)
3. povezivanje `activation_journal_entry`
4. `status = active`, `in_service_date`
5. commit ili rollback svega zajedno

### Faza 6.4 — plan amortizacije i knjiženje

**Polja na `FixedAsset`:**

| Polje | Opis |
|-------|------|
| `useful_life_months` | Trajnost u mjesecima (obavezno za amortizaciju) |
| `depreciation_method` | `linear` (jedina podržana metoda) |
| `residual_value` | Ostatak vrijednosti na kraju (default 0) |

**Model `DepreciationSchedule`:** jedan zapis po imovini + razdoblje (`year`, `month`), unique constraint za idempotentnost.

**`generate_monthly_depreciation(asset, year, month)`** — izračun linearnog iznosa; vraća postojeći zapis ako već postoji.

**`post_depreciation(schedule, user)`** — knjiženje D trošak amortizacije (`depreciation_expense_account`, npr. **4314**) / K ispravak (**0393**); audit u `AuditLog`.

**`accumulated_depreciation`** — agregacija iz `DepreciationSchedule` s knjiženom temeljnicom.

**Periodični run:** Celery `accounting.run_monthly_depreciation` (1. u mjesecu 06:00) + `depreciate_fixed_assets --tenant --year --month [--dry-run]`.

**API:** `domains.assets.facade` (maturity L3); legacy re-export u `accounting.services.fixed_assets`.
