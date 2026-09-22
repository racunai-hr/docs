# Mjesta troška (MT)

Upravljačka dimenzija za RDG. Nije konto i nije `AnalyticAccount`.

## Kanonski trag

| Sloj | Uloga |
|------|--------|
| Source dokument (`Expense`, `Invoice`, `OfficialDocument`) | Input za resolver |
| `Vehicle.cost_center` / `ExpenseCategory.default_cost_center` / `FixedAsset.cost_center` | Fallback input |
| `JournalEntryLine.cost_center` | Kanonski trag nakon knjiženja |

Promjena MT-a na dokumentu **ne** mijenja već proknjižene stavke — `JournalEntryLine.cost_center` se ne čita ponovo s dokumenta. `save()` na `Expense` / `OfficialDocument` / `Invoice` odbija izmjenu nakon knjiženja. Ispravak: storno + ponovno knjiženje.

## Resolver

Prioritet: `source.cost_center` → `source.vehicle.cost_center` → `source.category.default_cost_center` → `None`.

MT se stavlja samo na konta klasa **4, 5, 7**. Prazan `account_class` pada na `account_code[0]`; nepoznata klasa = bez MT (fail-closed).

## Šifarnik (preset `hospitality`)

Grupe nisu knjižive. Seed:

```bash
docker exec racunai_django python manage.py seed_cost_centers --tenant finestar --preset hospitality
```

Objektna vozila (`701`, `702`, …) se vežu ručno na `Vehicle.cost_center`.

## Kartica MT i dokumenti na imovini

Dva izvora ostaju odvojena:

- **Kartica MT** (`/izvjestaji/mjesta-troska/{id}`) prikazuje knjižene RDG stavke i dokumente čiji posted JE ima `JournalEntryLine.cost_center`. Nabava i PPMV na 0373 nisu RDG i ne ulaze na tu karticu.
- **Kartica imovine → Dokumenti** prikazuje operativni identitet: `Expense.vehicle` (preko `Vehicle.fixed_asset`) i `OfficialDocument.related_fixed_asset`.

Dok rashod nije proknjižen s tim MT, kartica MT može biti prazna iako tab Dokumenti na OS već pokazuje PPMV odluke.

## Izvan v1

Prevaljivanje po ključevima, MT po stavci računa, obavezni MT po kontu, budžet.
