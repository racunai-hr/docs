# PDV-S — R0 matrica poreznih scenarija

```text
Status: R0 PARTIAL
Datum: 2026-11-27
Podaci: isključivo sintetički. Brojevi, nazivi i PDV ID-ovi ne pripadaju stvarnim osobama.
Pravni zaključci i izvori: R0-REGULATORY-ANALYSIS.md
```

## Zajedničke pretpostavke

Ako scenarij ne kaže drukčije: obveznik **T** ima važeći hrvatski PDV ID, mjesečni je obveznik, kupuje kao porezni obveznik; dobavljač je porezni obveznik s važećim PDV ID-om navedene države; iznosi su u EUR; 2026. godina.

| Sintetički dobavljač | Država / PDV ID u obrascu |
|---|---|
| S-DE | `DE` / `000000001` |
| S-IT | `IT` / `00000000002` |
| S-NL | `NL` / `000000003B01` |
| S-XI | `XI` / `000000004` |
| S-GB | `GB` / `000000005` |

**Klasifikacija** (ciljne oznake za testove): `IC_GOODS` (stjecanje dobara), `EU_SERVICE_RC` (usluga po Z čl. 75. st. 1. t. 6.), `OUT` (izvan PDV-S), `REVIEW` (nema pravila — blokira finalizaciju).

**PDV-S:** DA / NE / BLOK (vidljiva blokirajuća stavka; bez tihog uključivanja ili isključivanja) / — (nije primjenjivo).

**Status:** **V** = VERIFIED, **D** = DERIVED, **O** = OPEN. Scenarij sa statusom O ne smije dobiti test koji učvršćuje pretpostavljeni iznos; testira se samo da završava kao `REVIEW` / BLOK.

Oznake izvora kao u regulatornoj analizi: Z (Zakon), P (Pravilnik), UPUTE (ePorezna PDV-S).

## S — Obveza i rok

| ID | Ulazne činjenice | Razdoblje | Klasifikacija | PDV-S | I1 / I2 | Ispravak | Izvor | Status |
|---|---|---|---|---|---|---|---|---|
| S01 | T (registriran) stekao robu 1.000,00 od S-DE | 2026-07 | IC_GOODS | DA | 1.000,00 / 0,00 | novi cjeloviti obrazac | P182/1 | V |
| S02 | T s PDV ID-om izvan redovnog sustava prima uslugu 200,00 od S-NL | 2026-07 | EU_SERVICE_RC | DA | 0,00 / 200,00 | isto | P182/1; M2585 | V |
| S03 | Mali obveznik ispod praga stjecanja (ukupno 3.000 EUR u godini, prošla ispod praga, bez odustajanja) kupi robu 500,00 od S-DE | 2026-07 | OUT (ispod praga) | NE | — | — | Z5; P182/6/11 | D (učinak davanja PDV ID-a: O, Q-09) |
| S04 | Isti obveznik: kumulativ 9.800, nova nabava 600,00 prelazi 10.000 | 2026-07 | IC_GOODS od te nabave | DA | 600,00 / 0,00 | isto | P27/1 | V |
| S05 | Mjesec bez stjecanja i usluga; nema prethodne predaje | 2026-07 | — | NE (nema obveze) | — | prazan prvi obrazac odbiti | P182/5; UPUTE 005 | V |
| S06 | T nema PDV ID koji vrijedi za razdoblje, a primio je uslugu S-NL | 2026-07 | EU_SERVICE_RC | BLOK | — | nakon dodjele ID-a | Z77/4; UPUTE 011 | V |
| S07 | Rok za razdoblje 2026-01 | 2026-01 | — | — | — | propisano 28.2. (sub); rok 2.3.2026. | Z86/2; ZUP81/2 | D |
| S08 | Rok za razdoblje 2025-12 | 2025-12 | — | — | — | 20.1.2026. (staro pravilo) | NN151 čl.10 | V |
| S09 | Rok za razdoblje 2025-11 | 2025-11 | — | — | — | propisano 20.12.2025. (sub); rok 22.12.2025. | NN73 čl.86/2; ZUP81/2 | D |
| S10 | T počinje EU transakcije usred tromjesečja | svaki mjesec tromjesečja | — | DA za mjesece s transakcijama | — | PDV mjesečno za sva tri mjeseca | P173/4–5; P182/4–5 | V |

## G — Dobra

