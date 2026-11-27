# PDV-S — R0 regulatorna analiza (R0-01 do R0-09)

```text
Status: R0 PARTIAL (otvorena materijalna pravna pitanja — vidi §11)
Datum analize i dohvata izvora: 2026-11-27
ADR: ../../architecture/ADR-0034-pdv-s-workflow.md (Proposed)
Ovo nije porezno mišljenje. Zaključci označeni DERIVED i OPEN traže potvrdu.
```

Povezani dokumenti: [matrica scenarija](R0-SCENARIO-MATRIX.md) · [audit implementacije](R0-EXISTING-IMPLEMENTATION-AUDIT.md) · [arhitekturna revizija](R0-ARCHITECTURE-REVIEW.md) · [završni izvještaj](R0-FINAL-REPORT.md)

## 0. Kako čitati

| Oznaka | Značenje |
|---|---|
| **VERIFIED** | Izričito propisano ili prikazano u dohvaćenom službenom izvoru |
| **DERIVED** | Slijedi iz službenih izvora tumačenjem; primjenjivo uz potvrdu računovođe |
| **OPEN** | Službeni izvor ne daje jednoznačan odgovor ili nije dohvaćen; ne implementirati kao pravilo |

„Primjena od” navodi datum samo kad je utvrđen. „Važeće” znači tekst na snazi na datum analize, bez utvrđivanja povijesnog datuma uvođenja.

## 1. Registar izvora

