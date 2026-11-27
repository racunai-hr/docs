# ADR-0033 — Ulazni računi u stranoj valuti i prikaz u EUR

```text
Status: Proposed
Date: 2026-11-27
Type: Domain
Supersedes: —
Related: ADR-0020-document-read-model.md, ADR-0019-tax-classification-engine.md, ADR-0007-pdv-module.md
```

## Status

**Proposed** — predlaže jedinstveni obračun i prikaz EUR vrijednosti ulaznih računa. Prihvaćanje i implementacija, uključujući ispravak postojećih podataka, zasebni su koraci.

## Context

Sustav vodi knjige u EUR, ali dobavljački računi mogu biti izdani u drugoj valuti. Korisnik treba vidjeti EUR vrijednost kao glavni iznos te i dalje imati pristup izvornom iznosu i dokazivom tečaju.

Primjer je račun F261166 od 02.11.2026. dobavljača K.H. AUTO ApS: prema dokumentaciji dobavljača 103.500,00 DKK uz 7,50 DKK za 1 EUR odgovara 13.800,00 EUR. Ovaj primjer ne utvrđuje automatski porezni tretman ni stanje postojećeg knjiženja.

Postojeći model već ima polja za tečaj i dio projekcije može izračunati EUR vrijednost iz spremljenog tečaja. Dohvat tečaja za DKK još nije općenito dostupan; prikaz liste stavlja izvornu valutu na prvo mjesto. Pomoćna funkcija za knjiženje pri nedostajućem tečaju može vratiti nepretvoreni brojčani iznos, što može dovesti do pogrešnog EUR knjiženja.

## Decision

### Izvorni podaci i tečaj

- Sačuvati izvornu valutu i iznose s računa; EUR vrijednost ih ne zamjenjuje. Iznos u valuti dobavljača ostaje dostupan za provjeru i plaćanje.
- Za račun evidentirati potvrđeni tečaj kao broj jedinica izvorne valute za 1 EUR, njegov izvor i datum tečaja. Kod ručnog unosa ili preuzimanja iz dobavljačke dokumentacije sačuvati i dokaz ili referencu na dokumentaciju. Zabilježiti tko je i kada potvrdio tečaj.
- Budućim računima sustav predlaže tečaj iz pouzdanog izvora za relevantni datum. Korisnik ga potvrđuje ili mijenja prije knjiženja; dobavljačev dokumentirani tečaj može se unijeti i potvrditi. Dostupnost izvora ne smije uvjetovati ručni unos.
- Koristiti decimalnu aritmetiku: izvorni iznos dijeli se potvrđenim tečajem, a EUR rezultat zaokružuje na cent, `ROUND_HALF_UP`. Tečaj mora biti pozitivan. Za EUR je vrijednost nepromijenjena; za HRK se zadržava postojeći fiksni tečaj.
- Porezni tečaj i datum utvrđuju se prema primjenjivim pravilima i mogu se razlikovati od dobavljačeva ili knjigovodstvenog preračuna. Njihova svrha i izvor moraju biti razlučivi; dobavljačev tečaj nije automatski potvrda porezne ispravnosti. Prije implementacije poreznih pravila potrebna je računovodstvena provjera.

### Centralni servis tečajeva

- Zajednički backend servis za sve module i tenante; ECB kao predloženi primarni referentni izvor, uz računovodstvenu provjeru dopuštenog izvora za svaku svrhu. Podržati dostupne valute, uključujući DKK, USD, GBP, CHF, SEK i NOK.
- Dnevni idempotentni dohvat, lokalna povijesna pohrana i ponovni pokušaji. Evidentirati valutu, smjer kotacije, datum objave i primjene, izvor, vrijeme dohvata i verziju.
- Za vikende i praznike predložiti prethodni objavljeni tečaj samo ako pravila konkretne svrhe to dopuštaju; prikazati stvarni datum. Bez budućeg tečaja ili tihog fallbacka. Za nepokrivene valute omogućiti dokumentirani alternativni izvor ili ručni unos.
- Odvojiti referentni tečaj od potvrđenog snapshot-a računa; ručna izmjena zahtijeva razlog, dokaz, ovlaštenog korisnika i audit. Read model ne poziva vanjske servise.

### Računi s više valuta

