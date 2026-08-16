# Track 0 — Preduvjeti

Checklist administrativnih i infrastrukturnih preduvjeta prije implementacije `fiscal_gateway`.

**Zadnje ažuriranje:** 2026-07-10 (M1.6 UI cutover + prod DIRECT migracija 0008)

Legenda: `[x]` gotovo · `[~]` u tijeku · `[ ]` nije započeto

---

## Dokumentacija Porezne

| # | Zadatak | Status | Napomena |
|---|---------|--------|----------|
| 0.1 | Preuzeti tehničke specifikacije | [x] | [`../../porezna/`](../porezna/) |
| 0.2 | Raspakirati XSD/Schematron u `schemas/` | [x] | `fiscal_gateway/schemas/` (M1.2 Schematron kasnije) |
| 0.3 | Fixture-i za testove u repou | [ ] | `Primjeri eRacuna.zip` itd. |

---

## FINA certifikati

### Korak 00 — PKI preduvjet (prvi Finin certifikat)

| # | Dokument | Status | Napomena |
|---|----------|--------|----------|
| 0.3a | Izvadak sudskog registra (≤ 6 mj.) | [x] | [`../../finestar-sudski-registar-2026-05-27.pdf`](../../finestar-sudski-registar-2026-05-27.pdf) |
| 0.3b | Obavijest DZS — NKD 2025. (prijepis) | [~] | Molba DZS → [journal](../journal/2026-06-12-dzs-nkd-molba.md) |
| 0.3c | OI zastupnika (Toni) + skrbnika (Ante) | [x] | `FineStar_OI_skrbnik_Toni_Supe_*.jpg`, `FineStar_OI_skrbnik_Ante_Vrcan_*.jpg` |

**Blokator za produkcijske obrasce:** čekamo **0.3b (NKD)**. Sudski registar imamo.

### Demo i produkcija

| # | Zadatak | Status | Napomena |
|---|---------|--------|----------|
| 0.4 | Demo certifikat — predaja dokumentacije | [x] | [journal](../journal/2026-06-12-fina-demo-certifikat-ispravak.md) |
| 0.5 | Demo certifikat — preuzimanje `.p12` | [x] | `.certificates/36619131370.F2.2.p12` |
| 0.6 | Produkcijski certifikat (OSPD + uplata) | [~] | Zahtjev + **punomoć** spremni · ugovor + NKD + uplata → [procedura](../procedures/fina-produkcijski-certifikat.md) |

---

## ePorezna / PTS

| # | Zadatak | Status | Napomena |
|---|---------|--------|----------|
| 0.7a | Punomoć ePorezna-JPIP (kôd 478876) — predaja u PU | [x] | [journal](../journal/2026-06-12-eporezna-punomoc-porezna.md) |
| 0.7b | Punomoć ovjerena; opunomoćenik Ante Vrcan na ePorezni | [x] | Fine Star vidljiv na ePorezni |
| 0.8 | Uloga Tester na PTS portalu | [x] | [journal](../journal/2026-06-13-pts-tester.md) |
| 0.8b | PTS svrha: **Za sebe** (Faza 1, ne posrednik) | [x] | [journal](../journal/2026-06-13-pts-za-sebe.md) |
| 0.9 | FiskAplikacija — potvrda AMS adrese | [ ] | Nakon MPS implementacije |

---

## DNS / infra

| # | Zadatak | Status | Napomena |
|---|---------|--------|----------|
| 0.10 | A zapis `mps.racunai.hr` (DNS only) | [x] | `65.108.196.92`, proxied=false — [`scripts/cloudflare_dns_upsert.sh`](../../../scripts/cloudflare_dns_upsert.sh) |
| 0.11 | A zapis `as4-test.racunai.hr` (DNS only) | [x] | `65.108.196.92`, proxied=false |
| 0.11b | A zapis `as4.racunai.hr` (produkcija) | [~] | Planirano nakon FINA prod cert |
| 0.12 | Firewall 443 inbound | [ ] | |
| 0.13 | Traefik routeri za MPS/AS4 | [x] | MPS `priority=200`, `lehttp`; AS4 već postoji |

---

## ISO 27001 (Faza 2 — odgođeno)

| # | Zadatak | Status | Napomena |
|---|---------|--------|----------|
| 0.14 | Gap analiza — 3 ponude | [ ] | Samo za informacijskog posrednika |
| 0.15 | ISMS dokumenti | [~] | Nacrti u `posrednik/` — arhiva Faze 2 |
| 0.16 | Certifikat ISO 27001 | [ ] | **Nije potreban** za PTS „Za sebe“ |

---

## Sljedeća akcija

1. Admin PTS: proširiti svrhu → **Fiskalizacija i eIzvještavanje** (ako još nije na portalu)
2. **Čekati NKD** prijepis od DZS — produkcijski FINA cert ([procedura](../procedures/fina-produkcijski-certifikat.md))
3. Nakon prod cert: DNS `as4.racunai.hr` + firewall 443 (0.12)
4. Prvi live outbound/inbound — [prod-cutover-finestar.md](../procedures/prod-cutover-finestar.md)
5. FiskAplikacija (0.9) nakon MPS/AS4 produkcijskih testova
