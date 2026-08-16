# ePorezna Submission Matrix — racunAI ERP

```text
Status: SSOT — inicijalni paket završen (plan v3, 2026-07-10)
Package: 9ec6958 (sadržaj) + 8d1b010 (cross-linkovi)
Last updated: 2026-07-10
Related: FORM_REGISTRY.md, ADR-0009, pdv-extensions-roadmap.md §3
```

Referentni SSOT za **tri kanala predaje** (XSD / G2B / Portal) i razliku između **generiranja XML-a** i **strojne predaje**.

**Trenutno u kodu:** v1 = `SubmissionService` + ručna predaja + `payload_hash` ([ADR-0009](../architecture/ADR-0009-submission-module-v1.md)). **G2B konektor** = v2 backlog.

**Scope ovog dokumenta:** dokumentacija samo. Tax infrastruktura (Sprint 3 freeze) ostaje nepromijenjena.

---

## 1. Uvod — tri kanala

| Kanal | Što je | Uloga u racunAI |
|-------|--------|-----------------|
| **XSD** | Službena XML shema iz `ePorezna_Schemas.zip` | Generator + validator u ERP-u (`render` / `parse` / XSD) |
| **G2B** | SOAP web servis za strojnu predaju (B2G) | Planirani v2 konektor; potpis + upload |
| **Portal** | ePorezna web sučelje (ručni upload) | Trenutni produkcijski put — korisnik potpisuje i predaje izvan ERP-a ili kroz portal |

**Generate XML ≠ Submit:** ERP može generirati i validirati XML bez automatske predaje. `SubmissionService` bilježi **audit** predaje (`payload_hash`, `external_identifier`) neovisno o kanalu.

**REST:** nema službenog REST API-ja Porezne uprave za predaju obrazaca (2026-07). Stupac Transport koristi `REST` samo kao **rezervirano / TBD** za buduće interne API-je racunAI-a, ne kao PU kanal.

---

## 2. Legenda stupaca

### Tier 1 — Master matrica

| Stupac | Vrijednosti | Napomena |
|--------|-------------|----------|
| **XSD** | ✅ / ❌ / TBD | Službena shema u `ePorezna_Schemas.zip` |
| **G2B** | ✅ / Partial / ❌ / TBD | Strojna predaja putem G2B servisa |
| **Portal** | ✅ / ❌ | Ručna predaja kroz ePorezna portal |
| **Status potvrđenosti** | `Confirmed` / `Partial` / `Research` / `Unknown` | Samo `Confirmed` uz citat službenog izvora |
| **Transport** | `SOAP` / `SOAP+MTOM` / `HTTPS Upload` / `REST` / `Portal` / `TBD` | Tehnički prijenos do PU |
| **Authentication** | `FINA poslovni certifikat` / `AKD certifikat` / `NIAS` / `korisničko ime/lozinka` / `Portal login` / `TBD` | |
| **Response** | `UUID` / `XML potvrda` / `PDF potvrda` / `status` / `polling` / `JIR` / `broj zaprimanja` / `TBD` | **Tehnički** odgovor servisa — mapiranje na `SubmissionEvent`, polling, API |
| **Biz Priority** | `P0` / `P1` / `P2` / `P3` | Poslovni prioritet u racunAI roadmapu |
| **racunAI maturity** | `L0`–`L3` | L0 planirano · L1 stub · L2 djelomično · L3 produkcijski |
| **Generate XML** | ✅ / ❌ / Planned | Implementirano u ERP-u |
| **Submit** | `Manual` / `G2B` / `N/A` / `Planned` | Stvarni kanal predaje iz ERP perspektive |

### Tier 2 — Extended matrix (appendix)

| Stupac | Vrijednosti |
|--------|-------------|
| **Source** | `G2B Guide` / `XSD package` / `PU dokumentacija` / `Research` / `ADR` |
| **Schema Version** | npr. `v11.0` (PDV) / `TBD` |
| **Submission Frequency** | `Monthly` / `Quarterly` / `Annual` / `On demand` |
| **Legal Reference** | `Zakon o PDV-u` / `OPZ` / `Pravilnik o PDV-u` / `TBD` |
| **Attachments** | `XML` / `XML+XAdES` / `XML+PDF` / `ZIP` / `TBD` |
| **Confirmation** | `UUID` / `PDF potvrda` / `XML potvrda` / `Status Accepted` / `JIR` / `broj zaprimanja` / `TBD` |
| **Automation Level** | `Manual` / `Assisted` / `Automatic` / `Planned` |