| ID | Ulazne činjenice | Razdoblje | Klasifikacija | PDV-S | I1 / I2 | Ispravak | Izvor | Status |
|---|---|---|---|---|---|---|---|---|
| G01 | Roba iz DE, isporuka i račun 10.7., 5.000,00 | 2026-07 | IC_GOODS | DA | 5.000,00 / 0 | cjeloviti | Z9, Z31/2 | V |
| G02 | Stroj (dugotrajna imovina) iz IT, 40.000,00, **bez** VIN-a ili posebne oznake u opisu | 2026-07 | IC_GOODS | DA | 40.000,00 / 0 | cjeloviti | Z9 | D |
| G03 | Rabljeno vozilo od trgovca iz DE, oslobođena isporuka unutar EU, 12.000,00; trgovac T posebno iskaže PPMV 800,00 koji prenosi na stjecatelja | 2026-07 | IC_GOODS | DA | 12.000,00 / 0 | cjeloviti | P28/4, P41/6 | V |
| G04 | Rabljeno vozilo od preprodavatelja iz DE, **postupak marže**, račun bez iskazanog PDV-a, 9.000,00 | 2026-07 | OUT (marža) | NE | — | — | Z5/1/c; P28/2 | V |
| G05 | Rabljeno vozilo od fizičke osobe iz IT | 2026-07 | OUT | NE | — | — | P28/3 | V |
| G06 | Novo vozilo (2.000 km) od obveznika iz DE s PDV ID-om, 30.000,00 | 2026-07 | IC_GOODS (NPS) | DA | 30.000,00 / 0 | cjeloviti; NPS prijava zasebno | P184/6 | V |
| G07 | Novo vozilo od prodavatelja **bez** PDV ID-a | 2026-07 | REVIEW | BLOK | — | — | P184/6; UPUTE 002 | O (Q-10) |
| G08 | EU društvo (S-NL) fakturira robu otpremljenu iz CH | 2026-07 | OUT (uvoz) | NE | — | — | Z9/1, Z11 | D |
| G09 | Društvo sa sjedištem u CH s PDV ID-om `DE000000006`, roba otpremljena iz DE | 2026-07 | IC_GOODS | DA | iznos / 0, ID `DE` | cjeloviti | Z4/1/2/a; P182/1 | D |
| G10 | Njemački dobavljač isporučuje robu koja se već nalazi u HR (bez otpreme iz druge države) | 2026-07 | OUT (II.11–13) | NE | — | — | Z75/2; P175/11–13 | D |
| G11 | Trostrani posao, T zadnji primatelj | 2026-07 | OUT (II.11–13, VI.5) | NE | — | — | Z10; P175; P178/5 | D |
| G12 | Stjecanje oslobođeno po Z čl. 42. | 2026-07 | REVIEW | BLOK | — | — | Z42; P182/6/11 | O (Q-04) |
| G13 | Roba 1.000,00 + prijevoz 100,00 + pakiranje 20,00 na računu isporučitelja | 2026-07 | IC_GOODS | DA | 1.120,00 / 0 | cjeloviti | Z33/2, Z34/1 | V |
| G14 | Trošarinska roba 2.000,00 + trošarina 300,00 koju plaća stjecatelj | 2026-07 | IC_GOODS | DA | 2.300,00 / 0 | cjeloviti | Z34/2; P41/5 | V |
| G15 | T premješta vlastitu opremu iz SI u HR radi poslovanja | 2026-07 | IC_GOODS (premještanje) | BLOK | — | — | Z9/4 | O (Q-05) |
| G16 | Domaća naknada/pristojba knjižena na konto dugotrajne imovine u pripremi (npr. posebni porez ili naknada za vozilo) | 2026-07 | OUT | NE | — | — | Z9/1 | V |
| G17 | Uvoz iz treće zemlje (JCD) | 2026-07 | OUT (II.15) | NE | — | — | Z11 | V |
| G18 | Roba iz Sjeverne Irske, S-XI, 700,00 | 2026-07 | IC_GOODS | DA | 700,00 / 0, `XI` | cjeloviti | EC-NI; XSD | V |
| G19 | Roba iz GB, S-GB, 2020-11, 700,00 (povijesno, u kunama) | 2020-11 | IC_GOODS | DA (`GB`) | iznos u HRK / 0 | ispravak u HRK | BREXIT; NN156 čl.3 | V |
| G20 | Roba iz GB, 2021-02 | 2021-02 | OUT (uvoz) | NE | — | — | BREXIT | V |
| G21 | Otprema iz GB počela 30.12.2020., stigla 4.1.2021. | prema Z31/2 | IC_GOODS | DA | iznos / 0 | — | BREXIT | V |

