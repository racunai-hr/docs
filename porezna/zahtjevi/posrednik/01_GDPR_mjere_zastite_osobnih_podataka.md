# Dokument o tehničkim i organizacijskim mjerama zaštite osobnih podataka

**u skladu s člankom 32. Uredbe (EU) 2016/679 (GDPR)**

---

## 1. Podaci o voditelju obrade

| | |
|---|---|
| **Naziv** | Fine Star d.o.o. |
| **OIB** | 36619131370 |
| **Matični broj** | 080885494 |
| **Sjedište** | Bana Josipa Jelačića 58, 22000 Šibenik, Republika Hrvatska |
| **Djelatnost** | Računovodstvene i knjigovodstvene usluge; informacijski sustav **racunAI** |
| **Odgovorna osoba** | Toni Šupe, direktor |
| **Kontakt za zaštitu podataka** | Ante Vrcan, avrcan@finestar.hr |

**Informacijski posrednik / pristupna točka:** Fine Star d.o.o. putem platforme **racunAI** (`racunai.hr`), u okviru provedbe Zakona o fiskalizaciji (Fiskalizacija 2.0).

---

## 2. Svrha i opseg dokumenta

Ovaj dokument detaljno opisuje tehničke i organizacijske mjere implementirane radi osiguranja razine sigurnosti osobnih podataka u skladu s čl. 32. GDPR-a, u kontekstu:

- razmjene eRačuna (AS4),
- metapodatkovnog servisa (MPS),
- fiskalizacije i eIzvještavanja eRačuna (CIS),
- ERP sustava racunAI.

Obrada uključuje poslovne podatke klijenata (OIB, naziv, adresa, podaci s računa), podatke o transakcijama i tehničke identifikatore u porukama fiskalizacije/eRačuna.

---

## 3. Tehničke mjere

### 3.1. Lokacija i infrastruktura

- Produkcijska infrastruktura smještena je u **Europskoj uniji** (Hetzner Online GmbH, datacentar **Helsinki, Finska**).
- Nema namjernog prijenosa osobnih podataka izvan EU/EEA.
- DNS zapisi za javne servise (`mps.racunai.hr`, `as4-test.racunai.hr`) upravljani uz **DNS-only** (bez proxyja izvan EU kontrole).

### 3.2. Šifriranje i komunikacija

- Svi javni servisi dostupni isključivo putem **HTTPS (TLS 1.2+)**; certifikati Let's Encrypt / Cloudflare DNS challenge.
- AS4 komunikacija prema specifikaciji Porezne uprave: **WS-Security**, potpis poruka aplikacijskim certifikatom (FINA PKI, profil FISKAL).
- Fiskalizacijske poruke potpisane **XAdES-B** demo/produkcijskim certifikatom pohranjenim u zaštićenom volumenu (`.certificates/`, pristup ograničen na Docker volume, lozinka u `.secret` izvan repozitorija).
- Baza podataka ERP-a: PostgreSQL u privatnoj Docker mreži; pristup samo iz aplikacijskog sloja.

### 3.3. Kontrola pristupa

- ERP: autentifikacija korisnika, tenant izolacija (multi-tenant arhitektura), role-based pristup.
- Poslužitelj: SSH pristup ograničen; administracija putem Tailscale VPN gdje je primjenjivo.
- Domibus admin i WS plugin: dostupni samo na internal rutama / privatnoj mreži, ne javno.
- `.env` datoteke i tajne **nisu** u git repozitoriju.

### 3.4. Mrežna sigurnost

- Traefik reverse proxy s automatskim HTTPS preusmjeravanjem.
- CrowdSec + Fail2ban na razini hosta (AS4 endpoint izuzet od CrowdSec middleware-a radi PU/PTS klijenata).
- Firewall: UFW neaktivan na hostu; iptables s Fail2ban za SSH.

### 3.5. Logiranje i nadzor

- Aplikacijski logovi: Django (`erp/logs/django.log`), Domibus (Tomcat access + application log), Traefik JSON access log.
- Logovi sadrže tehničke identifikatore (IP, Message-ID, OIB u poslovnom kontekstu); retencija prema operativnim potrebama.
- Docker healthcheck na MPS i bazi podataka.