**Response vs Confirmation**

- **Response** — što servis/portala vraća tehnički (za integraciju).
- **Confirmation** — što računovođa/korisnik čuva kao dokaz predaje (PDF, UUID u portalu, …).

**Automation Level** — derivacija iz Generate + Submit:

| Generate XML | Submit | Automation Level |
|--------------|--------|------------------|
| ✅ | Manual (portal) | `Manual` |
| ✅ | Potpis izvan ERP-a, predaja ručno | `Assisted` |
| ✅ | G2B | `Automatic` |
| ostalo | — | `Planned` |

---

## 3. Master matrica (Tier 1)

Sažetak po obrascu. Detalji implementacije u ERP-u: [`FORM_REGISTRY.md`](FORM_REGISTRY.md).

| Obrazac | Svrha (kratko) | Tko predaje | XSD | G2B | Portal | Status | Transport | Authentication | Response | Biz | Maturity | Gen XML | Submit |
|---------|----------------|-------------|-----|-----|--------|--------|-----------|----------------|----------|-----|----------|---------|--------|
| **PDV** | Mjesečna/kvartalna prijava PDV-a | Obveznik PDV-a | ✅ | ✅ | ✅ | Confirmed | SOAP | FINA poslovni certifikat | UUID | P0 | L3 | ✅ | Manual |
| **PDV-ispravak** | Ispravak predanog PDV-a | Obveznik PDV-a | ✅ | ✅ | ✅ | Confirmed | SOAP | FINA poslovni certifikat | UUID | P0 | L3 | ✅ | Manual |
| **PDV-S** | Zbirna prijava isporuka u EU (VIES) | Obveznik PDV-a | ✅ | Partial | ✅ | Partial | SOAP | FINA poslovni certifikat | UUID | P0 | L2 | ✅ | Manual |
| **ZP** | Prijava usluga prema EU (reverse charge kontekst) | Obveznik PDV-a | ✅ | TBD | ✅ | Research | TBD | TBD | TBD | P0 | L0 | Planned | Planned |
| **JOPPD** | Jedinacna prijava poreza i doprinosa | Poslodavac | ✅ | ✅ | ✅ | Confirmed | SOAP | FINA poslovni certifikat | broj zaprimanja | P0 | L0 | Planned | Planned |
| **PDV-K** | Godišnja/kvartalna PDV-K evidencija | Obveznik PDV-a | ✅ | TBD | ✅ | Research | TBD | TBD | TBD | P1 | L0 | Planned | Planned |
| **OSS** | One Stop Shop (e-trgovina EU) | Obveznik | ✅ | TBD | ✅ | Research | TBD | TBD | TBD | P1 | L0 | Planned | Planned |
| **PD** | Porez na dobit | Obveznik poreza na dobit | ✅ | TBD | ✅ | Research | TBD | TBD | TBD | P1 | L0 | Planned | Planned |
| **PPO** | Porez na potrošnju | Obveznik PPO | ✅ | TBD | ✅ | Research | TBD | TBD | TBD | P2 | L0 | Planned | Planned |
| **PZ 42/63** | Prijava za posebne postupke (42, 63) | Obveznik PDV-a | TBD | TBD | ✅ | Unknown | Portal | Portal login | TBD | P2 | L0 | Planned | Planned |
| **PD-IPO** | Porez na dobit — IPO | Obveznik | ✅ | TBD | ✅ | Research | TBD | TBD | TBD | P2 | L0 | Planned | Planned |
| **DONH** | Porez na dohodak (godišnja prijava) | Porezni obveznik | ✅ | TBD | ✅ | Research | TBD | TBD | TBD | P2 | L0 | Planned | Planned |
| **DOH** | Porez na dohodak | Porezni obveznik | ✅ | TBD | ✅ | Research | TBD | TBD | TBD | P2 | L0 | Planned | Planned |
| **INO-DOH** | Inozemni dohodak | Porezni obveznik | ✅ | TBD | ✅ | Research | TBD | TBD | TBD | P2 | L0 | Planned | Planned |
| **EPOM** | Evidencija prometa gotovinom | Obveznik | ✅ | TBD | ✅ | Research | TBD | TBD | TBD | P2 | L0 | Planned | Planned |
| **PPN** | Porez na promet nekretnina | Obveznik | ✅ | TBD | ✅ | Research | TBD | TBD | TBD | P2 | L0 | Planned | Planned |
| **OPZ-STAT-1** | Statistički obrazac (OPZ) | Obveznik | ✅ | TBD | ✅ | Research | TBD | TBD | TBD | P2 | L0 | Planned | Planned |
| **Preknjiženja** | Evidencija preknjiženja | Obveznik PDV-a | TBD | TBD | ✅ | Unknown | Portal | Portal login | TBD | P3 | L0 | Planned | Planned |

