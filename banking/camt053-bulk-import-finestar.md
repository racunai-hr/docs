# CAMT.053 bulk import — Fine Star preduvjet

Jednokratna ručna priprema prije prvog uvoza OTP izvoda u Fine Star adminu.

## BankAccount

U Django adminu za tenant **finestar** kreirajte (ili provjerite) bankovni račun:

| Polje | Vrijednost |
|-------|------------|
| Naziv računa | npr. *Fine Star EUR — OTP* |
| Banka | OTP banka |
| IBAN | `HR6124070001100204771` |
| Valuta | `EUR` |
| Status | `active` |

Bez ovog zapisa bulk import CAMT.053 datoteke vraća grešku *Nepoznat IBAN* za svaki `<Stmt>` s tim IBAN-om.

## Import u adminu

1. Otvorite **Bankovni izvodi** na tenant adminu (npr. `https://finestar.racunai.hr/admin/`).
2. Kliknite **Uvezi bankovni izvod** na changelistu.
3. Odaberite OTP CAMT.053 XML (npr. `BCS.*.xml` iz OTP e-bankinga).
4. Pregledajte flash poruke: uspjeh, upozorenja (salda, fallback hash) i greške po izvodu.

## Napomena

- Auto-kreiranje `BankAccount` iz XML-a **nije** uključeno u v1.
- Više `<Stmt>` elemenata u jednoj datoteci mapira se na odgovarajući račun po IBAN-u unutar istog tenant-a.
