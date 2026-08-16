# PTS — M1.8 runbook (Fine Star)

Operativni koraci za PTS certifikaciju (čl. 63.). Svaki korak mora ostaviti audit trag u `IntegrationAuditLog` — vidi [interoperability-matrix.md](interoperability-matrix.md).

## Preduvjeti

- [x] Demo certifikat `.certificates/36619131370.F2.2.p12`
- [x] PTS tester login (OIB `11528564544`)
- [ ] Admin proširio svrhu testiranja → **Fiskalizacija i eIzvještavanje**
- [x] `IntegrationConfig` DIRECT + CIS za test okruženje (migracija `0006`)
- [x] Admin UI cutover — Računi/Troškovi koriste `IntegrationManager` + audit timeline (M1.6)

---

## Korak A — CIS fiskalizacija (PTS 8511)

Upute: [`../../porezna/zahtjevi/PTS_Fiskaliziraj_Izlazni_eRacun.pdf`](../../porezna/zahtjevi/PTS_Fiskaliziraj_Izlazni_eRacun.pdf)

1. Na PTS-u: **POKRENI TEST** za scenarij „Fiskaliziraj Izlazni eRačun“
2. Pošalji poruku (oznaka **`I`**, izdavatelj Fine Star):

```bash
docker compose exec django python manage.py fiscal_submit_demo \
  --tenant finestar \
  --cis-env pts \
  --invoice-kind I \
  --issuer-name "Fine Star d.o.o." \
  --issuer-oib 36619131370 \
  --operator-oib 11528564544 \
  --document-number "PTS-IZL-1" \
  --issue-date "2026-06-12"
```

3. **Provjera:**
   - `FiscalSubmissionLog.jir` + `cis_request_id`
   - Audit: `fiscalized` (success) s `correlation_id`
   - Admin → Integracijski audit logovi → timeline

4. Očekivani CIS odgovor: `<fis:prihvacenZahtjev>true</fis:prihvacenZahtjev>`

**Napomena:** PTS endpoint `https://cis.porezna-uprava.hr:8511/FiskalizacijaService` (`--cis-env pts`). Demo okolina: `--cis-env demo` (port 8509).

---

## Korak B — MPS/AMS registracija

```bash
docker compose exec django python manage.py pts_mps_ams --action list
docker compose exec django python manage.py pts_mps_ams --action create --oib 36619131370
```

**Provjera:** AMS lista sadrži OIB Fine Star; zapis u interoperability matrici.

---

## Korak C — Slanje eRačuna (lookup + AS4)

```bash
docker compose exec django python manage.py pts_eracun_lookup \
  --xml fiscal_gateway/fixtures/pts_invoice.xml

docker compose exec django python manage.py pts_eracun_send \
  --xml fiscal_gateway/fixtures/pts_invoice.xml --lookup-only

# Stvarno slanje kad Domibus/PTS spremni:
docker compose exec django python manage.py pts_eracun_send \
  --xml fiscal_gateway/fixtures/pts_invoice.xml
```

**Provjera:**
- Lookup output: `lookup OK`, AS4 endpoint, PT primatelja
- Send: `Domibus submit OK`, `message_id` u Domibus adminu
- Audit: `signed` → `outbound_sent`
- Status polling: `poll_outbound_statuses` (Celery ili ručno)

---

## Korak D — Zaprimanje eRačuna

1. Inbound push na `As4InboundPushView` (`/api/fiscal/as4/inbound/`)
2. **Provjera:**
   - `Expense` kreiran
   - `As4DocumentLink` (inbound)
   - Automatski `ApplicationResponse`

---

## Korak E — Admin send kroz puni pipeline

1. `IntegrationConfig`: DIRECT + CIS za test tenant (`environment=test`)
2. Admin → Računi → Pošalji eRačun
3. **Očekivani audit koraci:**

```
document_built → semantic_ok → xsd_ok → schematron_ok → signed → outbound_sent → fiscalized
```

4. Ops pregled: Admin → Integracije → `/admin/integrations/integrationconfig/ops/`

---

## Audit trag (obavezno za matricu)

Za svaki Verified red u [interoperability-matrix.md](interoperability-matrix.md) zapisati:

| Polje | Primjer |
|-------|---------|
| `correlation_id` | `a1b2c3d4-...` |
| Vanjski ID | `jir`, `message_id`, `cis_request_id` |
| Fixture | `fiscal_gateway/fixtures/pts_invoice.xml` |
| Datum | 2026-07-02 |
| Tenant | finestar |
| Operator | 11528564544 |

---

## Napomene

- Greška CIS `S003` (certifikat nije na pouzdanom popisu) može privremeno biti aktivna dok FINA demo CA nije propagiran u CIS.
- UBL potpis: automatski za DIRECT provider; verifikacija: `ubl.signing.verify.verify_invoice_signature`
- Schematron: puni PU ZIP u `ubl/schematron/HRUBLSchematron*.zip`

Vidi [journal/2026-06-13-pts-tester.md](../journal/2026-06-13-pts-tester.md).
