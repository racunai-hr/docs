# Fiskalizacija 2.0 — praćenje projekta

Projekt: **Fine Star d.o.o.** → vlastita pristupna točka (Faza 1) → **racunAI** informacijski posrednik (Faza 2).

| Podatak | Vrijednost |
|---------|------------|
| Tvrtka | Fine Star d.o.o. |
| OIB | 36619131370 |
| Matični broj | 080885494 |
| Infrastruktura | Hetzner dedicated, Finska (EU) |
| Platforma | racunAI (`racunai.hr`) |

## Struktura dokumentacije

```
fiskalizacija-2.0/
├── README.md                 ← ovaj indeks
├── milestones.md             ← milestone-i i implementacijski plan
├── status/
│   └── track0-preduvjeti.md  ← checklist preduvjeta (FINA, PTS, DNS, ISO)
├── journal/                  ← kronološki zapis akcija
│   ├── 2026-06-12-dzs-nkd-molba.md
│   ├── 2026-06-12-eporezna-punomoc-porezna.md
│   ├── 2026-06-12-fina-demo-certifikat.md
│   └── 2026-06-13-pts-tester.md
└── procedures/               ← procedure po temama
    ├── fina-demo-certifikat.md
    ├── fina-produkcijski-certifikat.md
    ├── eporezna-pristup.md
    └── pts-fiskalizacija-test.md
```

**Službene specifikacije:** [`../porezna/`](../porezna/)  
**Zahtjevi / OI:** [`../porezna/zahtjevi/`](../porezna/zahtjevi/)  
**Kontakti / email pravila:** [`kontakti-i-emailovi.md`](kontakti-i-emailovi.md)

## Trenutni status (sažetak)

| Track / Milestone | Status |
|-------------------|--------|
| Specifikacije Porezne | Gotovo (`docs/porezna/`) |
| FINA Demo certifikat | **Preuzet** — `.certificates/36619131370.F2.2.p12` |
| FINA korak 00 (NKD) | **Sudski registar [x]** — **NKD čekamo (DZS)** |
| FINA Produkcijski certifikat | **Zahtjev + punomoć** spremni — čeka NKD, ugovor OSPD, uplata |
| PTS / ePorezna | **Tester aktivan** — punomoć 478876 |
| DNS (`mps`, `as4-test`) | Nije započeto |
| ISO 27001 | Nije započeto |
| `fiscal_gateway` (kod) | **U implementaciji** (M1.1–M1.3) |

Detalji: [`status/track0-preduvjeti.md`](status/track0-preduvjeti.md)

## Journal (zadnje akcije)

| Datum | Događaj |
|-------|---------|
| 2026-06-12 | [DZS molba — prijepis NKD 2025.](journal/2026-06-12-dzs-nkd-molba.md) |
| 2026-06-12 | [ePorezna punomoć — Toni predaje u PU (kôd 478876)](journal/2026-06-12-eporezna-punomoc-porezna.md) |
| 2026-06-12 | [FINA produkcijski Zahtjev popunjen](journal/2026-06-12-fina-produkcijski-zahtjev-popunjen.md) |
| 2026-06-12 | [Potvrda boravišta Ante Vrcan](journal/2026-06-12-fina-boraviste-ante-vrcan.md) |
| 2026-06-12 | [OI skrbnika Ante Vrcan](journal/2026-06-12-fina-oi-ante-vrcan.md) |
| 2026-06-12 | [FINA punomoć — Toni → Ante (zahtjev/ugovor)](journal/2026-06-12-fina-punomoc-fiskalizacija.md) |
| 2026-06-13 | [PTS tester + demo cert preuzet](journal/2026-06-13-pts-tester.md) |
| 2026-06-12 | [FINA Demo certifikat — email poslan](journal/2026-06-12-fina-demo-certifikat.md) |

## Faze

- **Faza 1:** Fine Star samo za sebe (PT čl. 63., bez ISO) → PTS test → produkcija
- **Faza 2:** Informacijski posrednik na racunAI (ISO 27001 + čl. 61. + završni PTS)

Vidi [`milestones.md`](milestones.md).
