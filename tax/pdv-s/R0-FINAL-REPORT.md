# PDV-S — R0 završni izvještaj

```text
R0 status: PARTIAL
Datum: 2026-11-27
ADR-0034: Proposed (nepromijenjen status)
```

Dokumenti: [regulatorna analiza](R0-REGULATORY-ANALYSIS.md) · [matrica scenarija](R0-SCENARIO-MATRIX.md) · [audit implementacije](R0-EXISTING-IMPLEMENTATION-AUDIT.md) · [arhitekturna revizija](R0-ARCHITECTURE-REVIEW.md) · [ADR-0034](../../architecture/ADR-0034-pdv-s-workflow.md)

## 1. Zašto PARTIAL

| Kriterij PASS | Stanje |
|---|---|
| Sve regulatorne cjeline obrađene | da (R0-01 … R0-09) |
| Zaključci potkrijepljeni službenim izvorima | da; neizvedeni zaključci označeni DERIVED/OPEN; Direktiva 2006/112 i VIES FAQ nisu dohvaćeni |
| Scenariji dovoljno precizni za testove | **djelomično** — scenariji OPEN nemaju očekivani iznos |
| Implementacija pregledana | da, statički; testovi nisu izvršeni |
| Arhitekturni konflikti identificirani | da (K-1 … K-10) |
| Neriješena materijalna pravna pitanja | **postoje** (Q-01, Q-02, Q-07) |

## 2. Najvažniji regulatorni zaključci

1. PDV-S je mjesečni; od razdoblja 1/2026 rok je zadnji dan idućeg mjeseca, ranije 20. (za 12/2025 vrijedi 20.1.2026.).
2. Obveznik je svatko s PDV ID-om (u sustavu ili ne) koji ima stjecanja ili primljene EU usluge; bez transakcija nema obveze, a prvi prazan obrazac ne može se poslati.
3. Usluge ulaze samo ako primatelj duguje PDV po općem pravilu (čl. 75. st. 1. t. 6.); oslobođene usluge, posebna pravila mjesta, marža, uvoz i treće zemlje ne ulaze.
4. Razdoblje određuje porezni datum: dobra — datum računa, najkasnije 15. idućeg mjeseca; usluge — dan obavljanja; predujam ne stvara obvezu.
5. Ispravak je cjeloviti zamjenski obrazac; prazan zamjenski briše prethodni.
6. Tečaj je srednji HNB (iznimno ESB) na dan nastanka obveze; razdoblja prije 2023. iskazuju se i ispravljaju u kunama.
7. Službeni XSD prihvaća negativne, nulte, prazne i duple retke te pogrešne zbrojeve — poslovna pravila mora provoditi aplikacija. Prihvaćanje negativnih iznosa na portalu nije potvrđeno.
8. VIES nema upit na datum; današnji rezultat nije dokaz povijesne valjanosti.
9. Portalna kontrola prema PDV-u je upozorenje (PDV-S ≤ II.5–7 i ≤ II.8–10 zadnje zaprimljenog PDV-a).

## 3. Najvažniji nedostaci postojeće implementacije

| Nalaz | Sažetak |
|---|---|
| A-01 | EU nabave evidentirane kao trošak izostaju (legacy) ili se roba bez VIN-a vodi kao usluga / izostaje (engine) |
| A-02 | Registar PDV polja ne odgovara službenom obrascu (modul PDV) |
| A-03 | Domaće knjiženje na 0373 (uključujući službene dokumente) postaje „EU stjecanje” ili ruši PDV-S |
| A-04 | Odobrenja i storna se ne oduzimaju; odobrenje dobavljača nema model |
| A-05 | Nema poreznog datuma |
| A-06, A-07 | Nema posebnih režima; identifikacija dobavljača krhka |
| A-08, A-09 | Nema snimke/verzije; predaja bez provjera |

**Operativno upozorenje:** dok A-01 do A-05 nisu otklonjeni, XML iz postojećeg PDV-S toka ne treba predati bez ručne usporedbe sa svim EU računima i odobrenjima za razdoblje. Rok za razdoblje 10/2026 je **30.11.2026.** Nalazi su izvedeni iz koda; stvarni podaci i predani obrasci nisu provjeravani.

