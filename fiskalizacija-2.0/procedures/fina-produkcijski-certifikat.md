# Procedura — FINA produkcijski aplikacijski certifikat

**Ne započinjati** dok nije gotov **korak 00 Preduvjet** (NKD prijepis od DZS-a).

Izvor: [FINA — izdavanje certifikata](https://www.fina.hr/poslovni-digitalni-certifikati/poslovni-certifikati-za-fiskalizaciju/izdavanje-certifikata-za-fiskalizaciju)

**Lokalni obrasci:** [`../../porezna/zahtjevi/produkcijski/`](../../porezna/zahtjevi/produkcijski/)

---

## Preduvjet — korak 00 (FINA PKI, prvi certifikat)

| Dokument | Status |
|----------|--------|
| Izvadak iz sudskog registra (≤ 6 mj.) | [x] [`../../finestar-sudski-registar-2026-05-27.pdf`](../../finestar-sudski-registar-2026-05-27.pdf) |
| Obavijest DZS — NKD 2025. | [ ] čeka se — [journal](../journal/2026-06-12-dzs-nkd-molba.md) |
| OI zastupnika (Toni Šupe) | [x] [`../../porezna/zahtjevi/FineStar_OI_skrbnik_Toni_Supe_*.jpg`](../../porezna/zahtjevi/) |
| OI skrbnika (Ante Vrcan) | [x] [`FineStar_OI_skrbnik_Ante_Vrcan_*.jpg`](../../porezna/zahtjevi/) · [journal](../journal/2026-06-12-fina-oi-ante-vrcan.md) |
| Potvrda boravišta (Ante Vrcan) | [x] [`FineStar_boraviste_Ante_Vrcan_2026-05-19.jpg`](../../porezna/zahtjevi/FineStar_boraviste_Ante_Vrcan_2026-05-19.jpg) · [journal](../journal/2026-06-12-fina-boraviste-ante-vrcan.md) |
| Punomoć za potpis zahtjeva/ugovora | [x] [`FineStar_Punomoc_zahtjev_ugovor_fiskalizacija.pdf`](../../porezna/zahtjevi/FineStar_Punomoc_zahtjev_ugovor_fiskalizacija.pdf) — [journal](../journal/2026-06-12-fina-punomoc-fiskalizacija.md) |

---

## Korak 01 — Obrasci (pripremljeno u repou)

Službeni obrasci su preuzeti i spremljeni u `porezna/zahtjevi/produkcijski/`:

| Obrazac | Lokalna datoteka | Napomena |
|---------|------------------|----------|
| **Ugovor o certificiranju (OSPD)** | [`FineStar_UgovorAplikacijskieIDAS_OSPD_obrazac.pdf`](../../porezna/zahtjevi/produkcijski/FineStar_UgovorAplikacijskieIDAS_OSPD_obrazac.pdf) | Ispravan FINA ugovor; **popuniti + e-potpis** |
| **Zahtjev za certifikat** | [`FineStar_ZahtjevCertFiskal_36619131370.pdf`](../../porezna/zahtjevi/produkcijski/FineStar_ZahtjevCertFiskal_36619131370.pdf) | [x] **Popunjen** 2026-06-12 — [journal](../journal/2026-06-12-fina-produkcijski-zahtjev-popunjen.md) |
| Prazan obrazac (referenca) | [`ZahtjevCertFiskal_FINA_obrazac.pdf`](../../porezna/zahtjevi/produkcijski/ZahtjevCertFiskal_FINA_obrazac.pdf) | arhiva |

### Provjera ugovora (OSPD)

Ugovor u repou odgovara službenom [`UgovorAplikacijskieIDAS_OSPD.pdf`](https://rdc.fina.hr/obrasci/UgovorAplikacijskieIDAS_OSPD.pdf):

- Naslov: *Ugovor o obavljanju usluga certificiranja za poslovne subjekte – Aplikacijski certifikati*
- Namijenjen **OSPD** elektroničkoj predaji (e-potpis)
- Polja za popunjavanje: naziv tvrtke, sjedište + OIB, zastupnik, potpis

**Popuniti:**

```
Fine Star d.o.o.
Republika Hrvatska, 22000 Šibenik, Bana Josipa Jelačića 58, OIB 36619131370
Toni Šupe, direktor
```

Skrbnik certifikata (u popunjenom Zahtjevu): **Ante Vrcan** (OIB 11528564544, avrcan@finestar.hr). Potpisuje na temelju punomoći [`FineStar_Punomoc_zahtjev_ugovor_fiskalizacija.pdf`](../../porezna/zahtjevi/FineStar_Punomoc_zahtjev_ugovor_fiskalizacija.pdf) — opunomoćitelj **Toni Šupe**, direktor (OIB 63281973348), datum 12. 6. 2026.

**Prije predaje dopuniti/ispraviti:** u zahtjevu polje **Vrijedi do** OI (12. 5. 2030), potpisi.

---

## Korak 02 — Uplata

| Stavka | Iznos |
|--------|-------|
| Produkcijski certifikat (5 god.) | 39,82 € + PDV |
| **Uplata sada** | **49,78 €** (s PDV-om) |

- IBAN: **HR42 2390 0011 1000 0170 42**
- Model: **HR05**
- Poziv na broj: **7544103-36619131370**

Prva registracija u FINA PKI (+10,62 € + PDV) — račun kasnije, ako prvi put u sustavu.

---

## Korak 03 — Predaja (OSPD)

1. Popuniti **ZahtjevCertFiskal** + **Ugovor OSPD**
2. Potpisati **naprednim e-potpisom** (kvalificirani certifikat)
3. Upload na https://ospd.fina.hr/
4. Prilozi: korak 00 (sudski registar + **NKD** + OI), **punomoć**, dokaz uplate

Alternativa: FINA poslovnica (Split/Zadar) — papirnato (3× ugovor).

Email dopune: **certifikati-fiskalizacija@fina.hr**

---

## Korak 04 — Preuzimanje `.p12`

1. Skrbnik: referentni broj (email) + autorizacijski kod (SMS)
2. Online preuzimanje na FINA portalu
3. Lozinka za `.p12` — spremiti sigurno (ne u git)

---

## Checklist prije predaje

- [ ] NKD prijepis od DZS
- [x] ZahtjevCertFiskal popunjen → [`FineStar_ZahtjevCertFiskal_36619131370.pdf`](../../porezna/zahtjevi/produkcijski/FineStar_ZahtjevCertFiskal_36619131370.pdf)
- [x] Punomoć Toni → Ante → [`FineStar_Punomoc_zahtjev_ugovor_fiskalizacija.pdf`](../../porezna/zahtjevi/FineStar_Punomoc_zahtjev_ugovor_fiskalizacija.pdf)
- [ ] Ugovor OSPD popunjen i e-potpisan
- [ ] Uplata 49,78 €
- [ ] Prilozi: sudski registar, NKD, OI (Toni + Ante), boravište Ante, punomoć, dokaz uplate
- [ ] OSPD upload

---

## Redoslijed u projektu

```
1. Demo certifikat (.p12)     ← u tijeku (email 2026-06-12)
2. NKD prijepis (DZS)         ← čekamo — BLOKATOR za produkcijsku predaju
3. PTS test (CIS DEMO)        ← nakon Demo .p12 + ePorezna
4. Produkcijski certifikat    ← Zahtjev popunjen; čeka NKD + ugovor + uplata
```

Produkcijski certifikat **nije potreban** za PTS test u DEMO okolini — ali **korak 00** (NKD) može trebati i za dovršetak Demo izdavanja ako je Fine Star prvi put u FINA PKI.