## U — Usluge

| ID | Ulazne činjenice | Razdoblje | Klasifikacija | PDV-S | I1 / I2 | Ispravak | Izvor | Status |
|---|---|---|---|---|---|---|---|---|
| U01 | Savjetovanje S-DE 1.500,00, obavljeno 20.7. | 2026-07 | EU_SERVICE_RC | DA | 0 / 1.500,00 | cjeloviti | Z17/1, Z75/1/6 | V |
| U02 | Mjesečna softverska pretplata S-NL 99,00 | 2026-07 | EU_SERVICE_RC | DA | 0 / 99,00 | cjeloviti | Z17/1 | D |
| U03 | Provizija rezervacijske platforme S-NL 550,00 | 2026-07 | EU_SERVICE_RC | DA | 0 / 550,00 | cjeloviti | Z17/1 | D |
| U04 | Prijevoz robe B2B, prijevoznik S-IT 400,00, zaseban račun trećeg prijevoznika | 2026-07 | EU_SERVICE_RC | DA | 0 / 400,00 | cjeloviti | Z17/1 | D |
| U05 | Usluga na nekretnini u HR, izvođač iz DE | 2026-07 | OUT (posebno pravilo; II.11–13) | NE | — | — | Z19; P182/6/12 | D |
| U06 | Hotelski smještaj u DE, račun s njemačkim PDV-om | — | OUT (mjesto izvan HR) | NE | — | — | Z19 | D |
| U07 | Ulaznica za sajam u IT / restoran / kratkoročni najam vozila u IT | — | OUT (posebno pravilo) | NE | — | — | Z19–26 | D |
| U08 | EU usluga oslobođena u RH (osiguranje) 300,00 | 2026-07 | OUT (oslobođeno) | NE | — | — | P182/3; Z40 | V |
| U09 | EU usluga po stopi 5 % ili 13 % | 2026-07 | EU_SERVICE_RC | DA | 0 / iznos | cjeloviti | P175/8–9 | V |
| U10 | Usluga od EU obveznika bez PDV ID-a | 2026-07 | REVIEW | BLOK | — | — | Z86/1; P182/1 | O (Q-03) |
| U11 | Usluga od društva sa sjedištem u trećoj zemlji koje koristi EU PDV ID, bez EU stalne poslovne jedinice | 2026-07 | OUT (II.11–13) | NE | — | — | Z86/1; P175/11–13 | D |
| U12 | Usluga od S-XI | 2026-07 | OUT (treća zemlja) | NE | — | — | EC-NI | V |
| U13 | Račun: roba 800,00 + samostalno godišnje održavanje 200,00 | po svakoj činjenici | IC_GOODS + EU_SERVICE_RC | DA | 800,00 / 200,00 | cjeloviti | Z31, Z30 | D |
| U14 | EU dobavljač pogrešno obračuna svoj PDV na uslugu općeg pravila | 2026-07 | REVIEW | BLOK | — | — | Z75/1/6 | O |

## T — Porezni datum