- Razlikovati izvornu valutu i iznose dokumenta, ugovorenu valutu i iznos obveze, informativni protuvrijednosni iznos, knjigovodstvenu valutu EUR, porezni preračun i stvarnu valutu plaćanja.
- Ako je obveza u EUR, informativni DKK iznos ne stvara deviznu obvezu. Ako je obveza u DKK, voditi otvoreni iznos u DKK i EUR knjigovodstvenu vrijednost. Ne zaključivati valutu obveze samo iz prisutnosti dvaju iznosa; tražiti potvrdu na temelju dokumentacije.
- Za F261166 provjeriti je li ugovorena obveza 13.800 EUR ili 103.500 DKK; odnos 7,50 nije automatski službeni knjigovodstveni ili porezni tečaj.

### Tečajne razlike i zatvaranje obveza

- Evidentirati datum, valutu, iznos i EUR vrijednost svakog plaćanja, primijenjeni tečaj i izvor; bankovne naknade odvojiti od tečajnih razlika.
- Podržati djelomična i višestruka plaćanja, alokaciju na račune, preostali saldo u valuti obveze i EUR te realizirane pozitivne/negativne tečajne razlike. Primjer: obveza 1.000 USD knjižena na 900 EUR, plaćanje 920 EUR bez naknada, razlika 20 EUR rashoda, uz primjenjiva računovodstvena pravila.
- Podržati vrednovanje otvorenih monetarnih deviznih obveza na datum izvještavanja prema primjenjivom standardu, nerealizirane razlike i kasnije zatvaranje bez dvostrukog priznavanja. Ne revalorizirati EUR obvezu zbog informativne strane valute.
- Knjiženja razlika moraju biti uravnotežena, sljediva, idempotentna i poštovati zaključana razdoblja; konkretne kontne sheme, datume i pravila vrednovanja potvrditi prije implementacije.

### Prikaz i ugovor podataka

- U listi i detalju ulaznog računa glavni je prikaz EUR bruto iznos; izvorni bruto iznos i valuta prikazuju se sekundarno. Ondje gdje se prikazuju neto iznos, PDV i stavke, njihove se valute i EUR ekvivalenti jasno označavaju. Za EUR račun nema suvišnog drugog prikaza istog iznosa.
- Read model koristi kanonski potvrđeni izračun i vraća izvorne iznose i valutu, dostupne EUR vrijednosti te tečaj s podrijetlom. Frontend ne izračunava tečaj niti zaključuje da je strani iznos u EUR. Postojeća polja izvornih iznosa zadržavaju svoju semantiku; eventualno proširenje API-ja ostaje kompatibilno s postojećim klijentima.
- Dok tečaj nije poznat ili potvrđen, EUR iznos označava se kao nedostupan, uz zahtjev za unos ili potvrdu; izvorni iznos ostaje vidljiv. Nepoznati EUR iznosi ne ulaze u EUR zbrojeve kao nula ili kao izvorni broj, a nepotpunost agregata mora biti vidljiva.
- Ovo je predložena dopuna ADR-a 0020: nakon prihvaćanja u njegovoj tablici izvora istine umjesto opće tvrdnje da je tečaj `not_recorded` navesti potvrđeni kanonski tečaj kad postoji, a `not_recorded` kad ne postoji. Read model i dalje ne dohvaća tečaj s mreže niti mijenja podatke.

### Knjiženje i postojeći računi

- Knjiženje, saldakonto i povezani EUR izvještaji koriste isti odgovarajući potvrđeni knjigovodstveni preračun, uz sljedivost do izvornog iznosa i tečaja. Ako obvezni tečaj nedostaje, operacija koja bi stvorila financijski ili porezni EUR zapis mora se zaustaviti s jasnom porukom; zabranjen je povrat nepretvorenog stranog broja kao EUR.
- Potvrđeni izračun za knjiženi račun mora ostati reproducibilan i nakon promjene izvora tečajeva. Promjena tečaja nakon knjiženja ide kroz postojeći postupak ispravka, uz očuvanje revizijskog traga, ne tihim prepisivanjem temeljnice, saldakonta ili poreznih evidencija.
- Prije ispravka računa 547 provjeriti izvorni dokument, evidentirani tečaj, temeljnicu, saldakonto i porezne zapise te stanje razdoblja. Ako je potreban ispravak knjiženja, koristiti primjenjivi postupak storna ili korekcije; ne pretpostavljati da je račun neknjižen. Podatak o 7,50 DKK za 1 EUR unijeti uz dokumentaciju dobavljača i potvrdu odgovorne osobe.
- Postojeće račune u stranoj valuti obuhvatiti pregledom nedostajućih tečajeva i mogućih pogrešnih EUR knjiženja. Automatska migracija ne smije nagađati povijesne tečajeve ni mijenjati knjižene zapise bez postupka ispravka.

