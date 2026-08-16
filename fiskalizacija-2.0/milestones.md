# Milestone-i — Fiskalizacija 2.0

## Faza 1 — Fine Star (pristupna točka samo za sebe)

| ID | Milestone | Status | Ovisi o |
|----|-----------|--------|---------|
| M1.1 | `fiscal_gateway` Django app | [x] | — |
| M1.2 | UBL HR profil + XSD/Schematron validator | [x] | 0.2, M1.1 |
| M1.3 | Fiskalizacija + XAdES + eIzvještavanje (CIS DEMO) | [x] | 0.5, M1.2 |
| M1.4 | MPS servis + AMS (Fine Star OIB) | [x] | 0.10, M1.1 |
| M1.5 | AS4 gateway (Java/Spring) | [x] | 0.11, M1.1 |
| M1.6 | Integracija racunAI UI (invoices/expenses) | [x] | M1.3–M1.5 |
| M1.7 | Uklanjanje `super_integration` | [~] | M1.6 — deprecated + rollback flag; app ostaje za povijest |
| M1.8 | PTS test (čl. 63.) + produkcija Fine Star | [~] | 0.7–0.8 — cutover migracija 0008; stability window do 2026-08-01 |

**M1.8 napredak:** Runbook A–E izvršen; migracija `0008_fiscal_prod_cutover` aktivira DIRECT prod. **Verified** retci u matrici popunjeni. **Preostaje:** CIS JIR (S003 cert), PU AMS registracija, NKD → FINA prod cert, završetak 30-day window.

**Kriterij uspjeha Faze 1:** 30 dana stabilnog B2B eRačuna za Fine Star OIB — praćenje: [`procedures/stability-window.md`](procedures/stability-window.md).

### Tehnički izvori (lokalno)

| Milestone | PDF / ZIP |
|-----------|-----------|
| M1.2 | `porezna/Specifikacija osnovne uporabe…`, `UBL2.1 eRačun.zip`, `HRUBLSchematron…` |
| M1.3 | `Tehnička specifikacija_Fiskalizacija…`, `eFiskalizacijaSchema.zip`, `eIzvjestavanjeSchema.zip` |
| M1.4 | `Tehnička specifikacija_eRacun_MPS.pdf`, `Tehnička specifikacija_eRacun_AMS.pdf` |
| M1.5 | `Tehnička specifikacija_eRacun_PT_AS4.pdf` |
| M1.8 | `Procedure za provođenje testiranja.pdf` |

---

## Faza 2 — racunAI informacijski posrednik

| ID | Milestone | Status | Ovisi o |
|----|-----------|--------|---------|
| M2.1 | ISO 27001 + dokumentacija čl. 61. | [ ] | 0.14–0.16 |
| M2.2 | Multi-OIB MPS + onboarding | [ ] | M1.8, M2.1 |
| M2.3 | Posrednički sloj (ugovori, FiskAplikacija ovlaštenja) | [ ] | M2.2 |
| M2.4 | HA infra (2+ instanca AS4, DB replica) | [ ] | M2.3 |
| M2.5 | Završni PTS test posrednika + popis Porezne | [ ] | M2.1–M2.3 |

---

## Arhitektura (cilj)

```
racunAI ERP (Django)
    └── fiscal_gateway
            ├── mps-service      → AMS
            ├── as4-gateway      → druge PT (Java)
            └── signing/CIS      → Porezna fiskalizacija
```

SUPER integracija se uklanja u M1.7.

---

## Implementacijski redoslijed (kod)

1. M1.1 scaffold `fiscal_gateway`
2. M1.2 UBL + validatori
3. M1.3 fiskalizacija (Demo certifikat iz Track 0)
4. M1.4 MPS
5. M1.5 AS4
6. M1.6 UI cutover
7. M1.7 remove SUPER
8. M1.8 PTS

Vidi [`README.md`](README.md) za trenutni status.