| ID | Ulazne činjenice | Razdoblje | Klasifikacija | PDV-S | Napomena / ispravak | Izvor | Status |
|---|---|---|---|---|---|---|---|
| T01 | Dobra stečena 15.7., račun 15.7. | 2026-07 | IC_GOODS | DA | | Z31/2; M1782 | V |
| T02 | Dobra 15.7., račun 15.8. | 2026-08 | IC_GOODS | DA | | isto | V |
| T03 | Dobra 15.7., račun 25.8. | 2026-08 (15.8.) | IC_GOODS | DA | rok izdavanja nastupio 15.8. | Z78/3; M1782 | V |
| T04 | Dobra 15.7., račun 10.9., kolovoz već predan | 2026-08 | IC_GOODS | DA | novi cjeloviti obrazac za kolovoz; **ne** u rujan | M1782; P182/7 | V |
| T05 | Račun 15.7., dobra stečena 5.8. (nije predujam) | 2026-07 | IC_GOODS | DA | iznimka po PU | M1782 | V |
| T06 | Plaćen predujam za dobra 10.7., isporuka u rujnu | — u srpnju | — | NE u srpnju | | P38/1 | V |
| T07 | Usluga obavljena 15.7., račun 15.8. | 2026-07 | EU_SERVICE_RC | DA | | Z30/1; M1782 | V |
| T08 | Usluga obavljena 15.7., račun nije izdan | 2026-07 | EU_SERVICE_RC | DA (potreban dokaz i osnovica) | bez dokumenta: dokumentirana korekcija (D01) | Z30/6; M1782 | V |
| T09 | Račun 15.7., usluga obavljena 15.8. | 2026-08 | EU_SERVICE_RC | DA | | M1782 | V |
| T10 | Predujam za uslugu plaćen 10.7., usluga u rujnu | 2026-09 | EU_SERVICE_RC | DA u rujnu | | P38/3 | V |
| T11 | Usluga srpanj–rujan, jedan račun za tromjesečje | — | REVIEW | BLOK | | Z30/2; P36/4 | O (Q-06) |
| T12 | Kontinuirana usluga > 1 godine bez računa i plaćanja | 12. mjesec svake godine | EU_SERVICE_RC | DA | | Z30/4 | V |
| T13 | Licenca na 2 godine plaćena unaprijed, račun primljen u kolovozu, počinje u srpnju | 2026-08 | EU_SERVICE_RC | DA | | M1782 | D |
| T14 | Usluga u srpnju, unesena u knjige u listopadu | 2026-07 | EU_SERVICE_RC | DA | ispravak srpnja; unos ne mijenja razdoblje | P36/2; P182/7 | V |

## N — Odobrenja, povrati, storno

| ID | Ulazne činjenice | Razdoblje | Klasifikacija | PDV-S | I1 / I2 | Ispravak | Izvor | Status |
|---|---|---|---|---|---|---|---|---|
| N01 | S-DE: račun 1.000,00 i odobrenje 200,00, oboje srpanj | 2026-07 | IC_GOODS | DA | 800,00 / 0 | — | Z33/7; P43.a | D |
| N02 | Račun 1.000,00 i puno storno u istom mjesecu; nema druge nabave od S-DE | 2026-07 | netto 0 | redak se izostavlja | — | ako je već predano: cjeloviti bez retka | UPUTE 009 | D |
| N03 | Popust 100,00 u kolovozu na srpanjsku nabavu; u kolovozu nova nabava 900,00 | ? | REVIEW | BLOK | — | razdoblje i netiranje nepotvrđeni | Z63/3 (samo pretporez) | O (Q-01) |
| N04 | Samo odobrenje 300,00 u kolovozu, bez drugih nabava | ? | REVIEW | BLOK | — | negativan iznos nepotvrđen | XSD; UPUTE 009 | O (Q-01, Q-02) |
| N05 | S-DE: dobra netto 0,00; usluge 150,00 | 2026-07 | — | DA | 0,00 / 150,00 | — | UPUTE 009 | D |
| N06 | S-DE +500,00, S-IT −500,00 (isti mjesec, S-IT odobrenje iz prethodnog) | ? | REVIEW za S-IT | BLOK | — | ukupna nula nije „nema obveze” | UPUTE 009, 012 | O |
| N07 | Predan iznos 1.100,00 umjesto 1.000,00 (pogreška unosa) | izvorno | IC_GOODS | DA | 1.000,00 / 0 | novi cjeloviti obrazac | P182/7; UPUTE 008 | V |
| N08 | Predana nabava koja se nikad nije dogodila (račun poništen) | izvorno | OUT | redak se uklanja | — | cjeloviti; prazan zamjenski ako nema drugih | P182/7; UPUTE 008 | D |
| N09 | Povrat robe nakon predaje | ? | REVIEW | BLOK | — | — | — | O (Q-01) |

## F — Obrazac, predaja, ispravci, kontrole

