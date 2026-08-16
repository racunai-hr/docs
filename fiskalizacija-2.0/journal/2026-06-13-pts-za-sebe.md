# 2026-06-13 — PTS: odluka „Za sebe“ (Faza 1)

## Odluka

Fine Star **ne ide** putem **informacijskog posrednika** (Faza 2) na PTS-u.

Testiranje sukladnosti provodi se kao **pristupna točka za vlastite potrebe** — **čl. 63. ZOF**, bez dokumentacije iz čl. 61. (GDPR paket, ISO 27001, izjave posrednika).

Faza 2 (racunAI posrednik, ISO, popis PU) **odgađa se**.

## PTS postavke

| Polje | Vrijednost |
|-------|------------|
| Svrha testiranja | **Za sebe** |
| Područje | **eRačun, MPS, Fiskalizacija** (proširiti na eIzvještavanje kad admin omogući) |
| Dokumentacija (4 uploada) | **Ne treba** |

## Tehnički identifikatori (Fine Star)

| | |
|---|---|
| OIB subjekta | 36619131370 |
| AS4 endpoint | `https://as4-test.racunai.hr/EracunAS4/services/msh` |
| MPS | `https://mps.racunai.hr` |
| Certifikat | `finestar_demo.cer` / `.p12` (CN FISKAL 2) |

## Sljedeći koraci

1. PTS admin: svrha **Za sebe** + područje eRačun/MPS/Fiskalizacija
2. **Slanje eRačuna:** POKRENI TEST → GENERIRAJ → `pts_eracun_send` u istoj sesiji
3. **Zaprimanje:** endpoint + cert u modalu; primatelj OIB **36619131370**
4. **Fiskalizacija:** `fiscal_submit_demo` nakon proširenja svrhe
5. Mail PU podršci: fokus na Zaprimanje (PU ne POST-a) + S003 cert, ne na posrednika

Nacrti dokumentacije posrednika ostaju u [`../../porezna/zahtjevi/posrednik/`](../../porezna/zahtjevi/posrednik/) — arhiva za Fazu 2.