| Oznaka | Izvor | Verzija / datum | Napomena |
|---|---|---|---|
| **Z** | [Zakon o PDV-u — redakcijski pročišćeni tekst (Porezna uprava)](https://porezna-uprava.gov.hr/Propisi/Propis/zakon-o-porezu-na-dodanu-vrijednost-redakcijski-procisceni-tekst/1651) | NN 73/13 … 151/25, 32/26, 48/26; na snazi od 9.5.2026. | Redakcijski tekst; pravno mjerodavne su objave u Narodnim novinama |
| **P** | [Pravilnik o PDV-u — redakcijski pročišćeni tekst (Porezna uprava)](https://porezna-uprava.gov.hr/Propisi/Propis/pravilnik-o-porezu-na-dodanu-vrijednost-redakcijski-procisceni-tekst/1652) | NN 79/13 … 11/26, 35/26; od 4.4.2026. | isto |
| **NN73** | [Zakon o PDV-u NN 73/13](https://narodne-novine.nn.hr/clanci/sluzbeni/2013_06_73_1451.html) | 18.6.2013. | izvorni tekst (rokovi 2013.–2025.) |
| **NN113** | [Izmjene ZPDV NN 113/22](https://narodne-novine.nn.hr/clanci/sluzbeni/2022_09_113_1660.html) | euro; prag 10.000 EUR; čl. 36. st. 2. | primjena od 1.1.2023. |
| **NN151** | [Izmjene ZPDV NN 151/25](https://narodne-novine.nn.hr/clanci/sluzbeni/2025_12_151_2262.html) | 15.12.2025. | na snazi 1.1.2026. |
| **NN11** | [Izmjene Pravilnika NN 11/26](https://narodne-novine.nn.hr/clanci/sluzbeni/full/2026_01_11_90.html) | 30.1.2026. | mijenja čl. 182. st. 4. |
| **NN156** | [Pravilnik o prijavama poreza vezano za uvođenje eura NN 156/22](https://narodne-novine.nn.hr/clanci/sluzbeni/2022_12_156_2537.html) | na snazi 1.1.2023. | valuta prijava i ispravaka |
| **UPUTE** | [ePorezna — Obrazac PDV-S, korisničke upute](https://e-porezna.porezna-uprava.hr/Upute/G2B/Korisni%C4%8Dke%20upute%20-%20Obrazac%20PDV-S.pdf) | PDF od 28.1.2026.; SHA-256 `e40906ce7353dd9284e493c90c9529a0ae4aa76ee0ec5ccc1379f37cc1603dde` | identičan lokalnom arhivu |
| **XSD** | [ePorezna_Schemas.zip](https://e-porezna.porezna-uprava.hr/Upute/G2B/ePorezna_Schemas.zip) | dohvat 27.11.2026.; SHA-256 `5aa49b88272ab198f3d350196ac3c429ca8effdeebd22d8e0fef1d293d7251b8` | PDV-S datoteke datirane 2013.–2021. |
| **OPIS** | „Opis elemenata XML sheme PDV” (u XSD paketu, mapa `PDV/`) | v11.0 | semantika polja Obrasca PDV |
| **OBR** | [Obrazac PDV-S (tiskani)](https://porezna-uprava.gov.hr/UserDocsImages/Arhiva/HR_obrasci/Documents/POREZ%20NA%20DODANU%20VRIJEDNOST/PDV-S.pdf) | PDF 12/2022. (euro) | polja (1)–(17) |
| **NEWS26** | [Obavijest PU o izmjeni Pravilnika](https://porezna-uprava.gov.hr/hr/izmjena-i-dopuna-pravilnika-o-porezu-na-dodanu-vrijednost-8156/8156) | 11.12.2025. | sadržaj PDV-S i ZP ne mijenja se od 1.1.2026. |
| **U1488** | [Uputa za popunjavanje prijave PDV-a](https://porezna-uprava.gov.hr/Misljenja/Detaljno/1488) | 17.6.2013. | starija uputa; vidi R0-09 |
| **M1782** | [Mišljenje: nastanak porezne obveze](https://porezna-uprava.gov.hr/Misljenja/Detaljno/1782) | 1.4.2014. | tablice datuma; starije mišljenje |
| **M2585** | [Mišljenje: mali porezni obveznici](https://porezna-uprava.gov.hr/Misljenja/Detaljno/2585) | 30.12.2024. | PDV-S za male obveznike |
| **QA** | [Pitanja i odgovori — pristupanje EU](https://porezna-uprava.gov.hr/hr/pitanja-i-odgovori-na-temu-izmjena-u-pdv-u-vezanih-za-pristupanje-republike-hrvatske-u-europsku-uniju/4323) | 2013. | razdoblje stjecanja |
| **BREXIT** | [Porezna uprava — Brexit](https://www.porezna-uprava.hr/bi/Stranice/Brexit.aspx) | — | prijelazno razdoblje, Sjeverna Irska |
| **EC-NI** | [Europska komisija, Notice to stakeholders — VAT for goods, REV3](https://commission.europa.eu/document/download/f43c412b-f00b-4c92-b0a5-f1f8263b8642_en?filename=vat-goods_en.pdf) | 10.12.2020. | XI: dobra da, usluge ne |
| **VIES-WSDL** | [VIES checkVat — definicija servisa](https://ec.europa.eu/taxation_customs/vies/checkVatService.wsdl) | dohvat 27.11.2026. | ulazi, izlazi, greške |
| **YEU** | [Your Europe — Check a VAT number](https://europa.eu/youreurope/business/taxation/vat/check-vat-number-vies/index_en.htm) | — | značenje valid/invalid |
| **OPZ** | [Opći porezni zakon — redakcijski tekst](https://porezna.gov.hr/gdpr/Opci_porezni_zakon.pdf) | **samo do NN 114/22** | čl. 4. i 76.; kasnije izmjene nisu provjerene u ovom tekstu |
| **ZUP** | [Zakon o općem upravnom postupku NN 47/09](https://narodne-novine.nn.hr/clanci/sluzbeni/2009_04_47_1065.html) | — | čl. 80.–81. rokovi |
| **BLAG** | [Zakon o blagdanima NN 110/19](https://narodne-novine.nn.hr/clanci/sluzbeni/2019_11_110_2212.html) | od 2020. | neradni dani |

**Nedohvaćeni izvori:** tekst Direktive 2006/112/EZ na EUR-Lexu (poslužitelj je automatiziranom klijentu vraćao prazan odgovor) i VIES FAQ Europske komisije (sadržaj se učitava skriptom). Nijedan zaključak ne oslanja se na njih. `zakon.hr` nije korišten.

## R0-01 — Obveza i razdoblje predaje

| # | Zaključak | Izvor | Primjena | Posljedica za implementaciju | Status |
|---|---|---|---|---|---|
| 1.1 | PDV-S podnosi pravna ili fizička osoba upisana u registar obveznika PDV-a **ili registrirana za potrebe PDV-a** s PDV ID-om, za stjecanja dobara i primljene usluge iz drugih država članica od osoba registriranih za PDV u tim državama | P čl. 182. st. 1.–2.; Z čl. 86. st. 1. | važeće | Obveza ovisi o PDV ID-u i transakcijama, ne o statusu „u sustavu PDV-a” | VERIFIED |
| 1.2 | Mali porezni obveznik s transakcijama s EU podnosi PDV-S (i PDV za ta razdoblja) | M2585 (t. 1.6.); Z čl. 85. st. 3.; P čl. 173. st. 2. | 2024.+ | Status `vat_id` mora biti podržan | VERIFIED |
| 1.3 | Stjecanje dobara osobe iz Z čl. 5. st. 1. t. b) (samo oslobođene isporuke bez odbitka; pravna osoba koja nije obveznik) ispod praga **10.000 EUR** (tekuća i prethodna godina) nije predmet oporezivanja, osim uz pisano odustajanje; ne vrijedi za nova prijevozna sredstva i trošarinsku robu. PDV-S obuhvaća dobra „koja podliježu oporezivanju PDV-om u RH” | Z čl. 5. st. 1.–4.; P čl. 27.; P čl. 182. st. 6. t. 11. | prag u EUR od 1.1.2023. (NN113); prije 77.000 kn (NN73) | Potreban atribut praga/odustajanja po obvezniku i godini; ispod praga dobra se ne uključuju, usluge se uključuju | DERIVED (OPEN: učinak davanja PDV ID-a dobavljaču ispod praga) |
| 1.4 | Prag za malog poreznog obveznika (domaći promet 60.000 EUR) nije prag stjecanja i ne isključuje PDV-S za usluge | Z čl. 90. st. 1.; čl. 77. st. 4. | važeće | Ne vezati obvezu uz prag malog obveznika | DERIVED |
| 1.5 | Obveznik koji prima EU usluge po čl. 75. st. 1. t. 6. mora zatražiti PDV ID i kad je inače oslobođen | Z čl. 77. st. 4.–5. | važeće | Nedostajući PDV ID obveznika = blokada i uputa, ne prazan obrazac | VERIFIED |
| 1.6 | Portal ne prima obrazac ako obveznik za razdoblje nema važeći PDV ID (poruka 011); pristup traži RPO status i PDV obvezu 07/09/14 aktivnu barem jedan dan u zadnjih 5 godina | UPUTE str. 5. i 14.; Z čl. 77. st. 9. (suspenzija) | 2026. | Validacija PDV statusa **po razdoblju**; aplikacija ne vidi RPO — upozoriti | VERIFIED |
| 1.7 | Razdoblje je kalendarski mjesec. Obveznici s transakcijama unutar EU ne mogu biti tromjesečni; onaj koji ih počne obavljati tijekom tromjesečja podnosi mjesečne prijave PDV-a za svaki mjesec tog tromjesečja, a onaj koji prestane ostaje mjesečni do kraja godine | P čl. 182. st. 4.; Z čl. 84. st. 3. i čl. 86. st. 2.; P čl. 173. st. 4.–5. | važeće | PDV-S je uvijek mjesečni; višemjesečni `VATPeriod` ne smije biti osnova | VERIFIED |
| 1.8 | Rok od razdoblja **siječanj 2026.**: zadnji dan mjeseca koji slijedi | Z čl. 86. st. 2. (NN151 čl. 5.); P čl. 182. st. 4. (NN11) | Zakon 1.1.2026.; Pravilnik 31.1.2026. | Rok je funkcija razdoblja | VERIFIED |
| 1.9 | Rok za razdoblja **srpanj 2013. – prosinac 2025.**: 20. dan mjeseca koji slijedi. Za zadnje razdoblje 2025. vrijede stari propisi i rokovi (20.1.2026.) | NN73 čl. 86. st. 2.; M2585; NN151 čl. 10. | do razdoblja 12/2025 | Povijesni rok se ne smije računati po novom pravilu | VERIFIED |
| 1.10 | Ako zadnji dan roka padne na nedjelju, blagdan ili drugi dan kad javnopravno tijelo ne radi, rok istječe prvog sljedećeg radnog dana | ZUP čl. 81. st. 2. uz OPZ čl. 4. (supsidijarna primjena); BLAG | važeće | Prikazati propisani i pomaknuti datum; potvrditi za elektroničku predaju i na aktualnom tekstu OPZ-a | DERIVED |
| 1.11 | Ako u razdoblju nema stjecanja ni primljenih usluga, obveze podnošenja nema; prva prazna prijava ne može se poslati (005) | P čl. 182. st. 5.; UPUTE str. 13. | važeće | Status „nema obveze” samo uz potpunu evidenciju | VERIFIED |

### Rokovi za tekuće cikluse (izvedeno iz 1.8–1.10)

| Razdoblje | Propisani datum | Rok uz pomak |
|---|---|---|
| 2025-11 | 20.12.2025. (sub) | 22.12.2025. |
| 2025-12 | 20.1.2026. | 20.1.2026. |
| 2026-01 | 28.2.2026. (sub) | 2.3.2026. |
| 2026-02 | 31.3.2026. | 31.3.2026. |
| 2026-03 | 30.4.2026. | 30.4.2026. |
| 2026-04 | 31.5.2026. (ned) | 1.6.2026. |
| 2026-05 | 30.6.2026. | 30.6.2026. |
| 2026-06 | 31.7.2026. | 31.7.2026. |
| 2026-07 | 31.8.2026. | 31.8.2026. |
| 2026-08 | 30.9.2026. | 30.9.2026. |
| 2026-09 | 31.10.2026. (sub) | 2.11.2026. |
| 2026-10 | 30.11.2026. | 30.11.2026. |
| 2026-11 | 31.12.2026. | 31.12.2026. |
| 2026-12 | 31.1.2027. (ned) | 1.2.2027. |

Popis blagdana mijenjao se; zakon iz BLAG vrijedi od 2020. Za starija razdoblja pomak računati po tada važećem popisu (nije analizirano).

## R0-02 — Dobra i usluge

### Temeljni test

- **Dobra (stupac 11 / `I1`)**: stjecanje prava raspolaganja pokretnim dobrom otpremljenim ili prevezenim **iz druge države članice** (Z čl. 9. st. 1.), oporezivo u RH (Z čl. 4. st. 1. t. 2., čl. 27.), od prodavatelja koji je porezni obveznik u drugoj državi članici i nije mali obveznik. Država sjedišta dobavljača sama po sebi ne određuje ništa.
- **Usluge (stupac 12 / `I2`)**: usluge **za koje je primatelj obvezan platiti PDV po Z čl. 75. st. 1. t. 6.**, dakle po općem pravilu mjesta obavljanja za porezne obveznike (Z čl. 17. st. 1.), obavljene od poreznog obveznika sa sjedištem u drugoj državi članici (Z čl. 86. st. 1.; P čl. 182. st. 1. i st. 6. t. 12.).

| # | Kategorija | PDV-S | Izvor | Status |
|---|---|---|---|---|
| 2.1 | Stjecanje robe iz EU po općem postupku | I1 | Z čl. 9., 75. st. 1. t. 3., 86.; P čl. 182. st. 6. t. 11. | VERIFIED |
| 2.2 | Dugotrajna imovina stečena iz EU | I1 (nema posebnog isključenja) | isto | DERIVED |
| 2.3 | Rabljeno prijevozno sredstvo od obveznika koji je isporuku oslobodio kao isporuku unutar EU | I1 | P čl. 28. st. 4. | VERIFIED |
| 2.4 | Rabljena dobra / prijevozna sredstva uz **posebni postupak marže** ili javnu dražbu | **ne** | Z čl. 5. st. 1. t. c)–d); P čl. 28. st. 2. | VERIFIED |
| 2.5 | Rabljeno prijevozno sredstvo od fizičke osobe, malog obveznika ili neobveznika bez prava na odbitak | **ne** | P čl. 28. st. 3. | VERIFIED |
| 2.6 | **Novo prijevozno sredstvo** (≤ 6 mjeseci ili ≤ 6.000 km za kopnena vozila) — obveznik upisan u registar ili s PDV ID-om iskazuje ga u PDV i PDV-S; posebna prijava NPS Carinskoj upravi je odvojena obveza | I1 | Z čl. 4. st. 3., 87. st. 2.; P čl. 29., 184. st. 6. | VERIFIED (OPEN: prodavatelj bez PDV ID-a) |
| 2.7 | PPMV kod trgovca vozilima / leasinga koji prenosi porez na stjecatelja | ne ulazi u osnovicu | P čl. 41. st. 6. | VERIFIED |
| 2.8 | Trošarine pri stjecanju trošarinske robe | ulaze u I1 | Z čl. 34. st. 2.; P čl. 41. st. 5. | VERIFIED |
| 2.9 | Provizije, prijevoz, pakiranje, osiguranje koje **isporučitelj** zaračuna uz robu | dio vrijednosti robe (I1) | Z čl. 33. st. 2. uz čl. 34. st. 1. | VERIFIED |
| 2.10 | Mješoviti račun (roba + samostalna usluga) | razdvojiti po poreznoj činjenici; sporedni trošak slijedi robu | 2.9 | DERIVED |
| 2.11 | B2B usluge općeg pravila: softver kao usluga, licence, pretplate, savjetovanje, oglašavanje | I2 | Z čl. 17. st. 1., 75. st. 1. t. 6.; M1782 (licence) | VERIFIED / DERIVED po vrsti |
| 2.12 | Provizije posrednika (npr. rezervacijske platforme) primljene kao B2B usluga | I2, osim posredovanja vezanog uz posebno pravilo mjesta (npr. nekretnine) | isto | DERIVED |
| 2.13 | Prijevozne i logističke usluge prijevoza dobara B2B od EU prijevoznika | I2 | Z čl. 17. st. 1.; posebna pravila prijevoza čl. 20. provjeriti po vrsti | DERIVED |
| 2.14 | Usluge pod posebnim pravilima mjesta (nekretnine, prijevoz putnika, ulaznice/priredbe, restoran, kratkoročni najam prijevoznih sredstava) | **ne** — ako su oporezive u RH idu u PDV II.11–13, ako izvan RH nisu u hrvatskoj prijavi | Z čl. 19.–26.; P čl. 175. t. 11.–13.; P čl. 182. st. 6. t. 12. | DERIVED |
| 2.15 | Usluge **oslobođene PDV-a u RH** (npr. financijske, osiguranje) | **ne** | P čl. 182. st. 3.; Z čl. 39.–40. | VERIFIED |
| 2.16 | EU usluge po sniženim stopama (5 %, 13 %) | I2 | P čl. 175. t. 8.–9. | VERIFIED |
| 2.17 | Domaće nabave, uključujući tuzemni prijenos porezne obveze (PDV II.4) | **ne** | Z čl. 75. st. 3.; P čl. 175. t. 4. | VERIFIED |
| 2.18 | Isporuka u RH od EU obveznika bez sjedišta u RH (roba se ne otprema iz druge države članice) | **ne** (II.11–13) | Z čl. 75. st. 2.; P čl. 175. t. 11.–13. | DERIVED |
| 2.19 | Uvoz iz treće zemlje, i kad račun izdaje EU društvo, a roba dolazi iz treće zemlje | **ne** | Z čl. 9. st. 1., 11. | DERIVED |
| 2.20 | Usluge od obveznika bez sjedišta u EU (i ako ima EU PDV ID) | **ne** (II.11–13) | Z čl. 86. st. 1.; P čl. 175. t. 11.–13. | DERIVED |
| 2.21 | Dobra otpremljena iz države članice od prodavatelja sa sjedištem u trećoj zemlji koji koristi PDV ID te države | I1 s tim PDV ID-om | Z čl. 4. st. 1. t. 2. a); P čl. 182. st. 1. | DERIVED |
| 2.22 | Trostrani posao — RH je zadnji primatelj | **ne** u I1 (II.11–13 i VI.5) | Z čl. 10.; P čl. 175. t. 11.–13., čl. 178. t. 5. | DERIVED |
| 2.23 | Premještanje vlastitih dobara iz druge države članice | jest stjecanje (PDV II.5–7); način iskaza isporučitelja u PDV-S | Z čl. 9. st. 4. | OPEN |
| 2.24 | Oslobođena stjecanja (Z čl. 42.) | formulacija „podliježu oporezivanju” ne rješava izričito | Z čl. 42.; P čl. 182. st. 6. t. 11. | OPEN |
| 2.25 | Usluge EU obveznika **bez PDV ID-a** | Z čl. 86. st. 1. traži „sve podatke”, P čl. 182. st. 1. govori o registriranima, a obrazac traži PDV ID | Z, P, UPUTE | OPEN |
| 2.26 | **Sjeverna Irska (XI)** od 1.1.2021.: dobra prema pravilima EU; **usluge kao iz treće zemlje** | dobra → I1 s `XI`; usluge **ne** | EC-NI (uvod i t. 1.); BREXIT; XSD `tKodDrzave` | VERIFIED |
| 2.27 | **Velika Britanija (GB)**: do 31.12.2020. pravila EU; stjecanje čija je otprema počela prije isteka prijelaznog razdoblja, a roba stigla kasnije, jest stjecanje u EU; od 1.1.2021. treća zemlja | do 2020. I1/I2 s `GB`; poslije ne | BREXIT; EC-NI; XSD sadrži `GB` | VERIFIED (2.27 usluge DERIVED) |

**Zaključak za implementaciju:** klasifikacija treba najmanje: vrstu isporuke (dobro/usluga/sporedni trošak), pravilo mjesta (opće/posebno), oslobođenje, postupak marže, državu otpreme, državu i PDV ID dobavljača za transakciju, status praga obveznika. Država partnera ili oblik PDV ID-a nisu dovoljni.

## R0-03 — Porezni datum

| # | Slučaj | Trenutak nastanka obveze obračuna | Izvor | Status |
|---|---|---|---|---|
| 3.1 | Stjecanje dobara | izdavanje računa, a ako do tada nije izdan, istek roka za izdavanje: **15. dan mjeseca** nakon mjeseca oporezivog događaja | Z čl. 31. st. 2., čl. 78. st. 3.; M1782 t. 2. | VERIFIED |
| 3.2 | Račun za stjecanje izdan prije isporuke | tablica PU: obveza na dan računa; takav slučaj treba biti iznimka (prije isporuke izdaje se račun za predujam) | M1782 t. 2. (tablica) | VERIFIED (starije mišljenje) |
| 3.3 | Primljene usluge EU | kad je usluga obavljena, neovisno o računu | Z čl. 30. st. 1. i 6.; P čl. 36. st. 2. i 4., čl. 40. st. 4.; M1782 t. 3.; M2585 | VERIFIED |
| 3.4 | Kontinuirane usluge | Z čl. 30. st. 2.: po isteku razdoblja na koje se računi/plaćanja odnose; P čl. 36. st. 4.: po proteku svakog razdoblja oporezivanja, bez obzira na fakturiranje | Z, P | OPEN (uskladiti dvije formulacije za periodično fakturiranje dulje od mjeseca) |
| 3.5 | Kontinuirane usluge uz prijenos obveze dulje od godine, bez računa i plaćanja | istek svake kalendarske godine | Z čl. 30. st. 4. | VERIFIED |
| 3.6 | Licenca na godinu dana ili dulje, plaćena unaprijed | razdoblje **primitka računa** | M1782 t. 3. | DERIVED (mišljenje iz 2014. — potvrditi) |
| 3.7 | Predujam za stjecanje dobara | **ne nastaje obveza** | P čl. 38. st. 1. | VERIFIED |
| 3.8 | Predujam za usluge i dobra uz obvezu primatelja prema stranom dobavljaču | **ne nastaje obveza** | P čl. 38. st. 3. | VERIFIED |
| 3.9 | Rok plaćanja, odgoda, kreditiranje | ne utječu | P čl. 36. st. 6. | VERIFIED |
| 3.10 | Naknadno zaprimljen ili naknadno evidentiran račun | razdoblje se **ne** pomiče na datum primitka/unosa; ako je prijava predana, ispravlja se razdoblje nastanka (iznimka 3.6) | M1782 (tablica: račun 10.9. za stjecanje 15.7. → kolovoz); P čl. 182. st. 7. | VERIFIED / DERIVED |
| 3.11 | Usluga nije obavljena, a račun izdan | primatelj nema oporezivi događaj; duguje izdavatelj | M1782 t. 3. | VERIFIED (starije mišljenje) |

Starija uputa (QA, 2013.) kaže: stjecanje se prijavljuje u razdoblju izdavanja računa, a ako račun nije primljen, u mjesecu koji slijedi mjesec oporezivog događaja. To je u skladu s 3.1.

### Datumi koji se moraju voditi odvojeno

| Datum | Uloga |
|---|---|
| Datum računa | određuje razdoblje za dobra (3.1) |
| Datum isporuke / završetka usluge | oporezivi događaj; određuje razdoblje za usluge, a za dobra rok od 15. dana |
| **Porezni datum** (obveza obračuna) | **jedini** određuje PDV-S razdoblje; izvodi se iz prethodnih |
| Datum primitka računa | dokaz; ne određuje razdoblje (osim 3.6) |
| Datum knjiženja / unosa | audit; ne određuje razdoblje |
| Datum plaćanja | ne određuje razdoblje (3.7–3.9) |

## R0-04 — Negativne osnovice i odobrenja

| # | Zaključak | Izvor | Status |
|---|---|---|---|
| 4.1 | XSD tip `tDecimal15-2` za `I1`/`I2` nema `minInclusive`: shema prihvaća negativne iznose. Provjereno validacijom sintetičkog primjera na službenoj shemi | XSD | VERIFIED (samo tehnički) |
| 4.2 | Upute traže iznos „različit od 0,00” u (11) ili (12); ne kažu da mora biti pozitivan | UPUTE str. 11. i 14. (009) | VERIFIED; **prihvaća li portal negativan iznos nije provjereno** → OPEN |
| 4.3 | Osnovica kod stjecanja utvrđuje se po istim elementima kao za isporuke u tuzemstvu; popusti odobreni pri isporuci ne ulaze, a naknadni popust ili opoziv mogu umanjiti osnovicu uz obavijest isporučitelja | Z čl. 33. st. 3. i 7., čl. 34. st. 1.; P čl. 41. st. 9., 43.a | VERIFIED |
| 4.4 | Povećanje naknade povećava osnovicu | P čl. 43. | VERIFIED |
| 4.5 | **Odbitak pretporeza** ispravlja se u razdoblju u kojem je primljena isporučiteljeva obavijest o ispravku | Z čl. 63. st. 3. | VERIFIED za pretporez |
| 4.6 | Primjena istog razdoblja na **osnovicu stjecanja/usluge** (PDV II.5–10 i PDV-S) nakon odobrenja iz drugog mjeseca nije izričito propisana u dohvaćenim izvorima | — | **OPEN** |
| 4.7 | Ispravak **pogrešnih podataka** već podnesene prijave radi se novom prijavom za to razdoblje | P čl. 182. st. 7.; UPUTE 008 | VERIFIED |
| 4.8 | Izdani, a stornirani račun za neobavljenu isporuku ne stvara obvezu isporučitelju | P čl. 36. st. 9. | VERIFIED (pravilo izdavatelja) |

Treba razlikovati dva pravna događaja: **pogrešku** u predanim podacima (ispravak izvornog razdoblja, 4.7) i **naknadnu promjenu osnovice** (popust, povrat, odobrenje), za koju razdoblje treba potvrdu (4.6). Radna hipoteza koju treba potvrditi: razdoblje primitka obavijesti, po analogiji s 4.5. Sustav je ne smije primijeniti bez potvrde.

| Slučaj | Stanje |
|---|---|
| Djelomično/potpuno odobrenje u istom razdoblju | netira se unutar istog dobavljača i vrste (DERIVED) |
| Neto nula za dobavljača (I1 i I2 = 0) | redak se ne može unijeti (009) (VERIFIED) |
| Neto negativan za dobavljača | shema dopušta; portal i razdoblje OPEN |
| Ukupni zbroj nula, a reci nisu | reci ostaju; nulti zbroj ne dokazuje nepostojanje obveze (DERIVED) |

**Posljedica postojećeg pravila** (`aggregate_pdv_s_rows()` preskače `base_amount <= 0`): odobrenja i storna se ne oduzimaju, pa PDV-S može biti **veći** od stvarnog i veći od PDV II.5–10 (poruke 013/014); dobavljač s negativnim netom nestaje; razlika nije vidljiva korisniku. Pravilo se ne smije zadržati niti zamijeniti jednostavnim zbrajanjem dok 4.6 nije potvrđen — nejasni slučajevi moraju biti vidljivi i blokirajući.

## R0-05 — Ispravci i poništavanje

| # | Pravilo | Izvor | Status |
|---|---|---|---|
| 5.1 | Prva predaja razdoblja mora imati podatke | UPUTE 005 | VERIFIED |
| 5.2 | Nova prijava za isto razdoblje dopuštena je; prethodna postaje nevažeća (upozorenje 008) | UPUTE str. 13.–14. | VERIFIED |
| 5.3 | Ispravak je **cjelovit** zamjenski obrazac s ispravnim podacima, ne razlika | P čl. 182. st. 7.; UPUTE 008 | VERIFIED |
| 5.4 | „Brisanje” predanog obrasca: poslati novi za isto razdoblje bez stavki | UPUTE 008 | VERIFIED |
| 5.5 | Prazan obrazac je valjan po shemi (`Isporuka` `minOccurs=0`; zbrojevi `0.00`) | XSD | VERIFIED |
| 5.6 | Ispravak PDV ID-a dobavljača = novi cjeloviti obrazac | 5.3 | DERIVED |
| 5.7 | Naknadna računovodstvena izmjena koja ne mijenja službene retke ne zahtijeva ispravak | P čl. 182. st. 7. | DERIVED |
| 5.8 | Opće pravilo ispravka porezne prijave: netočna ili nepotpuna prijava koja može dovesti do manje ili više plaćenog poreza; rok tri godine od isteka roka, do obavijesti o nadzoru; iznimke | OPZ čl. 76. (tekst do NN 114/22) | VERIFIED za navedeni tekst; primjena na PDV-S i aktualnost OPEN |
| 5.9 | Ispravak se podnosi u valuti izvorne prijave: razdoblja prije 1.1.2023. u **kunama** | NN156 čl. 2.–3. | VERIFIED |
| 5.10 | Pravne osobe potpisuju obrazac; ePorezna izdaje potvrdu zaprimanja (PDF/XML) i zasebne statuse obrade s oznakom „konačan” | UPUTE str. 17.–21. | VERIFIED |

„Zaprimljen” nije isto što i obrađen. Sustav smije evidentirati samo ono za što ima dokaz.

## R0-06 — Devizni računi

| # | Pravilo | Izvor | Primjena | Status |
|---|---|---|---|---|
| 6.1 | Elementi osnovice (osim uvoza) u stranoj valuti preračunavaju se po **srednjem tečaju HNB-a na dan nastanka obveze obračuna** | Z čl. 36. st. 2. | EUR od 1.1.2023. | VERIFIED |
| 6.2 | Iznimno se može primijeniti tečaj koji na taj dan objavi **ESB**; ostale valute preračunavaju se preko tečaja prema euru | Z čl. 36. st. 3. | važeće | VERIFIED |
| 6.3 | Relevantni datum je porezni datum iz R0-03, **ne** datum unosa niti automatski datum računa (osobito za usluge i dobra bez računa do 15.) | 6.1 + R0-03 | | DERIVED |
| 6.4 | Dan bez objavljene tečajne liste (vikend, blagdan) — koji tečaj | nije utvrđeno iz dohvaćenih izvora | | OPEN |
| 6.5 | Tečaj za odobrenje/storno devizne isporuke (izvorni ili na dan odobrenja) | nije utvrđeno | | OPEN |
| 6.6 | Iznosi obrasca: euro s dvije decimale | OBR; XSD `fractionDigits=2` | od 2023. | VERIFIED |
| 6.7 | Metoda zaokruživanja (po stavci, dokumentu ili retku) nije propisana u PDV-S izvorima | — | | OPEN (preporuka: po dokumentu, pola naviše, kao postojeći `cents()`) |
| 6.8 | Razdoblja do 12/2022 iskazuju se u kunama; XSD opisi i dalje glase „u kunama i lipama” | NN156 čl. 2.; XSD | | VERIFIED |
| 6.9 | Portal može odstupati od ZP podataka dobavljača zbog tečaja; PU to navodi kao uobičajeno | U1488 t. 4. | | VERIFIED (starija uputa) |

Konzistentnost s arhitekturom: ADR-0033 (devizni ulazni računi; postoji na grani `main`, ne na `develop`) razdvaja knjigovodstveni i porezni preračun, što odgovara 6.1–6.2. Vidi [arhitekturnu reviziju](R0-ARCHITECTURE-REVIEW.md) §3.

## R0-07 — VIES i identifikacija

| # | Zaključak | Izvor | Status |
|---|---|---|---|
| 7.1 | Hrvatski PDV ID = `HR` + OIB | Z čl. 77. st. 6. | VERIFIED |
| 7.2 | U PDV-S se upisuju kôd države i PDV ID **bez** kôda; shema: 28 vrijednosti (26 država članica osim HR, uz `GB` i `XI`), Grčka je `EL`; `HR` i `GR` nisu dopušteni; PDV ID ≤ 12 znakova | XSD `tKodDrzave`, `tPDVID`; OBR | VERIFIED |
| 7.3 | Shema ne provjerava format po državi i prihvaća prazan `PDVID`; portal javlja 006 za formalno neispravan broj | XSD (provjereno); UPUTE str. 13. | VERIFIED |
| 7.4 | Detaljna tablica formata po državama (VIES FAQ) | nije dohvaćena | OPEN — preuzeti iz službenog izvora prije implementacije |
| 7.5 | VIES nije baza nego upit nacionalnim bazama; odgovor je valid ili invalid; invalid može značiti nepostojeći, neaktiviran za EU transakcije ili još neupisan broj | YEU | VERIFIED |
| 7.6 | Servis `checkVat` prima samo državu i broj; **nema ulaznog datuma**, odgovor nosi `requestDate`. Servisom se ne može dokazati valjanost na raniji datum | VIES-WSDL | VERIFIED (iz definicije servisa) |
| 7.7 | `checkVatApprox` uz podatke tražitelja vraća `requestIdentifier` (referencu upita) | VIES-WSDL | VERIFIED |
| 7.8 | Greške `SERVICE_UNAVAILABLE`, `MS_UNAVAILABLE`, `TIMEOUT`, `MS_MAX_CONCURRENT_REQ`, `GLOBAL_MAX_CONCURRENT_REQ`, `INVALID_INPUT` nisu negativan rezultat (osim `INVALID_INPUT`, koji znači neispravan upit) | VIES-WSDL | VERIFIED / DERIVED |
| 7.9 | Komisija ne preuzima odgovornost za točnost podataka država članica; servis se smije koristiti samo za svrhu iz Uredbe 904/2010 | VIES-WSDL (documentation) | VERIFIED |

Posljedica: spremati rezultat, vrijeme upita i referencu; nikada ne prikazati današnji `valid` kao dokaz na datum transakcije; masovno ili periodično prikupljanje ograničiti na validaciju obrasca.

## R0-08 — XML i ePorezna

### Usporedba sheme

| Datoteka (mapa `PDV-S/`) | Lokalna kopija u `api` |
|---|---|
| `ObrazacPDVS-v1-0.xsd`, `ObrazacPDVStipovi-v1-0.xsd`, `ObrazacPDVSmetapodaci-v1-0.xsd`, `MetapodaciTipovi-v2-0.xsd`, `ObrazacPDVS-primjer.xml` | bajt-identične |
| `TemeljniTipovi-v2-1.xsd` | **nije** bajt-identična verziji iz mape `PDV-S/`; odgovara verziji iz drugih mapa paketa (npr. `PDV/`). Dodaje tri tipa (`tAdresaDodatakBroju`, `tDecimal15-4`, `tNonNegativeDecimal15-2`); zajednički tipovi semantički su jednaki |

### Pravila sheme i portala

| # | Element / pravilo | XSD | Portal (UPUTE) |
|---|---|---|---|
| 8.1 | `verzijaSheme="1.0"` fiksno; redoslijed `Metapodaci`, `Zaglavlje`, `Tijelo` | da | 003 za neispravnu strukturu |
| 8.2 | `Zaglavlje`: `Razdoblje`(`DatumOd`,`DatumDo`), `Obveznik`(`Naziv` ili `Ime`+`Prezime`, `OIB`, `Adresa`), `ObracunSastavio`(`Ime`,`Prezime`,`Telefon?`,`Fax?`,`Email?`), `Ispostava`, `Zastupnik?` | da (tim redom) | podaci obveznika popunjavaju se automatski |
| 8.3 | `Isporuka`: `RedBr`, `KodDrzave`, `PDVID`, `I1`, `I2` — svi obvezni | da; 0–5000 redaka | 002, 007, 010 |
| 8.4 | `IsporukeUkupno`: `I1`, `I2` | obvezno; **zbroj se ne provjerava** | portal ga računa sam i prikazuje tablicu razlika |
| 8.5 | Decimale: ≤ 15 znamenki, ≤ 2 decimale | da | |
| 8.6 | Jedan redak po PDV ID-u | **ne provjerava** | 012 |
| 8.7 | Barem jedan od (11)/(12) ≠ 0,00 | **ne provjerava** | 009 |
| 8.8 | Prva prazna prijava | **ne provjerava** | 005 |
| 8.9 | Telefon `+` i 8–13 znamenki; ispostava 4 znamenke; naziv/ime 2–128 znakova | da | |
| 8.10 | Postojanje knjiženog PDV obrasca | — | 004, upozorenje |

XSD validacija **nije** dovoljna: poslovna pravila 8.4 i 8.6–8.8 mora provoditi aplikacija.

### Identifikatori

`Metapodaci/Identifikator` je GUID dokumenta u XML-u. Portal za dostavljeni obrazac prikazuje identifikator i datum zaprimanja (UPUTE str. 20.). ADR-0009 (projektna odluka) zabranjuje korištenje XML identifikatora kao identifikatora predaje. Odnos dvaju identifikatora službene upute ne opisuju; pravilo ADR-0009 ostaje.

### Normalizacije portala

Portal pri učitavanju XML-a sam preračunava zaštićena polja (zbrojevi, podaci obveznika). Preuzeti predani XML zato se može razlikovati od učitanog u zaglavlju, metapodacima i potpisu. Usporedba dokaza treba se temeljiti na kanonskom sadržaju: razdoblje, OIB, retci i zbrojevi, neovisno o redoslijedu redaka.

G2B predaja nije analizirana (izvan opsega ADR-0034).

## R0-09 — Usklađivanje s Obrascem PDV

### Mapiranje

| PDV-S | Obrazac PDV (osnovice) | XML element PDV v11.0 | Izvor |
|---|---|---|---|
| (13) ukupno dobra | II.5 + II.6 + II.7 | `Podatak205`, `206`, `207` | UPUTE 013; P čl. 175.; OPIS |
| (14) ukupno usluge | II.8 + II.9 + II.10 | `Podatak208`, `209`, `210` | UPUTE 014; P čl. 175.; OPIS |
| pretporez (nije kontrola PDV-S) | III.5–III.7 dobra, III.8–III.10 usluge | `Podatak305`–`310` | P čl. 176.; OPIS |

Za usporedbu: II.4 = tuzemni prijenos porezne obveze (`204`), II.11–13 = isporuke obveznika bez sjedišta u RH (`211`–`213`), II.15 = uvoz (`215`). Postojeći registar polja u `api` ne odgovara ovom mapiranju za dio polja — vidi [audit](R0-EXISTING-IMPLEMENTATION-AUDIT.md), nalaz A-02.

### Pravila kontrole

| # | Zaključak | Izvor | Status |
|---|---|---|---|
| 9.1 | Portal **upozorava** (013/014) ako je zbroj PDV-S veći od odgovarajućih osnovica; manji ili jednak je dopušten; ne blokira slanje (žute poruke ne sprječavaju slanje) | UPUTE str. 11., 14.–15. | VERIFIED za 013; za 014 isto (opis rješenja) |
| 9.2 | Usporedba za 013 koristi **zadnje zaprimljeni** PDV obrazac za isto razdoblje, **bez obzira na status** | UPUTE str. 14. | VERIFIED |
| 9.3 | Starija uputa iz 2013.: dobra ≤ II.5–7, a usluge **moraju biti jednake** II.8–10 | U1488 t. 4. | VERIFIED da tako piše; novije portalne upute (2026.) prevladavaju za portalnu kontrolu. Interno prikazati svaku razliku |
| 9.4 | Opravdane razlike mogu nastati npr. zbog usluga dobavljača bez PDV ID-a (2.25), tečaja ili namjerno različitog obuhvata (npr. 2.23, 2.24) | R0-02 | DERIVED |
| 9.5 | Obveznik s EU transakcijama mjesečni je i za PDV (1.7), pa je redovna usporedba mjesec–mjesec; višemjesečne PDV prijave bez odvojenog potvrđenog pravila označiti neusporedivima | P čl. 173. st. 4.–5. | VERIFIED / DERIVED |
| 9.6 | Nepostojanje PDV obrasca daje upozorenje 004 i **ne blokira** PDV-S | UPUTE str. 13. | VERIFIED |
| 9.7 | Nakon ispravka bilo kojeg obrasca kontrolu ponoviti | 9.2; ADR-0034 D09 | DERIVED |
| 9.8 | Rok za PDV i PDV-S od 2026. je isti (zadnji dan mjeseca) | Z čl. 85. st. 6. i čl. 86. st. 2. | VERIFIED |

Portalna kontrola i interna rekoncilijacija zasebni su statusi: portalna po 9.2 gleda zadnje zaprimljenu verziju, interna po ADR-0034 D09 zadnje predane nepromjenjive verzije.

## 11. Otvorena pravna pitanja

| ID | Pitanje | Blokira scenarije | Materijalnost |
|---|---|---|---|
| Q-01 | U kojem razdoblju i s kojim predznakom iskazati naknadni popust, povrat ili odobrenje EU dobavljača u PDV-S i PDV II.5–10? | N03, N04, N09, X04 | **visoka** |
| Q-02 | Prihvaća li ePorezna negativne `I1`/`I2`; kako iskazati negativan neto po dobavljaču ili cijelom razdoblju? | N04 | **visoka** |
| Q-03 | Usluge EU obveznika bez PDV ID-a: iskazuju li se i kako? | U10 | srednja |
| Q-04 | Oslobođena stjecanja (Z čl. 42.) — ulaze li u PDV-S? | G12 | srednja |
| Q-05 | Premještanje vlastitih dobara — koji se isporučitelj upisuje? | G15 | niska–srednja |
| Q-06 | Kontinuirane usluge: Z čl. 30. st. 2. i P čl. 36. st. 4.; vrijedi li i dalje mišljenje za višegodišnje licence? | T11, T13 | srednja |
| Q-07 | Tečaj za dan bez tečajne liste i za odobrenja; metoda zaokruživanja | X03, X04, X06 | **visoka** za devizne račune |
| Q-08 | Pomak roka na radni dan za elektroničku predaju; provjera aktualnog OPZ-a (izmjene nakon NN 114/22) | S07, S09, F08 | niska |
| Q-09 | Mali obveznik ispod praga koji dobavljaču da PDV ID: smatra li se da je odustao od praga? | S03 | srednja |
| Q-10 | Novo prijevozno sredstvo od prodavatelja bez PDV ID-a: kako ga iskazati u PDV-S? | G07 | niska |
| Q-11 | Neobvezno za domaći PDV-S, ali potrebno: službena tablica formata PDV ID-a po državama | V01 | niska |

Pitanja Q-01, Q-02 i Q-07 materijalna su i razlog su statusa **PARTIAL**.
