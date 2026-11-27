# ADR-0033: računovodstvena i porezna provjera

Za: računovođu i vlasnika projekta. Datum: 2026-11-27.
Ovo je priprema za odluku, **nije** porezno ni računovodstveno mišljenje.

## 1. Pročitani izvori

Navodi su prepisani iz navedenih stranica 2026-11-27. Prije odluke provjerite ih u službenom tekstu.

### Zakon o PDV-u

Pročitano na [porezi.teb.hr](https://porezi.teb.hr/d/1089/zakon-o-porezu-na-dodanu-vrijednost-primjena-od-1-1-2026/) (neslužbeni pročišćeni tekst, popis NN do 48/26). Službeni pročišćeni tekst nije dohvaćen.

- čl. 36. st. 2.: „Ako su elementi za utvrđivanje porezne osnovice, osim pri uvozu dobara, utvrđeni u stranoj valuti, za preračun u eure koristi se srednji tečaj Hrvatske narodne banke na dan nastanka obveze obračuna PDV-a.”
- čl. 36. st. 3.: „Iznimno od stavka 2. ovoga članka, porezni obveznik može primijeniti tečaj koji na dan nastanka obveze obračuna PDV-a objavi Europska središnja banka.” Uz to: „Konverzija drugih valuta koje nisu euro vršit će se primjenom tečaja svake valute za euro.”
- čl. 31. st. 2.: obveza obračuna kod stjecanja unutar EU nastaje „u trenutku izdavanja računa ili istekom roka iz članka 78. stavka 3. ovoga Zakona ako račun do tada nije izdan”.

[Pravilnik o PDV-u](https://porezna-uprava.gov.hr/Propisi/Propis/pravilnik-o-porezu-na-dodanu-vrijednost-redakcijski-procisceni-tekst/1652) (u primjeni od 4. 4. 2026.), čl. 157. st. 4. upućuje na tečaj iz čl. 36. st. 2. Zakona.

### HSFI

[NN 86/2015](https://narodne-novine.nn.hr/clanci/sluzbeni/2015_08_86_1709.html), izvorni tekst.

- 11.12: „Tečaj na datum bilance – je spot tečaj na datum bilance, odnosno srednji tečaj Hrvatske narodne banke.”
- 11.25: transakcije u stranoj valuti evidentiraju se po spot tečaju na datum transakcije.
- 15.19: definicija monetarnih stavki.
- 16.37: tečajne razlike iz podmirenja monetarnih stavki ili njihova iskazivanja po drugom tečaju priznaju se u računu dobiti i gubitka razdoblja u kojem nastaju.

**Nije pročitano:** izmjene NN 105/20 i ispravak NN 9/21. NN 150/22 ne mijenja odredbe o valutama. U HSFI 13 nije nađena odredba o tečaju.

**Nije istraženo:** porezni tretman nerealiziranih razlika po Zakonu o porezu na dobit.

### Tečajevi

- [ECB](https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/html/index.en.html) objavljuje oko 16:00 CET na TARGET dane, „for information purposes only”, uz napomenu da se uporaba u transakcijske svrhe izričito ne preporučuje.
- Za 2. 11. 2026., DKK za 1 EUR: ECB **7,4737**, HNB srednji **7,4735**. Dobavljačev odnos 7,50 nije nijedan od njih.

## 2. Otvorena pitanja

| # | Pitanje | Što sustav sada radi | Odlučuje |
|---|---------|----------------------|----------|
| Q1 | Knjigovodstveni izvor tečaja. ADR predlaže ECB; HSFI 11.12 i ZPDV čl. 36. st. 2. navode HNB srednji. | Nudi oba (za knjiženje ECB prvi, za porez HNB prvi). Ništa se ne potvrđuje samo. | računovođa |
| Q2 | Relevantni datum za knjiženje i za porez (račun, isporuka, primitak, nastanak obveze PDV-a). | Predlaže prema datumu računa; API prima `target_date`, sučelje ne. | računovođa |
| Q3 | Smije li se za vikend/praznik uzeti prethodni objavljeni tečaj, za koju svrhu i koliko unatrag. | Isključeno (0 dana). Raniji tečaj vidi se s datumom, ali ga nije moguće potvrditi kao objavljeni. | računovođa |
| Q4 | Konta realiziranih razlika i naknade. | Prijedlog: rashod **4750**, prihod **7721**, naknada **4652**. Knjiženje zaključano do odobrenja. | računovođa |
| Q5 | Nerealizirane razlike: metoda, učestalost, konta, porezni tretman. | Predložena metoda: usklađenje na datum bilance i storno sljedeći dan. Rashod **4783**; za nerealizirani prihod u kontnom planu nije nađeno očito konto. Zaključano. | računovođa |
| Q6 | Predujmovi u stranoj valuti (plaćanje prije računa). | Odbija se; postojeći predujmovi rade samo u EUR. | računovođa |
| Q7 | Porezna osnovica kad je obveza ugovorena u EUR, a račun glavni iznos iskazuje u stranoj valuti. | Porezni preračun ide iz valute dokumenta (HNB, ECB ili dokumentirani unos). | računovođa |
| Q8 | Kad se porezni i knjigovodstveni preračun razlikuju, odakle osnovica za polja 207 i 210. | RC PDV računa se po poreznom tečaju; retci čija osnovica dolazi iz knjiženog iznosa idu na pregled. | računovođa |
| Q9 | Tečajne razlike na samom deviznom računu (4754 / 7720). | Nije obuhvaćeno. | računovođa |

Dopuštanje ranijeg tečaja je postavka (`FX_PRIOR_RATE_DAYS_ACCOUNTING`, `_TAX`, `_VALUATION`, `_PAYMENT`).

## 3. Shema knjiženja razlika (prijedlog, nije odobren)

Primjer: obveza 1.000 USD knjižena na 900,00 EUR, plaćeno 920,00 EUR i naknada 7,50 EUR.

| Konto | Duguje | Potražuje |
|-------|-------:|----------:|
| 2201 analitika dobavljača | 900,00 | |
| 4750 negativne tečajne razlike | 20,00 | |
| 4652 bankovne usluge | 7,50 | |
| banka | | 927,50 |

Kod djelomičnog plaćanja zatvara se razmjerni dio preostale EUR vrijednosti; zadnje plaćanje zatvara ostatak do centa.

## 4. Račun 547 (pregled samo za čitanje)

Račun F261166 (K.H. AUTO ApS, 2. 11. 2026., 103.500,00 DKK) pri pregledu 2026-11-27 bio je nacrt bez potvrđenog tečaja, temeljnice, saldakonta i PDV retka.

Dokumentacija dobavljača iskazuje i **103.500 DKK** i **13.800 EUR**, bez PDV-a, za rabljeno osobno vozilo.

**Valuta ugovorene obveze ne može se utvrditi iz računa.** Potreban je ugovor, narudžba ili potvrda dobavljača.

| Ako je obveza | Knjigovodstvena vrijednost | Posljedica |
|---------------|----------------------------|------------|
| 13.800 EUR | 13.800,00 EUR (ugovorni iznos) | nema tečajnih razlika; DKK je informativan |
| 103.500 DKK uz potvrđen 7,50 | 13.800,00 EUR | devizna obveza; razlika pri plaćanju |
| 103.500 DKK uz ECB 7,4737 | 13.848,56 EUR | devizna obveza |
| 103.500 DKK uz HNB 7,4735 | 13.848,93 EUR | devizna obveza |

Prije knjiženja odlučiti:

1. valutu obveze i dokaz;
2. knjigovodstveni tečaj (ako je obveza DKK);
3. PDV tretman: stjecanje unutar EU ili poseban postupak oporezivanja marže za rabljeno vozilo, te porezni tečaj;
4. posting profil i konto. Zadani zahtjev je konto **650**; za vozilo na zalihi postoji i profil `vehicle_inventory_cost`, koji traži vozilo u registru;
5. je li što plaćeno unaprijed (Q6).

Podaci računa 547 nisu mijenjani.
