# Mail PU podršci — PTS testiranje sukladnosti

**Predmet:** PTS testiranje sukladnosti — Fine Star d.o.o. (OIB 36619131370) — neusklađenost statusa testova i tehnički problemi

**Primatelj:** Porezna uprava — podrška za PTS / eRačun / fiskalizaciju 2.0  
**Pošiljatelj:** Ante Vrcan, avrcan@finestar.hr, +385 97 671 3511  
**Datum:** 13. 06. 2026.

---

Poštovani,

javljamo se u vezi testiranja sukladnosti na Portalu za testiranje sukladnosti (PTS) za **Fine Star d.o.o.**, OIB **36619131370**.

**Svrha testiranja:** Za sebe (pristupna točka za vlastite potrebe, čl. 63. Zakona o fiskalizaciji)  
**Područje testiranja:** eRačun, MPS, Fiskalizacija

Ispod navodimo sve probleme na koje nailazimo. Na našoj strani servisi su dostupni i logovi pokazuju da u većini slučajeva test pada prije nego što Porezna uprava uspješno dovrši komunikaciju s našim sustavom, ili PTS ne ažurira status unatoč uspješnom AS4 odgovoru.

---

## 1. Slanje eRačuna — ACKNOWLEDGED na našoj strani, PTS ostaje „U tijeku“ / ne prepoznaje uspjeh

**Test pokrenut:** 13. 06. 2026. u **22:22:54**  
Generirani testni eRačun poslan u skladu s PTS uputama (GENERIRAJ → slanje u aktivnoj sesiji testa).

| Podatak | Vrijednost |
|---------|------------|
| Broj računa | `13062026-TP-4129` |
| Izdavatelj (SBD header) | OIB `11528564544` (PTS tester) |
| Primatelj | OIB `99999999994` |
| Message-ID | `ec26595d-6765-11f1-bfbf-364e5b39ae80@racunai.hr` |
| AS4 endpoint primatelja | `https://cis.porezna-uprava.hr:8411/EracunAS4CT/services/msh` |
| Status u Domibusu | **ACKNOWLEDGED** (13. 06. 2026., **22:24:50** CEST) |

Poruka je tehnički uspješno poslana i potvrđena od strane PU demo AS4 endpointa, ali PTS portal **ne ažurira status testa** (ostaje „U tijeku“ ili neuspješno).

Raniji pokušaj (13. 06. 2026., **21:52**): račun `13062026-TP-601`, Message-ID `704f26d9-6761-11f1-be6e-a6393626edf9@racunai.hr` — također **ACKNOWLEDGED**.

Molimo provjeru zašto PTS ne korelira uspješno poslan eRačun s aktivnim test runom unatoč ACKNOWLEDGED statusu.

---

## 2. Zaprimanje eRačuna — nema dolaznih zahtjeva od Porezne uprave

**Endpoint:** `https://as4-test.racunai.hr/EracunAS4/services/msh`  
**Certifikat:** CN=**FISKAL 2**, OIB 36619131370, izdavatelj **Fina Demo CA 2020**  
**Način:** Identifikator nije objavljen u AMS-u (ručni endpoint + certifikat)  
**Primatelj:** OIB poslovnog subjekta **36619131370** (Fine Star)

Testovi **Zaprimanje eRačuna** (13. 06. 2026., npr. **22:02:46–22:02:59** i kasnije): status **Neuspješno**.

Analiza logova na našem serveru:

- Traefik access log
- Domibus Tomcat access log

U vremenskim prozorima PTS testova **nema niti jednog dolaznog POST zahtjeva** od Porezne uprave na `/EracunAS4/services/msh`. Endpoint je dostupan (GET → HTTP 200). Jedini vanjski promet su GET zahtjevi (provjera dostupnosti u browseru).

Testirali smo i direktni HTTP pristup na IP:port radi dijagnostike — isti rezultat: **nema dolaznih AS4 poruka od PU-a**.

Molimo provjeru zašto PTS test ne pokreće slanje eRačuna prema našem endpointu tijekom scenarija Zaprimanje.

---

## 3. MPS — Kreiranje zapisa u AMS-u (InternalError)

Poziv **CreateParticipantIdentifier** prema AMS-u (demo okolina, publisher **MPS36619131370**) vraća:

**HTTP 500 — InternalErrorFault**  
*„Dogodila se interna greška. Javite se administratoru AMS-a.“*

Naš MPS servis (`https://mps.racunai.hr/EracunMPS/`) radi i odgovara na GET upite metapodataka. Registracija publishera u AMS-u ne prolazi zbog greške na strani AMS-a.

---

## 4. Fiskalizacija — greška certifikata (S003)

Slanje testne fiskalizacijske poruke na CIS (PTS okolina, port 8511) vraća:

**S003** — *Certifikat nije izdan od davatelja usluga s Pouzdanog popisa ili je istekao ili je ukinut.*

Koristimo demo certifikat FINA-e (`.p12`, CN=**FISKAL 2**, OIB **36619131370**, Fina Demo CA 2020). Molimo provjeru je li demo certifikat registriran u pouzdanom popisu za PTS/CIS testnu okolinu.

---

## 5. Opća napomena — PTS ne prikazuje detalje grešaka

Portal za testiranje sukladnosti za sve navedene scenarije prikazuje samo status **Neuspješno** ili **U tijeku**, bez tehničkog objašnjenja uzroka. Zbog toga nismo u mogućnosti samostalno dijagnosticirati je li problem u konfiguraciji testa, u orkestraciji na strani PU-a ili u našoj infrastrukturi — osim analizom vlastitih logova (navedeno gore).

---

## Sažetak tehničkog stanja na našoj strani

| Scenarij | Naša infrastruktura | PTS portal |
|----------|---------------------|------------|
| Slanje eRačuna | ACKNOWLEDGED ✅ (2 potvrđena slanja) | U tijeku / neuspješno ⏳❌ |
| Zaprimanje eRačuna | Endpoint OK, **0 POST od PU** | Neuspješno ❌ |
| MPS Kreiranje zapisa | MPS servis OK | AMS HTTP 500 ❌ |
| Fiskalizacija | Pipeline OK | CIS S003 ❌ |

**Tehnički identifikatori:**

| Parametar | Vrijednost |
|-----------|------------|
| OIB subjekta | 36619131370 |
| AS4 endpoint | `https://as4-test.racunai.hr/EracunAS4/services/msh` |
| MPS URL | `https://mps.racunai.hr` |
| MPS Publisher ID | MPS36619131370 |
| AP Party ID | FISKAL 2 |
| Infrastruktura | EU (Hetzner, Helsinki), DNS-only za AS4/MPS |
| Tester | Ante Vrcan, OIB 11528564544 |

---

Molimo pomoć pri dijagnostici navedenih problema i upute za daljnje korake kako bismo uspješno završili testiranje sukladnosti u okviru svrhe **„Za sebe“**.

Spremni smo po potrebi dostaviti dodatne logove (Traefik, Domibus, točne timestampove testova) ili sudjelovati u koordiniranom testu u dogovorenom terminu.

S poštovanjem,

**Ante Vrcan**  
Tester (OIB 11528564544), u ime **Fine Star d.o.o.** (OIB 36619131370)  
Email: avrcan@finestar.hr  
Telefon: +385 97 671 3511
