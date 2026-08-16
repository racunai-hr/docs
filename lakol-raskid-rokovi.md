# Lakol Šibenik — raskid ugovora o računovodstvenim uslugama (Fine Star d.o.o.)

Dokumentacija rokova i izvora za prijelaz s Lakola na vlastito računovodstvo (ERP / racunai.hr).

## Izvori u repou

| Datoteka | Opis |
|----------|------|
| [Fwd_ FINE STAR OBAVIJEST...eml](./Fwd_%20FINE%20STAR%20OBAVIJEST%20O%20RASKIDU%20UGOVORA%20O%20OBAVLJANJU%20RA%C4%8CUNOVODSTVENIH%20...USLUGA.eml) | Proslijeđena poruka (Toni Supe → avrcan@finestar.hr, 26.5.2026.) |
| `SKM_C301i26052614141.pdf` | **Obavijest o raskidu od Lakola — NIJE u repou** (vidi dolje) |
| [finestar-sudski-registar-2026-05-27.pdf](./finestar-sudski-registar-2026-05-27.pdf) | Izvadak iz sudskog registra (Fina/sudreg), **nije** obavijest o raskidu |

## Lanac poruka (potvrđeno iz .eml i maildira)

1. **Lakol Šibenik d.o.o.** (`info@lakol.hr`) → Toni Supe (`tonisupe7@gmail.com`)
   - **Datum:** 26. svibnja 2026., 14:23:52 CEST
   - **Predmet:** FINE STAR OBAVIJEST O RASKIDU UGOVORA O OBAVLJANJU RAČUNOVODSTVENIH USLUGA
   - **Prilog:** `SKM_C301i26052614141.pdf` (Obavijest o raskidu ugovora)
   - **Message-ID:** `<001601dced0a$84111480$8c333d80$@lakol.hr>`

2. **Toni Supe** → Ante Vrcan (`avrcan@finestar.hr`)
   - **Datum:** 26. svibnja 2026., 20:15:29 CEST
   - Proslijeđeno s iPhonea — **prilog nije uključen** u .eml (samo tekstualna referenca na PDF)

## Rokovi iz obavijesti Lakola

> **Status:** konkretni rokovi nisu ekstrahirani — originalni PDF `SKM_C301i26052614141.pdf` nije dostupan na serveru niti u proslijeđenom mailu (iPhone forward bez attachmenta). Pretraženi su maildir (`avrcan@finestar.hr`), `/opt/stacks` i cijeli repozitorij.

| Stavka | Vrijednost | Izvor |
|--------|------------|-------|
| Datum obavijesti o raskidu | **26.05.2026.** | Mail Lakola / proslijeđena poruka |
| Datum prestanka usluge Lakola | **TBD** | PDF `SKM_C301i26052614141.pdf` |
| Rok predaje knjigovodstvene dokumentacije | **TBD** | PDF `SKM_C301i26052614141.pdf` |
| Zadnji obračunski period koji Lakol vodi | **TBD** | PDF `SKM_C301i26052614141.pdf` |

### Kako dopuniti (jedan korak)

1. Preuzeti `SKM_C301i26052614141.pdf` iz originalne poruke na `tonisupe7@gmail.com` (Lakol, 26.5.2026., 14:23) — ne iz proslijeđene poruke bez priloga.
2. Spremiti u repozitorij kao:

   ```
   docs/SKM_C301i26052614141.pdf
   ```

3. Ažurirati tablicu rokova u ovom dokumentu (tri TBD polja).

Alternativa: ponovno proslijediti mail **s prilogom** na `avrcan@finestar.hr`, zatim ekstrahirati PDF iz maildira.

## Kontakt Lakol

- **Tvrtka:** LAKOL ŠIBENIK d.o.o. za računovodstvo i poslovne usluge
- **Adresa:** Petra Preradovića 7, 22000 Šibenik
- **OIB:** 83651629490
- **Email:** info@lakol.hr, marijana@lakol.hr
- **Tel.:** +385 22 200 032 / +385 22 213 780

## Povezani podaci (Fine Star — sudski registar)

Iz [finestar-sudski-registar-2026-05-27.pdf](./finestar-sudski-registar-2026-05-27.pdf) (izvadak od 27.05.2026.):

| Podatak | Vrijednost |
|---------|------------|
| Tvrtka | FINE STAR d.o.o. za usluge |
| MBS | 080885494 |
| OIB | 36619131370 |
| Sjedište | Bana Josipa Jelačića 58, 22000 Šibenik |
| Direktor | Toni Šupe (OIB 63281973348) |
| Zadnji predani GFI (u registru) | Godina **2025**, razdoblje **01.01.2025 – 31.12.2025**, predano **28.04.2026.** (GFI-POD) |

Ovi podaci pomažu kod prijelaza knjiga, ali **ne zamjenjuju** rokove iz Lakolove obavijesti o raskidu.

## Napomena o PDF-u s Windowsa

Datoteka `009Bc-cishH-dWq32-31AiM-bgrB9-oQZoX-UYdaf-Dw7TO-XMY1H.pdf` (Downloads) identificirana je kao **izvadak iz sudskog registra**, ne obavijest o raskidu. Kopija je u repou pod imenom `finestar-sudski-registar-2026-05-27.pdf`.

---

*Zadnje ažuriranje: 8. lipnja 2026.*
