# Capability Map — racunAI ERP

Product view: što kupac vidi. Developer view: [`DOMAIN_MAP.md`](DOMAIN_MAP.md).

---

## Capabilities

| Capability (kupac vidi) | Domena(e) | Maturity | AI capability | Faza |
|-------------------------|-----------|----------|---------------|------|
| **Accounting** (knjigovodstvo) | Finance | L3 | auto-kontiranje (plan) | 1 |
| **Tax** (porezi, PDV, obrasci) | Tax | L3 | auto-PDV pravila (plan) | 1 | [`FORM_REGISTRY.md`](../tax/FORM_REGISTRY.md) |
| **Sales** (prodaja, računi) | Sales | L2 | prepoznavanje partnera (plan) | 1 |
| **Purchasing** (nabava, troškovi) | Purchasing | L2 | OCR ulaznih računa (plan) | 1–2 |
| **Banking** (banke, plaćanja) | Banking | L3 | auto-zatvaranje izvoda (plan) | 1 |
| **Integration** (eRačun, fiskalizacija) | Integration | L3 | — | 1 |
| **Reporting** (izvještaji, Bilanca/RDG, Documents read model) | Reporting | L2 | AI asistent za računovođe (plan) | 1 | [`ADR-0020`](ADR-0020-document-read-model.md) |
| **Assets** (dugotrajna imovina) | Assets | L3 | — | 1 |
| **Inventory** (zalihe) | Inventory | L0 | — | 2 |
| **CRM** (kupci, prodaja) | CRM | L0 | — | 2 |
| **Workflow** (odobravanja) | Workflow | L1 | — | 2 |
| **DMS** (dokumenti) | DMS | L0 | — | 2 |
| **HR** (zaposlenici, plaće) | HR | L0 | — | 3 |
| **Platform** (korisnici, postavke) | Platform + Core | L2–L3 | — | 1 |
| **Compliance** (zakonska usklađenost) | cross-cutting | L2 | — | 1 |

---

## AI Capabilities (Faza 4, paralelno)

AI nije zasebna domena — implementira se po domenama (ADR-0010):

| AI capability | Domena | Ciljna putanja | Status |
|---------------|--------|----------------|--------|
| Auto-PDV pravila, klasifikacija | Tax | `domains/tax/ai/` | stub |
| Auto-kontiranje, provjera knjiženja | Finance | `domains/finance/ai/` | stub |
| OCR ulaznih računa | Purchasing | `domains/purchasing/ai/` | v1 review draft |
| Prepoznavanje partnera | Sales | `domains/sales/ai/` | stub |
| AI asistent za računovođe | Reporting | `domains/reporting/ai/` | stub |

**Pravilo:** AI pomaže korisniku, ali ne donosi nepovratne odluke bez pregleda (vidi [`NORTH_STAR.md`](NORTH_STAR.md)).

---

## Mapiranje Capability → GitHub Milestone

| Milestone | Capability | Faza |
|-----------|------------|------|
| `Platform` | Platform | 1 |
| `Core` | Platform | 1 |
| `Finance` | Accounting | 1 |
| `Tax` | Tax | 1 |
| `Sales` | Sales | 1 |
| `Purchasing` | Purchasing | 1 |
| `Integration` | Integration | 1 |
| `Reporting` | Reporting | 1 |
| `Assets` | Assets | 1 |
| `Inventory` | Inventory | 2 |
| `CRM` | CRM | 2 |
| `Workflow` | Workflow | 2 |
| `DMS` | DMS | 2 |
| `HR` | HR | 3 |
| `AI: Tax` | Tax AI | 4 |
| `AI: Finance` | Accounting AI | 4 |
| `AI: Purchasing` | Purchasing AI | 4 |
| `AI: Reporting` | Reporting AI | 4 |