### 3.6. Sigurnost razvoja i promjena

- Verzioniranje koda (git); promjene fiskalnog modula (`fiscal_gateway`) kroz code review.
- Unit testovi za AS4 inbound handler i fiskalne servise prije produkcijskog deploya.
- Odvojeni demo i produkcijski certifikati; demo okolina (`as4-test`, CIS demo portovi) odvojena od produkcije.

### 3.7. Pouzdanost i kontinuitet

- Docker Compose s `restart: unless-stopped` na kritičnim servisima.
- MySQL (Domibus) i PostgreSQL (ERP) na persistent volumenima.
- Redoviti backupi infrastrukture (Hetzner / operativna praksa administratora).

### 3.8. Obrada putem informacijskog posrednika (MPS/AS4)

- MPS servis (`https://mps.racunai.hr/EracunMPS/...`) vraća metapodatke primatelja; ne pohranjuje sadržaj eRačuna duže od operativne potrebe.
- AS4 gateway (Domibus) prima i šalje poruke prema ebMS 3.0 / AS4 specifikaciji; payload validacija UBL-om pri zaprimanju.

---

## 4. Organizacijske mjere

### 4.1. Upravljanje i odgovornosti

- Direktor odgovoran za usklađenost s propisima o zaštiti podataka.
- Ante Vrcan — skrbnik tehničke infrastrukture i FINA certifikata (ovlašten punomoć).
- Toni Šupe — zastupnik i odgovorna osoba za poslovne odluke.

### 4.2. Obuka i svijest

- Osoblje s pristupom ERP-u i poslužitelju upoznato s obvezom povjerljivosti poslovnih podataka klijenata.
- Pristup produkciji ograničen na ovlaštene osobe.

### 4.3. Ugovori s obrađivačima

- Hetzner Online GmbH (hosting, EU) — ugovor o obradi u skladu s GDPR-om.
- FINA (certifikati PKI) — izdavanje i upravljanje aplikacijskim certifikatima za fiskalizaciju.

### 4.4. Prava ispitanika

- Klijenti Fine Star d.o.o. ostvaruju prava ispitanika (pristup, ispravak, brisanje, ograničenje) putem pisane molbe na avrcan@finestar.hr.
- Zahtjevi se obrađuju u roku od 30 dana.

### 4.5. Procjena utjecaja (DPIA)

- Za fiskalizaciju 2.0 i eRačun obrada je nužna radi zakonske obveze; rizici mitigirani mjerama iz ovog dokumenta.
- Detaljna DPIA bit će ažurirana u sklopu ISMS-a (ISO 27001).

### 4.6. Incidenti i povrede podataka

- U slučaju sumnje na povredu osobnih podataka: interna eskalacija na direktora i DPO kontakt unutar 24 h.
- Obavijest PUZO-u (AZOP) u roku od 72 h ako postoji rizik za prava ispitanika, u skladu s čl. 33. GDPR-a.
- Evidencija incidenata vodi se u internom zapisniku.

---

## 5. Kategorije osobnih podataka i rokovi

| Kategorija | Primjer | Rok |
|------------|---------|-----|
| Identifikacijski podaci | OIB, naziv tvrtke | Trajanje ugovora + zakonski rok arhiviranja |
| Kontakt podaci | Adresa, email | Isto |
| Financijski podaci | Računi, stavke | Prema Zakonu o računovodstvu i poreznim propisima |
| Tehnički logovi | IP, Message-ID | Operativno, rotacija logova |

---

## 6. Ažuriranje

Ovaj dokument pregledava se najmanje **jednom godišnje** ili nakon značajne promjene infrastrukture.

| Verzija | Datum | Opis |
|---------|-------|------|
| 1.0 | 13. 06. 2026. | Inicijalna verzija za PTS dokumentaciju posrednika |

---

**Fine Star d.o.o.**

Mjesto i datum: Šibenik, 13. 06. 2026.

Potpis odgovorne osobe: _________________________

**Toni Šupe**, direktor
