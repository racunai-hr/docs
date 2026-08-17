# PDV obrazac — vodič za računovođe

Operativni vodič za mjesečni Obrazac PDV u racunAI adminu. Tehnička arhitektura: [`pdv-obrazac-architecture.md`](pdv-obrazac-architecture.md).

## Preduvjeti

- Tenant ima ispunjene **Postavke tvrtke** (`CompanySettings`): OIB, normalizirana adresa, **Porezna ispostava** (`TaxOffice`).
- Odgovorna osoba (računovođa) postavljena u postavkama s ulogom `accountant`.
- Računi, troškovi i temeljnice za razdoblje su knjiženi u ERP-u.

## Mjesečni workflow

```text
1. Generiraj PDV ledger
2. Provjeri kontrolne preglede (admin / PDV-S export)
3. Generiraj draft PDV obrasca
4. Preuzmi unsigned XML → potpiši (Finin cert / ePorezna)
5. Predaj na ePoreznu (PDV + PDV-S)
6. Označi predano u ERP-u (portal UUID + payload_hash)
7. Priloži potvrdu (XML/PDF/…)
8. (Opcionalno, Alati) Upload potpisani XML ili Reconciliation
```

### Korak 1 — Generiraj PDV ledger

**Admin → Accounting → PDV razdoblja** → odaberi razdoblje → akcija **Generiraj PDV knjige** (admin labela; to je ledger, ne zakonska knjiga — vidi [`ADR-0019`](../architecture/ADR-0019-tax-classification-engine.md)).

Ledger se gradi iz:
- izlaznih računa → boxovi 201 / 202 / 203 prema stopi (I-RA kontrolni pregled),
- ulaznih troškova → box 303 (U-RA kontrolni pregled),
- stavki temeljnice (konto 1400, 24001*).

Ručne korekcije (`is_manual=True`) ostaju pri ponovnom generiranju.

Alternativa (CLI):

```bash
docker compose exec django python manage.py generate_vat_ledger \
  --tenant finestar --year 2026 --month 4 --replace
```

### Korak 2 — Provjeri kontrolne preglede

- Pregledaj **PDV ledger** (inline na razdoblju ili VATLedgerEntry admin) i I-RA/U-RA kontrolne preglede.
- Export **PDV-S XLSX** za ručnu provjeru (legacy; obrazac PDV ne ovisi o XLSX-u).
- Sažetak **PDV za uplatu** (Podatak400) prikazan u listi razdoblja.

### Korak 3 — Generiraj draft

Akcija **Generiraj draft PDV obrasca**.

ERP:
1. Agregira ledger po boxovima,
2. gradi `PdvPayload`,
3. renderira unsigned XML,
4. validira protiv XSD sheme,
5. sprema verziju (`payload.json` + `unsigned.xml`).

Ako validacija padne, draft se ne kreira — poruka u adminu.

### Korak 4 — Potpis

1. Otvori `VATReturn` verziju → provjeri **Draft Integrity** panel: status mora biti **SYNC**.
2. Preuzmi **unsigned XML** samo kad je SYNC (link je onemogućen uz OUT OF SYNC).
3. Ako je OUT OF SYNC: **Uskladi XML iz payload.json** (resync) ili **Generiraj novi draft** (nova verzija).
4. Potpiši certifikatom (Finin RDC / ePorezna alat) **izvan ERP-a**.

> v1: ERP ne potpisuje XML. Integracija s ePoreznom planirana u proširenjima.

### Korak 5 — Predaja na ePoreznu

Predaj potpisani Obrazac PDV i Obrazac PDV-S putem ePorezne (van ERP-a). Spremi **portal UUID** potvrde predaje za svaki obrazac (ekran ePorezne / PDF potvrde).

> **Važno:** UUID iz XML metapodataka (`Metapodaci/Identifikator`) **nije** isti kao portal UUID — ERP bilježi samo portal UUID u `external_identifier`.

### Korak 6 — Označi predano

Na stranici **PDV razdoblja** → **Označi predano** (PDV) i **Označi PDV-S predano**.

Forma traži:
- datum i vrijeme predaje (s ePorezne),
- **ePorezna identifikator** (portal UUID),
- potvrdu da je predana ista verzija obrasca iz ERP-a.

ERP **ne parsira** potpisani XML u ovom koraku — bilježi evidenciju u `SubmissionEvent` (`source=manual`, `destination=eporezna`, `payload_hash` iz drafta).

### Korak 7 — Priloži potvrdu

Nakon označavanja predaje, priložite potvrdu s ePorezne:

- **Admin → Evidencija predaje** → **Priloži potvrdu**, ili
- link u inline povijesti predaje na `VATReturn` / `PDV-S`.

| Tip priloga | Validacija |
|-------------|------------|
| XML | OIB, razdoblje, XSD, digitalni potpis; hash se uspoređuje s `payload_hash` |
| PDF / ZIP / screenshot | Spremi se; provjera ekstenzije/MIME |

Prilog se postavlja **jednom** — nije dopušteno prepisivanje.

### Korak 8 — Alati (opcionalno)

Sekcija **Alati** na stranici razdoblja:

- **Upload potpisani XML** — arhivira potpisani XML i uspoređuje s draftom (ePorezna logika parsiranja/hash/diff). Koristi se za reviziju, ne kao primarni put evidencije.
- **Usklađivanje ERP vs. predano** — usporedba trenutnog ERP izračuna s arhiviranim predanim obrascem. Korisno nakon naknadnih knjiženja u razdoblju.

## Arhivski import (prošla razdoblja)

Za uvezivanje već predanih XML-ova (Fine Star 01–04/2026):

```bash
docker compose exec django python manage.py import_pdv_xml \
  --tenant finestar \
  --dir /path/to/pdv_obrazac/
```

Kreira `VATReturn` sa `status=imported` bez usporedbe s draftom. Ne kreira automatski `SubmissionEvent` — portal UUID se bilježi ručno kroz **Označi predano**.

## Regresija nakon mjesečnog obračuna

Nakon zatvaranja razdoblja i predaje:

```bash
docker compose exec django python manage.py verify_pdv_period \
  --tenant finestar --year 2026 --month 4 \
  --xml /path/to/submitted.xml
```

Izlaz: usporedba implementiranih boxova (201, 202, 203, 303, 400). Detalji: [`pdv-stabilization-runbook.md`](pdv-stabilization-runbook.md).

## Česta pitanja

**Zašto se upload odbija iako sam samo potpisao draft?**  
Potpis ne smije mijenjati poslovne podatke u tijelu obrasca. Ako se razlikuju, provjeri je li draft generiran nakon zadnjeg knjiženja.

**Mogu li obrisati predani obrazac?**  
Ne. `submitted` i `imported` verzije su immutable. Ispravak = nova verzija.

**Što ako nemam izlazne račune u razdoblju?**  
Box 660 (`nema prometa`) postavlja se automatski kad su svi izlazni boxovi nula.

**Koji boxovi su trenutno aktivni u ERP-u?**  
201, 202, 203 (izlaz), 303 (ulaz). Ostali su u registryju za buduća proširenja — vidi [`pdv-mapping.md`](pdv-mapping.md).
