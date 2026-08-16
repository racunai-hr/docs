# 2026-06-12 — FINA Demo aplikacijski certifikat (email poslan)

**Track 0** · **Preduvjet** za CIS DEMO / PTS testiranje  
**Status:** poslano — čeka potvrdu zaprimanja i aktivacijske podatke od Fine

---

## Sažetak

Poslan je email na **certifikati-fiskalizacija@fina.hr** s potpunom dokumentacijom za izdavanje **Demo aplikacijskog certifikata za fiskalizaciju** (Fina Demo CA 2020, profil FISKAL).

| Polje | Vrijednost |
|-------|------------|
| Datum slanja | 2026-06-12 |
| Primatelj | certifikati-fiskalizacija@fina.hr |
| Pošiljatelj | avrcan@finestar.hr (Toni Šupe) |
| Kontakt mobitel | +385 97 671 3511 |
| Skrbnik certifikata | Toni Šupe (OIB 63281973348) |
| Poslovni subjekt | Fine Star d.o.o. (OIB 36619131370) |

---

## Privitci (3)

Datoteke na disku: [`../../porezna/zahtjevi/`](../../porezna/zahtjevi/)

| # | Datoteka | Opis |
|---|----------|------|
| 1 | `ZahtjevDemoFiskal_FineStar_36619131370.pdf` | Potpisani Zahtjev (obrazac D20) |
| 2 | `FineStar_OI_skrbnik_Toni_Supe_prednja.jpg` | OI skrbnika — prednja strana |
| 3 | `FineStar_OI_skrbnik_Toni_Supe_straznja.jpg` | OI skrbnika — stražnja strana |

---

## Tekst poslanog emaila

**Predmet:** Zahtjev za izdavanje Demo aplikacijskog certifikata za fiskalizaciju — Fine Star d.o.o. (OIB 36619131370)

---

Poštovani,

u privitku dostavljamo potpunu dokumentaciju za **izdavanje Demo aplikacijskog certifikata za fiskalizaciju** (Fina Demo CA 2020, profil FISKAL), u sklopu pripreme za **Fiskalizaciju 2.0** i testiranje sukladnosti na Portalu Porezne uprave (CIS DEMO okolina).

**Podaci o poslovnom subjektu:**
- Naziv: Fine Star d.o.o.
- OIB: 36619131370
- Matični broj: 080885494
- Sjedište: Bana Josipa Jelačića 58, 22000 Šibenik

**Skrbnik certifikata:** Toni Šupe (OIB: 63281973348)

**Traženi certifikat:**
- Demo aplikacijski certifikat standardne razine sigurnosti (NCP)
- Certificijsko tijelo: Fina Demo CA 2020
- Naziv certifikata: FISKAL

**Privitci (3):**
1. `ZahtjevDemoFiskal_FineStar_36619131370.pdf` — potpisani Zahtjev (obrazac D20)
2. `FineStar_OI_skrbnik_Toni_Supe_prednja.jpg` — preslika osobne iskaznice skrbnika (prednja strana)
3. `FineStar_OI_skrbnik_Toni_Supe_straznja.jpg` — preslika osobne iskaznice skrbnika (stražnja strana)

Molimo **potvrdu zaprimanja** dokumentacije i informaciju o **daljnjim koracima** te roku za **dostavu aktivacijskih podataka** (SMS na mobitel i e-poštu, prema odabiru u Zahtjevu).

**Kontakt za odgovor:**  
avrcan@finestar.hr | +385 97 671 3511

S poštovanjem,

Toni Šupe  
Fine Star d.o.o.  
avrcan@finestar.hr  
+385 97 671 3511

---

## Sljedeći koraci (FINA procedura)

Prema [FINA proceduri](../procedures/fina-demo-certifikat.md):

- [ ] Primiti **potvrdu zaprimanja** od Fine (email)
- [ ] Skrbnik prima **aktivacijske podatke** — referentni broj (email) + autorizacijski kod (SMS na +385 97 671 3511)
- [ ] **Online preuzimanje** Demo certifikata na portalu za preuzimanje demo certifikata
- [ ] Spremiti `.p12` + lozinku na sigurno mjesto (ne u git) — `/run/secrets/fiscal-cert/` na serveru
- [ ] Ažurirati status u [`../status/track0-preduvjeti.md`](../status/track0-preduvjeti.md)

---

## Bilješke

- U Zahtjevu odabrana dostava aktivacijskih podataka: **SMS + e-pošta**
- Demo certifikat služi za **CIS DEMO** i PTS testiranje; produkcijski certifikat ide zasebno (OSPD + uplata)
- Referenca: [FINA — izdavanje certifikata za fiskalizaciju](https://www.fina.hr/poslovni-digitalni-certifikati/poslovni-certifikati-za-fiskalizaciju/izdavanje-certifikata-za-fiskalizaciju)

---

## Changelog

| Datum | Promjena |
|-------|----------|
| 2026-06-12 | Email poslan s 3 privitka; datoteke preimenovane u `porezna/zahtjevi/` |
