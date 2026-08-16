# Architecture Governance — racunAI ERP

Pravilo projekta (vidi [`NORTH_STAR.md`](NORTH_STAR.md)):

> **Architecture follows the product, not the other way around.**

---

## Tko odobrava ADR

| Uloga | Odgovornost |
|-------|-------------|
| **Architecture Owner** | Odobrava nove ADR-ove, vodi kvartalni review (TBD — vlasnik projekta) |
| **Domain Owner** | Predlaže ADR za svoju domenu (vidi `OWNERSHIP.md` kad postoji) |
| **Reviewer** | Min. 1 review prije mergea ADR-a |

---

## Kada je koji dokument potreban

| Promjena | Potreban dokument | Architecture Review |
|----------|-------------------|---------------------|
| Novi helper u `shared/` | ništa | ne |
| Novi servis u postojećoj domeni | GitHub Issue | ne |
| Novi event | Issue + ažuriranje [`EVENTS.md`](EVENTS.md) | ne |
| Novi connector | RFC (kratki) + Issue + `CONNECTORS.md` | ne |
| Nova domena | **ADR** | da |
| Novi Django app | **ADR** | da |
| Promjena PDV/banking/submission jezgre | **ADR** + Architecture Review | **obavezno** |
| Novi vanjski sustav (banka, provider) | RFC + Issue | ne |
| Nova vrsta dokumenta | **ADR** (amendment na freeze) | da |
| Breaking API promjena | **ADR** + `RELEASE_POLICY` update | da |
| Nova AI capability | Issue (u postojećoj domeni) | ne |
| Infrastrukturna promjena (Redis cluster) | RFC + Issue | ne |

---

## Architecture Decision Matrix

Svaki ADR označava **tip odluke** (stupac u [`ADR_INDEX.md`](ADR_INDEX.md)):

| Tip | Primjer | ADR prefix |
|-----|---------|------------|
| **Business** | PDV box 610, saldakonti pravila | ADR-BIZ- |
| **Domain** | Nova domena Inventory | ADR-DOM- |
| **Integration** | HPB bank connector | ADR-INT- |
| **Infrastructure** | Redis cluster | ADR-INF- |
| **Security** | MFA implementacija | ADR-SEC- |
| **Compliance** | GDPR retention policy | ADR-COM- |
| **Performance** | Queue redesign | ADR-PERF- |
| **Governance** | Architecture Freeze | ADR-GOV- |

Retroaktivno mapiranje ADR-0007–0011:

| ADR | Tip |
|-----|-----|
| ADR-0007 | Business |
| ADR-0008 | Compliance |
| ADR-0009 | Domain |
| ADR-0010 | Domain |
| ADR-0011 | Governance |

---

## Kvartalni Architecture Review

Svaka 3 mjeseca:

1. Pregled **Module Maturity** levels u [`DOMAIN_MAP.md`](DOMAIN_MAP.md) — pomak L(n)→L(n+1)?
2. Pregled `TECHNICAL_DEBT_REGISTRY` — prioriteti još važeći?
3. Pregled `CONNECTORS.md` — statusi ažurni?
4. Pregled [`ROADMAP.md`](ROADMAP.md) — faza na tragu?
5. KPI snapshot (coverage, open bugs, tech debt count)
6. Zapis u [`ADR_INDEX.md`](ADR_INDEX.md) (review datum, bez novog ADR-a osim ako treba)

Sljedeći review: **2026-10-09** (Q4 2026).

---

## Operativno pravilo (nakon Freeze v1.0)

> Vrijeme ulagati u implementaciju. Novi dokumenti nastaju samo kada razvoj otkrije stvarnu potrebu za novom arhitekturnom odlukom.

- Nova funkcionalnost → ažurirati samo relevantne living dokumente
- Ne otvarati nove dokumente bez ADR-a
- Code has priority over documentation unless architecture changes

Vidi [`ADR-0011-architecture-freeze-v1.0.md`](ADR-0011-architecture-freeze-v1.0.md).
