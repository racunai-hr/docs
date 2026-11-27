# Technical Debt Register — racunAI ERP

Status: Living document
Date: 2026-11-27
Related: [PDVS-PRODUCTION-01](../tax/pdv-s/PRODUCTION-READINESS-PLAN.md), [ADR-0034](ADR-0034-pdv-s-workflow.md), [ROADMAP](ROADMAP.md).

## Odluka vlasnika i granice

RacunAI.hr je trenutačno u testnoj fazi. Vlasnik planira buduću migraciju Django backenda na FastAPI + SQLAlchemy, koristeći provjerena iskustva iz sigurnosne arhitekture koja se razvija na drugom projektu. Potpuna rekonstrukcija sigurnosnog sloja postojećeg Django backenda, rotacije pristupnih podataka kao zaseban program i organizacijska/compliance priprema za širu uporabu odgađaju se. To nije potvrda sigurnosti sadašnjeg sustava, prihvat svakog pojedinog rizika niti dokaz sanacije.

SEC-AUDIT-01 je izvršen; sanacija nije izvršena prema posljednjem dostupnom izvještaju. G01 se odvaja od kritičnog puta razvoja poreznog modula, ali ne označava PASS. Poznati rizici ostaju otvoreni. Minimalno ograničavanje pristupa stvarnim privatnim dokumentima ostaje preporučena zasebna mjera; ovaj zapis ne autorizira promjenu produkcije. Ne širiti uporabu na druge korisnike niti proglasiti produkcijsku spremnost dok relevantni sigurnosni kriteriji nisu dokazano ispunjeni. Privremeni razvoj i testiranje nisu certifikacija produkcije.

Ovaj registar ne sadrži putanje ranjivosti, pristupne podatke, stvarne identifikatore, poslovne dokumente ili detaljne dokaze. Oni ostaju u postojećem privatnom SEC-AUDIT-01 izvještaju izvan Gita.

## Registar

| ID | Područje / zadatak | Status | Ciljna faza / uvjet |
|---|---|---|---|
| TD-SEC-01 | Zaštita privatnih dokumenata, autorizirani download i sigurno rukovanje uploadom | OPEN — odgođena potpuna sanacija; minimalna zaštita preporučena odmah | Zasebna mjera za sadašnji sustav; obvezna verifikacija prije produkcijske spremnosti |
| TD-SEC-02 | Tenant izolacija, RBAC i autorizacija svih admin/API/datotečnih radnji | OPEN | Nova sigurnosna arhitektura; novi PDV-S ulazi moraju biti izolirani već sada |
| TD-SEC-03 | Sigurnost integracijskih endpointa i mrežne granice | OPEN | Zaseban infrastrukturni/security gate; verificirati prije šire uporabe |
| TD-SEC-04 | Upravljanje tajnama, pristupnim podacima i pravima datoteka | DEFERRED | Budući sigurnosni program; stvarno izložene tajne ne koristiti za novi sustav |
| TD-SEC-05 | Higijena repozitorija, sintetički fixturei, pregled javno objavljenih podataka i vidljivosti | OPEN / DEFERRED | Zaseban zadatak; bez prepisivanja Git povijesti ili promjene vidljivosti bez GO |
| TD-SEC-06 | Sigurnost XLSX/CSV izvoza i preostalih legacy admin radnji | OPEN | Novi exporti i novi ulazi moraju biti sigurni od početka; legacy sanacija zasebno |
| TD-SEC-07 | Organizacijska priprema, zaštita osobnih podataka, evidencija i operativni sigurnosni procesi | DEFERRED | Prije uključivanja drugih korisnika; postojeće zakonske obveze nisu poništene testnim statusom |
| TD-ARCH-01 | Migracija Django → FastAPI + SQLAlchemy 2.x | PLANNED — budući zaseban program | Novi ADR i zaseban GO; ne blokira sadašnji porezni razvoj |
| TD-ARCH-02 | Prenosivi domenski servisi i razdvajanje ORM adaptera od poslovnih pravila | ACTIVE DESIGN CONSTRAINT | TAX-CORE-01/02 i PDV-S: bez drugog poreznog enginea, postupno izdvajanje čistih pravila |
| TD-OPS-01 | Backup baze i privatnih datoteka, automatizacija i probni povrat | OPEN | Prije produkcijskog deploymenta; postojeći OPS-01 |
| TD-OPS-02 | Stage dostupnost i paritet verzija | OPEN | G10; postojeći OPS-02 |
| TD-OPS-03 | Lokalni CI, porezni regresijski testovi, GitHub Actions | OPEN | Preporučeno prije promjena porezne jezgre, obvezno G10; postojeći OPS-03 |
| TD-OPS-04 | Ponovljiv deploy/rollback i identitet imagea | OPEN | G11; postojeći OPS-04 |
| TD-OPS-05 | WIP-01: postojeće necommitane produkcijske izmjene | OPEN | Zaseban pregled prije prvog deploya; ne uklanjati ili automatski commitati |
| TD-GOV-01 | Konflikt numeracije ADR-0033 i usklađivanje dokumentacijskih grana | OPEN | G05; postojeći DOC-01; bez gubitka povijesti |

## Buduća migracija backenda

Potvrđen je smjer FastAPI + SQLAlchemy. Sljedeće su predložene komponente za zasebnu arhitekturnu razradu, ne prihvaćeni tehnički ugovor: Alembic, PostgreSQL, centralni identity provider (kandidat Keycloak), ponovno korištenje provjerenih security/access obrazaca drugog projekta, Next.js i OpenAPI kompatibilnost. Prije izbora provjeriti stvarno prihvaćene odluke i stanje tog projekta; ne prenositi tajne, konfiguraciju ili kod nekritički.

Migracijski program mora obuhvatiti inventar Django modela i API-ja, tenant izolaciju, autorizaciju, pohranu dokumenata, audit, migracije, kompatibilnost računovodstvenih podataka, predanih poreznih snapshotova i integracija, testove pariteta i kontrolirani cutover/rollback. PostgreSQL podaci i postojeći predani obrasci ne smiju se mijenjati nekontrolirano. Ne uvoditi drugi aktivni porezni engine; sadašnje izračune oblikovati kao čiste Python domenske funkcije gdje je razumno, s persistence adapterima, bez masovnog refaktoriranja samo radi buduće migracije.

## Pravilo zatvaranja

Svaka stavka dobiva status DONE tek uz dokaz isporuke i testova, ili zasebnu dokumentiranu odluku o zamjeni odgovarajućim novim rješenjem. Odgoda nije zatvaranje. Ne objavljivati detalje otvorenih sigurnosnih nalaza. Ažurirati registar na kontrolnim točkama i pri promjeni opsega uporabe sustava.