**Napomene uz matricu**

- **PDV / PDV-ispravak / JOPPD G2B:** potvrđeno G2B vodičem i shemama u [`ePorezna_Schemas.zip`](https://e-porezna.porezna-uprava.hr/Upute/G2B/ePorezna_Schemas.zip) (vidi [`docs/porezna/upute/2026/README.md`](../porezna/upute/2026/README.md)).
- **PDV-S G2B = Partial:** XSD i korisničke upute postoje; puni G2B flow nije implementiran u racunAI.
- **ZP:** XSD gate Sprint 3 — vidi [`ZP_ARCHITECTURE.md`](ZP_ARCHITECTURE.md), [`SPRINT_3_ZP_XSD_GATE.md`](../architecture/SPRINT_3_ZP_XSD_GATE.md).
- Inicijalno: Transport / Auth / Response = `TBD` ili `Research` osim gdje G2B Guide eksplicitno potvrđuje.

---

## 4. Extended matrix (Tier 2 — appendix)

Isti redoslijed redaka kao §3.

| Obrazac | Source | Schema Version | Frequency | Legal Reference | Attachments | Confirmation | Automation |
|---------|--------|----------------|-----------|-----------------|-------------|--------------|------------|
| PDV | G2B Guide, XSD package | v11.0 | Monthly / Quarterly | Zakon o PDV-u | XML+XAdES | UUID | Manual |
| PDV-ispravak | G2B Guide, XSD package | v11.0 | On demand | Zakon o PDV-u | XML+XAdES | UUID | Manual |
| PDV-S | G2B Guide, XSD package | TBD | Quarterly | Zakon o PDV-u | XML+XAdES | UUID | Manual |
| ZP | XSD package, Research | TBD | Quarterly | Zakon o PDV-u | XML+XAdES | TBD | Planned |
| JOPPD | G2B Guide, XSD package | TBD | Monthly | OPZ | XML+XAdES | broj zaprimanja | Planned |
| PDV-K | XSD package, Research | TBD | Annual / Quarterly | Zakon o PDV-u | XML+XAdES | TBD | Planned |
| OSS | XSD package, Research | TBD | Quarterly | Zakon o PDV-u | XML+XAdES | TBD | Planned |
| PD | XSD package, Research | TBD | Annual | Zakon o porezu na dobit | XML+XAdES | TBD | Planned |
| PPO | XSD package, Research | TBD | Monthly | Pravilnik o PPO | XML+XAdES | TBD | Planned |
| PZ 42/63 | Research | TBD | On demand | Zakon o PDV-u | TBD | TBD | Planned |
| PD-IPO | XSD package, Research | TBD | Annual | Zakon o porezu na dobit | XML+XAdES | TBD | Planned |
| DONH | XSD package, Research | TBD | Annual | Zakon o porezu na dohodak | XML+XAdES | TBD | Planned |
| DOH | XSD package, Research | TBD | Annual | Zakon o porezu na dohodak | XML+XAdES | TBD | Planned |
| INO-DOH | XSD package, Research | TBD | Annual | Zakon o porezu na dohodak | XML+XAdES | TBD | Planned |
| EPOM | XSD package, Research | TBD | On demand | Zakon o fiskalizaciji | XML+XAdES | TBD | Planned |
| PPN | XSD package, Research | TBD | On demand | Zakon o PPN | XML+XAdES | TBD | Planned |
| OPZ-STAT-1 | XSD package, Research | TBD | Annual | OPZ | XML+XAdES | TBD | Planned |
| Preknjiženja | Research | TBD | On demand | Zakon o PDV-u | TBD | TBD | Planned |

---

## 5. Glossary A — Službena terminologija Porezne uprave

Što obrazac **jest** prema propisima i PU dokumentaciji (normativni opis, ne implementacija).

| Obrazac | PU značenje |
|---------|-------------|
| **PDV** | Prijava poreza na dodanu vrijednost za obračunsko razdoblje (mjesečno ili kvartalno). |
| **PDV-ispravak** | Ispravak već predane PDV prijave za isto razdoblje. |
| **PDV-S** | Zbirna prijava isporuka dobara u druge države članice EU (VIES kontekst). |
| **ZP** | Prijava usluga obavljenih poreznim obveznicima u druge države članice EU. |
| **JOPPD** | Jedinacna prijava poreza na dohodak i prireza te doprinosa za mirovinsko i zdravstveno osiguranje. |
| **PDV-K** | Godišnja (ili kvartalna) prijava PDV-a — posebna evidencija uz redovnu PDV prijavu. |
| **OSS** | Prijava PDV-a u okviru posebnog postupka za e-trgovinu (One Stop Shop). |
| **PD** | Prijava poreza na dobit. |
| **PPO** | Prijava poreza na potrošnju. |
| **PZ 42/63** | Prijave u posebnim postupcima oporezivanja (čl. 42 i 63 Zakona o PDV-u). |
| **PD-IPO** | Prijava poreza na dobit za investicijske fondove (IPO). |
| **DONH** | Godišnja prijava poreza na dohodak. |
| **DOH** | Prijava poreza na dohodak (pojedinačni obrazac). |
| **INO-DOH** | Prijava inozemnog dohotka. |
| **EPOM** | Evidencija prometa gotovinom. |
| **PPN** | Porez na promet nekretnina — prijava pri prometu nekretnine. |
| **OPZ-STAT-1** | Statistički obrazac za potrebe Obrasca Poreza (OPZ). |
| **Preknjiženja** | Evidencija preknjiženja PDV-a (ispravak knjiženja u PDV evidenciji). |

---

## 6. Glossary B — racunAI model

Interna arhitektura i mapiranje modela. **§B nije normativ** — opisuje kako racunAI tretira obrazac u kodu i dokumentaciji.

| PU obrazac | racunAI koncept | Return model / izvor | Napomena |
|------------|-----------------|----------------------|----------|
| PDV | Mjesečni PDV pipeline | `VATReturn`, `VATLedgerEntry` | Frozen ADR-0007 |
| PDV-ispravak | Nova verzija + supersede | `VATReturn` + `SubmissionService.supersede()` | Isti payload kanon |
| PDV-S | Inbound EU ledger | `PDVSReturn`, agregat iz VAT knjige | Manual ePorezna |
| ZP | Outbound EU usluge | `ZPReturn` (TBD), box 101/103 u PDV | Sprint 3 — [`ZP_ARCHITECTURE.md`](ZP_ARCHITECTURE.md) |
| JOPPD | Cross-domain builder | `JOPPDReturn` (TBD), `joppd_builder/` | HR `Employee` samo izvor |
| PDV-K | Odvojen obrazac od PDV | `PDVKReturn` (planirano) | vidi [`pdv-extensions-roadmap.md`](../accounting/pdv-extensions-roadmap.md) §2 |
| OSS | Proširenje PDV konteksta | Planirano u PDV 610+ / zaseban modul | e-trgovina |
| Preknjiženja | Finance JE workflow | Nije `tax_forms/` obrazac | Cross-cutting Finance + Tax |

**Razlika PDV-S vs ZP u racunAI:** PDV-S = **inbound** (stjecanje/isporuke u EU, VIES); ZP = **outbound usluge** prema EU s utjecajem na PDV boxove 101/103.

---

## 7. Preporučeni redoslijed modula (P0–P3)

| Prioritet | Obrasci | racunAI faza |
|-----------|---------|--------------|
| **P0** | PDV, PDV-ispravak, PDV-S, ZP, JOPPD | v1.x jezgra — PDV ✅, ZP Sprint 3, JOPPD candidate v1.5 |
| **P1** | PD, OSS, PDV-K | Nakon stabilizacije PDV 610+ |
| **P2** | PPO, PZ 42/63, PD-IPO, DONH, DOH, INO-DOH, EPOM, PPN, OPZ-STAT-1 | Ovisno o djelatnosti klijenta |
| **P3** | Preknjiženja, ostali obrasci | Niži prioritet / cross-cutting |

---

## 8. Istraživački backlog — G2B konektor

Spike (ili serija spikeova) popunjava Tier 1 + Tier 2 stupce za svaki obrazac. Pravila:

1. **Status → `Confirmed`** samo uz citat (PDF stranica, XSD verzija, URL G2B vodiča).
2. Ažurirati stupac **Source** i **Decision Log** (§11).
3. Ne mijenjati `SubmissionService` ni ADR-0009 bez novog ADR-a.

### Research checklist (po obrascu)

| # | Korak | Izlaz |
|---|-------|-------|
| 1 | Locirati mapu u `ePorezna_Schemas.zip` | Schema Version |
| 2 | Pročitati korisničke upute / G2B tehničke preduvjete | Transport, Auth, Attachments |
| 3 | Identificirati response i korisničku potvrdu | Response, Confirmation |
| 4 | Mapirati na `SubmissionEvent` polja | `destination`, `external_identifier`, attachment tip |
| 5 | Procijeniti Automation Level | Manual / Assisted / Automatic |
| 6 | Ažurirati ovu matricu + [`FORM_REGISTRY.md`](FORM_REGISTRY.md) | PR samo docs |

**Prioritet spikea:** P0 obrasci s `G2B = TBD` (ZP), zatim PDV-S (Partial → Confirmed), zatim JOPPD (prije implementacije builder-a).

---

## 9. Submission Capability Roadmap

Horizontalne sposobnosti ERP-a — neovisno o pojedinom obrascu.

| Capability | Status | Napomena |
|------------|--------|----------|
| Generate XML | ✅ | PDV, PDV-ispravak, PDV-S |
| XSD Validation | ✅ | PDV pipeline + PDV-S |
| Import Signed XML | ✅ | `import_signed_vat_return` |
| Submission Audit | ✅ | `SubmissionService` + `payload_hash` (ADR-0009) |
| Manual Portal Submit | ✅ | Admin workflow + UUID unos |
| G2B Submit | Research | v2 — vidi [`pdv-extensions-roadmap.md`](../accounting/pdv-extensions-roadmap.md) §3 |
| Status Polling | Planned | Nakon G2B konektora |
| Download Confirmation | Planned | PDF/XML potvrda s portala |

Cross-link: [`REFERENCE_ARCHITECTURE.md`](../architecture/REFERENCE_ARCHITECTURE.md) (ePorezna connector = manual only).

---

## 10. Future Integrations

Planirane sposobnosti izvan trenutnog v1 scopea (ne commitment datuma):

| Integracija | Opis |
|-------------|------|
| Status check | Polling G2B / portala za status obrade predaje |
| Revoke submission | Povlačenje / storno predaje (ako PU dopušta) |
| Download confirmations | Automatski download PDF/XML potvrde u `SubmissionEvent` attachment |
| Scheduled submit | Celery beat — predaja na rok (uz cert i G2B) |
| Batch submit | Više obrazaca / razdoblja u jednom jobu |

---

## 11. Decision Log

| Datum | Promjena | Izvor |
|-------|----------|-------|
| 2026-07-10 | Dokumentacijski paket završen (matrica + registry + cross-linkovi) | commits 9ec6958, 8d1b010 |
| 2026-07 | Inicijalna matrica (Tier 1 + Tier 2 + glossary) | plan v3 |

---

## 12. Reference

- [`FORM_REGISTRY.md`](FORM_REGISTRY.md) — SSOT obrazaca (sažetak stupaca)
- [`README.md`](README.md) — Tax domena + infrastructure freeze
- [`ADR-0009`](../architecture/ADR-0009-submission-module-v1.md) — Submission v1
- [`ADR-0007`](../architecture/ADR-0007-pdv-module.md) — PDV pipeline
- [`pdv-extensions-roadmap.md`](../accounting/pdv-extensions-roadmap.md) — ePorezna v2 §3
- [`REFERENCE_ARCHITECTURE.md`](../architecture/REFERENCE_ARCHITECTURE.md) — connector status
- [`docs/porezna/upute/2026/README.md`](../porezna/upute/2026/README.md) — lokalni mirror PU uputa i shema