| ID | Ulazne činjenice | Očekivano | Izvor | Status |
|---|---|---|---|---|
| F01 | Prva predaja s dva dobavljača | 2 retka, zbrojevi jednaki zbroju redaka | UPUTE; XSD | V |
| F02 | Druga predaja istog razdoblja | upozorenje 008; prethodna nevažeća; povijest ostaje | UPUTE 008 | V |
| F03 | Prazna prva predaja | blokirano | UPUTE 005 | V |
| F04 | Prazna zamjenska predaja | dopuštena uz dokaz prethodne predaje i izričitu potvrdu | UPUTE 008 | V |
| F05 | Dva izvora istog PDV ID-a | jedan zbrojni redak | UPUTE 012 | V |
| F06 | Pogrešan PDV ID dobavljača u predanom obrascu | novi cjeloviti s ispravnim ID-om | P182/7 | D |
| F07 | Promjena konta ili mjesta troška nakon predaje | nema razlike u službenim recima → bez ispravka; razlika porijekla prikazana | P182/7 | D |
| F08 | Ispravak više od tri godine nakon roka | upozorenje/blokada uz ručnu odluku | OPZ76 | D (Q-08) |
| F09 | Ispravak razdoblja 2022-10 | iznosi u kunama | NN156 čl.3 | V |
| F10 | Nema knjiženog PDV obrasca | informacija/upozorenje 004, ne blokira | UPUTE 004 | V |
| F11 | PDV-S dobra > II.5+II.6+II.7 zadnje zaprimljenog PDV-a | upozorenje 013, ne blokira | UPUTE 013 | V |
| F12 | PDV-S usluge < II.8–II.10 | portal prolazi; interno neobrazložena razlika | UPUTE 014; U1488 | V |
| F13 | Redak 0,00 / 0,00 | blokirano | UPUTE 009 | V |
| F14 | Više od 5000 dobavljača | blokirano (granica sheme); postupak nepoznat | XSD | V (tehnički) |
| F15 | Zadnji PDV obrazac je zaprimljen i u obradi, stariji je knjižen | portalna kontrola nad zadnje zaprimljenim; interna nad zadnjim predanim po ADR D09 | UPUTE 013 | V / D |

## X — Valute

| ID | Ulazne činjenice | Očekivano | Izvor | Status |
|---|---|---|---|---|
| X01 | Usluga 1.000,00 USD obavljena 15.7., račun 3.8. | HNB srednji tečaj **15.7.** | Z36/2; R0-03 | D |
| X02 | Isto, obveznik primjenjuje ESB tečaj | ESB tečaj na dan nastanka obveze | Z36/3 | V |
| X03 | Obveza nastaje na dan bez objavljenog tečaja (npr. 15.8.2026., subota i blagdan) | REVIEW dok pravilo nije potvrđeno | Z36/2 | O (Q-07) |
| X04 | Devizno odobrenje u kasnijem mjesecu | REVIEW | — | O (Q-01, Q-07) |
| X05 | Račun u stranoj valuti s informativnim EUR iznosom dobavljača | porezni tečaj po Z36; informativni iznos ne preuzima se bez potvrde | Z36/2–3 | D |
| X06 | Tri dokumenta istog dobavljača s ostatkom zaokruživanja | deterministički; zbroj doprinosa = redak | — | O (Q-07) |
| X07 | Povijesni uvoz XML-a za razdoblje 2022-05 | iznosi ostaju u kunama, jasno označeni | NN156 čl.2 | V |

## V — Identifikacija i VIES

| ID | Ulazne činjenice | Očekivano | Izvor | Status |
|---|---|---|---|---|
| V01 | Formalno neispravan PDV ID | blokirano | UPUTE 006 | V (pravila po državama: Q-11) |
| V02 | VIES `valid` danas za transakciju od prije tri mjeseca | informacija; ne tvrditi povijesnu valjanost | VIES-WSDL | V |
| V03 | VIES `invalid` | dokumentirana provjera i odluka računovođe; materijalna nevaljanost blokira (D06) | YEU; ADR D06 | V |
| V04 | `MS_UNAVAILABLE` / `TIMEOUT` | neblokirajuće upozorenje „nepoznato” | VIES-WSDL; ADR D06 | V |
| V05 | Dobavljač s oznakom države `GR` | u obrascu `EL` | XSD | V |
| V06 | PDV ID s prefiksom `HR` među recima | blokirano | XSD | V |

## Minimalni skup za buduće automatizirane testove

Svi scenariji sa statusom **V** i **D** trebaju fixture (`accounting/tests/fixtures/tax/pdv_s/` prema `FORM_IMPLEMENTATION_CONVENTION.md`). Scenariji **O** dobivaju samo negativni test: stavka je vidljiva, blokira finalizaciju i nije tiho uključena ni isključena. Stanje postojećeg koda prema ključnim scenarijima: [audit, §4](R0-EXISTING-IMPLEMENTATION-AUDIT.md).
