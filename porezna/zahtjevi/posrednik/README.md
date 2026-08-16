# PTS — dokumentacija informacijskog posrednika (čl. 61. ZOF)

> **Status: odgođeno (Faza 2).** Fine Star testira PTS kao **„Za sebe“** — ovi dokumenti **nisu potrebni** za testiranje. Ostaju kao nacrt za budući racunAI posrednik.

**Subjekt:** Fine Star d.o.o., OIB 36619131370  
**Platforma:** racunAI (`racunai.hr`)  
**Svrha:** upload na PTS portal → **Dokumentacija**

## Datoteke za upload

| # | PTS polje | Datoteka (izvor) | PDF za upload |
|---|-----------|------------------|---------------|
| 1 | GDPR | [`01_GDPR_mjere_zastite_osobnih_podataka.md`](01_GDPR_mjere_zastite_osobnih_podataka.md) | `01_GDPR_mjere.pdf` |
| 2 | ISO certifikat | **Vidi [`02_ISO27001_README.md`](02_ISO27001_README.md)** | certifikat od certificirajućeg tijela |
| 3 | Izjava o upravljanju sustavom informacija | [`03_Izjava_upravljanje_sustavom_informacija_EU.md`](03_Izjava_upravljanje_sustavom_informacija_EU.md) | `03_Izjava_upravljanje.pdf` |
| 4 | Izjava o opsegu usluga | [`04_Izjava_opseg_usluga.md`](04_Izjava_opseg_usluga.md) | `04_Izjava_opseg.pdf` |

## Pretvorba u PDF

```bash
cd /opt/stacks/racunai.hr/erp/docs/porezna/zahtjevi/posrednik
pandoc 01_GDPR_mjere_zastite_osobnih_podataka.md -o 01_GDPR_mjere.pdf
pandoc 03_Izjava_upravljanje_sustavom_informacija_EU.md -o 03_Izjava_upravljanje.pdf
pandoc 04_Izjava_opseg_usluga.md -o 04_Izjava_opseg.pdf
```

Ako `pandoc` nije instaliran: otvoriti `.md` u Wordu/LibreOffice → Spremi kao PDF.

## Potpis

Izjave (3 i 4) potpisuje **odgovorna osoba** — Toni Šupe, direktor.  
GDPR dokument: potpis direktora + pečat tvrtke (preporuka).

## ISO 27001 — blokator

Zakon traži **važeći certifikat ISO/IEC 27001** (nije zamjenjiv izjavom).  
Fine Star ga trenutno **nema** — vidi `02_ISO27001_README.md` i track 0.14–0.16 u `fiskalizacija-2.0/status/track0-preduvjeti.md`.

Bez ISO certifikata PTS vjerojatno **neće odobriti** dokumentaciju posrednika u potpunosti.

## Pravna osnova

- Zakon o fiskalizaciji (NN 89/25), **čl. 61.** — dokumentacija prije završnog testiranja
- Uredba (EU) 2016/679 (GDPR), **čl. 32.** — tehničke i organizacijske mjere