## 4. Arhitekturni konflikti

K-1 porezne činjenice (ADR-0019) · K-2 PDV polja (ADR-0007) · K-3 korekcije i jedinstvena istina · K-4 jedan prilog po eventu · K-5 redoslijed uvezenih predaja · K-6 kune · K-7 dvije klasifikacijske putanje · K-8 kasni račun u zaključanom razdoblju · K-9 višemjesečna razdoblja i brisanje · K-10 hash. Detalji i preporuke: [arhitekturna revizija](R0-ARCHITECTURE-REVIEW.md) §4. Dodatno: dva ADR dokumenta s brojem 0033.

Nijedna od odluka D01–D10 ne traži ponovno otvaranje.

## 5. Otvorena pitanja za odluku vlasnika

| ID | Odluka | Preporuka |
|---|---|---|
| O-1 | Odobriti zaseban zadatak za registar PDV polja i read-only provjeru utjecaja na predane prijave? | da, prioritetno |
| O-2 | Odobriti amendment ADR-0019 (porezni datum, vrsta isporuke, režimi, snimka dobavljača, odobrenja) kao preduvjet PDVS-03? | da |
| O-3 | Korekcije (D01): porezna činjenica Tax domene s privremenim učinkom samo na PDV-S i vidljivom razlikom prema PDV-u? | da (K-3) |
| O-4 | Prilozi (D02): vlastiti PDV-S model uz nepromijenjen `SubmissionService`? | da (K-4) |
| O-5 | Uvoz starije predaje kad postoji novija (D10): blokirati uz pregled ili otvoriti amendment ADR-0009? | blokirati u prvoj verziji |
| O-6 | Povijesna razdoblja u kunama: samo uvoz i prikaz, ili i izrada ispravaka? | prvo uvoz i prikaz |
| O-7 | Renumeracija jednog ADR-0033 | renumerirati MCP Gateway ili devizni ADR prije spajanja grana |
| O-8 | Tko potvrđuje Q-pitanja (porezni savjetnik / upit Poreznoj upravi)? | pisani upit za Q-01, Q-02, Q-07 |

## 6. Pravna pitanja za potvrdu

Popis Q-01 … Q-11: [regulatorna analiza §11](R0-REGULATORY-ANALYSIS.md#11-otvorena-pravna-pitanja). Materijalna: **Q-01** (razdoblje odobrenja), **Q-02** (negativni iznosi na portalu), **Q-07** (tečaj bez tečajne liste i za odobrenja, zaokruživanje).

## 7. Rizici

| Rizik | Razina |
|---|---|
| Netočan PDV-S predan iz postojećeg toka | visoka |
| Netočna polja PDV prijave za pogođene vrste transakcija | visoka, neprovjereno |
| Početak PDVS gateova prije poreznih preduvjeta stvara paralelnu logiku | visoka |
| Pogrešna pretpostavka o odobrenjima učvršćena u testovima | srednja |
| Izvori se promijene (redakcijski tekstovi, XSD, upute) | srednja — hash i datum zapisani u registru izvora |
| OPZ pregledan samo u tekstu do NN 114/22 | niska |

## 8. Preporuka za sljedeći gate

1. **Ne prihvaćati ADR-0034 i ne počinjati implementaciju.**
2. R0.1: pisana potvrda Q-01, Q-02, Q-07.
3. Odluke O-1, O-2, O-3.
4. Nacrt amendmenta ADR-0019 i Architecture Review (zajedno s ADR-0034).
5. Nakon toga ponovna R0 ocjena i eventualni GO za PDVS-01.

## 9. Što nije učinjeno

Aplikacijski kod, migracije, baze i produkcija nisu mijenjani. Django testovi nisu pokretani, produkcijski podaci nisu čitani, ništa nije predano na ePoreznu, ADR nije prihvaćen i `develop` nije spajan u `main`. Primjeri su sintetički.
