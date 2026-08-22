# North Star — racunAI ERP

> **Architecture follows the product, not the other way around.**

Ne razvijamo funkcionalnosti zato što se lijepo uklapaju u arhitekturu. Arhitektura se prilagođava stvarnim poslovnim potrebama. Svaka nova domena mora riješiti konkretan problem korisnika.

---

## Vizija

> racunAI je cloud-native, multi-tenant hrvatski ERP koji pokriva cjelokupno poslovanje poduzetnika, s naglaskom na zakonsku usklađenost, automatizaciju i umjetnu inteligenciju.

---

## Načela

1. **Zakonska usklađenost ima prednost** — PDV, fiskalizacija, eRačun, računovodstveni standardi.
2. **Poslovna logika pripada domenama** — `domains/*/services/`, ne Django modelima ni viewovima.
3. **Automatizacija prije ručnog rada** — knjiženje, PDV agregacija, bankovno usklađivanje.
4. **AI pomaže korisniku, ali ne donosi nepovratne odluke bez pregleda** — OCR, auto-kontiranje, provjera knjiženja.
5. **Sve važne poslovne radnje su auditabilne** — `SubmissionEvent`, `AuditLog`, immutable accounting.
6. **Sustav je API-first i integracijski orijentiran** — OpenAPI, connectori, webhooks.
7. **Multi-tenant je zadani način rada** — izolacija podataka, tenant-scoped servisi.

---

## North Star KPI

> **Koliko novih klijenata može prijeći na racunAI bez Excela ili drugog ERP-a?**

Najvažniji pokazatelj uspjeha — ne broj modula, nego stvarna mogućnost vođenja poslovanja u racunAI-ju.

| Verzija | Samostalno vođenje poslovanja |
|---------|------------------------------:|
| v1.0 (Faza 1) | 10% |
| v1.5 | 40% |
| v2.0 (Faza 2) | 70% |
| v3.0 (Faza 3) | 90% |

Svaki sprint povećava ovaj postotak. Sprint retrospectives (ADR) bilježe koji blockeri su uklonjeni.

**Trenutno (2026-08-22):** ~35% — Sprint 3 zatvoren ([`ADR-0014`](ADR-0014-tax-domain-completion.md)); od tada isporučeni operativni slojevi Documents read model ([`ADR-0020`](ADR-0020-document-read-model.md)), Banking UI + reconcile ([`ADR-0021`](ADR-0021-banking-operational-ui-api.md), [`ADR-0025`](ADR-0025-bank-reconcile-open-item.md)), Partner MDM ([`ADR-0022`](ADR-0022-partner-management-mdm-api.md)), kaucije i privatna sredstva ([`ADR-0024`](ADR-0024-deposit-kaucija.md), [`ADR-0026`](ADR-0026-private-funds-claim.md)); **Faza 3a operativni workflow** Implemented / Accepted (app `9d088df`, [`FAZA3_SALDAKONTI_SPEC.md`](FAZA3_SALDAKONTI_SPEC.md)).

**Do v1.5 (40%):** početno stanje import, G2B ePorezna, puni fiskalizacijski produkcijski cutover, Sprint 4 (amortizacija + fiskalizacija produkcija). Faza 3b (aging widget) ostaje opcionalni backlog.

Detalji faza: [`ROADMAP.md`](ROADMAP.md).

---

## Gdje dalje

| Dokument | Svrha |
|----------|-------|
| [`ARCHITECTURE_PRINCIPLES.md`](ARCHITECTURE_PRINCIPLES.md) | Tehnička pravila |
| [`ARCHITECTURE_GOVERNANCE.md`](ARCHITECTURE_GOVERNANCE.md) | Tko/kada/kako odlučuje |
| [`DOMAIN_MAP.md`](DOMAIN_MAP.md) | Organizacija koda po domenama |
| [`ROADMAP.md`](ROADMAP.md) | Poslovni plan po fazama |
