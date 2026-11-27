# RFC — VIES connector

```text
Status: Accepted
Date: 2026-11-27
Related: ../architecture/ADR-0034-pdv-s-workflow.md
```

## Svrha

Provjera PDV identifikacijskih brojeva dobavljača prije finalizacije PDV-S (ADR-0034 D06).

## Sučelje

Javni SOAP servis Europske komisije `checkVatService`, operacija `checkVatApprox`. Adresa i uključenje: `VIES_URL`, `VIES_ENABLED`. Kod: `api/app/accounting/services/pdv_s/vies.py`.

## Podaci

Šalje se samo PDV ID dobavljača i PDV ID podnositelja kao tražitelja (radi referentne oznake upita). Nazivi, iznosi i dokumenti se ne šalju.

## Ponašanje

| Rezultat | Učinak |
|---|---|
| valjan | zapis s vremenom i referencom |
| nevaljan | traži dokumentiranu odluku računovođe; odbijanje blokira finalizaciju |
| nedostupan / nepoznat | upozorenje, ne blokira |

Vremensko ograničenje i ograničen broj ponavljanja. Rezultati se čuvaju uz verziju obrasca i ne brišu. Odgovor vrijedi za trenutak upita, nije dokaz povijesne valjanosti. `GB` nije pokriven; `XI` se provjerava.

## Odvojenost

Poslovne odluke ne ovise o dostupnosti servisa. Testovi koriste lažni prijenosni sloj i ne zovu vanjski servis.