## Consequences

### Prednosti

- Korisnik prvo vidi usporediv EUR iznos, uz sačuvanu izvornu valutu i dokaz preračuna.
- Isti potvrđeni podatak povezuje prikaz, knjiženje i saldakonto; nedostajući tečaj postaje vidljiv problem.

### Rizici / trade-off

- Dostupnost, datum i izvor automatskog tečaja te porezna pravila zahtijevaju zasebnu provjeru prije implementacije. Korisnička potvrda dodaje korak u obradu.
- Pregled povijesnih dokumenata može otkriti pogrešna knjiženja koja zahtijevaju pojedinačni računovodstveni ispravak, osobito u zatvorenim razdobljima.
- Razlike zbog zaokruživanja pojedinih stavki i ukupnog iznosa potrebno je uskladiti u implementaciji bez gubitka centa ili neuravnotežene temeljnice.

### Follow-up

- [ ] Potvrditi specifikaciju centralnog servisa, pravila izvora i datuma, model valute obveze i plaćanja te računovodstveni tretman realiziranih i nerealiziranih razlika.
- [ ] Provjeriti postojeće module plaćanja, saldakonta i knjiženja prije proširenja; definirati zasebne implementacijske zadatke i testove.

- [ ] Arhitekturna i računovodstvena provjera izvora tečaja, relevantnog datuma i zasebnih poreznih zahtjeva; po prihvaćanju dopuniti ADR-0020.
- [ ] Implementirati potvrdu tečaja, kanonski preračun, zaštitu knjiženja, kompatibilni read model i EUR primarni prikaz.
- [ ] Pregledati račun 547 i ostale pogođene račune te provesti potrebne korekcije kroz postojeći postupak.

## Acceptance scenariji

| Slučaj | Očekivanje |
|--------|------------|
| F261166: 103.500,00 DKK, potvrđen tečaj 7,50 | Glavni prikaz 13.800,00 EUR; sekundarno 103.500,00 DKK; izvor i tečaj dostupni |
| Račun u EUR | Iznos ostaje nepromijenjen; nema ponovljene sekundarne vrijednosti |
| Strana valuta bez tečaja | EUR vrijednost nedostupna; izvor vidljiv; knjiženje blokirano; agregat označen nepotpunim |
| Predloženi ili ručno izmijenjeni tečaj | Korisnik ga potvrđuje; vrijednost, datum, izvor i potvrda ostaju sljedivi |
| Nedostupan automatski izvor | Korisnik može unijeti i potvrditi dokumentirani tečaj |
| Porezni tečaj razlikuje se od dobavljačeva | Obje svrhe i izvori su razlučivi; porezni zapis ne koristi dobavljačev tečaj bez provjere |
| Zaokruživanje stavki | EUR iznosi su usklađeni i temeljnica je uravnotežena |
| Već knjižen račun s pogrešnim iznosom | Stanje se provjerava; ispravak je slijediv i ne prepisuje povijesnu temeljnicu |
| Vikend/praznik | Samo dopušten prethodni objavljeni tečaj uz vidljiv datum; inače potvrđeni alternativni izvor |
| Ponovljeni dnevni dohvat ili izmjena izvora | Nema duplikata ni promjene potvrđenih povijesnih preračuna |
| Dvije valute, obveza u EUR | Strani informativni iznos ne stvara deviznu obvezu |
| Dvije valute, obveza nejasna | Potvrda valute obveze prije knjiženja |
| Djelomično plaćanje devizne obveze | Ispravna alokacija, preostali saldo i realizirana razlika bez dvostrukog knjiženja |
| Otvorena devizna obveza na datum izvještavanja | Vrednovanje prema potvrđenim pravilima, sljedivost i bez dvostrukog priznavanja pri plaćanju |
| Plaćanje s bankovnom naknadom | Naknada odvojena od tečajne razlike |
