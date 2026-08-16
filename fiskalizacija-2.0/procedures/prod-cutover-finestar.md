# Produkcijski cutover — Fine Star (M1.8 B6)

**Tenant:** `finestar` · **OIB:** `36619131370`  
**Zadnje ažuriranje:** 2026-07-10

## Preduvjeti

- [x] M1.6 UI cutover — admin koristi `IntegrationManager` + DIRECT default
- [x] Migracija `0008_fiscal_prod_cutover` — SUPER prod deaktiviran, DIRECT prod aktiviran
- [~] FINA produkcijski `.p12` — čeka NKD prijepis ([fina-produkcijski-certifikat.md](fina-produkcijski-certifikat.md))
- [~] CIS S003 riješen — demo cert na trusted listi
- [x] DNS `mps.racunai.hr`, `as4-test.racunai.hr`
- [x] 30-day stability window pokrenut ([stability-window.md](stability-window.md))

## Koraci cutovera

### 1. Certifikati

```bash
# Produkcijski cert nakon NKD + OSPD upload
# Spremiti u /run/secrets/fiscal-cert/ (ili FISCAL_CERT_P12_PATH)
# Admin → Fiskalne konfiguracije → cis_env = produkcija
```

### 2. IntegrationConfig

Migracija `integrations/0008_fiscal_prod_cutover.py` automatski:

- deaktivira `IntegrationConfig(provider=super, environment=production)`
- aktivira `IntegrationConfig(provider=direct, environment=production)`
- postavlja `FiscalTenantConfig.cis_env = prod` (ako je bio demo)

Ručna provjera:

```bash
docker compose exec -T django python manage.py shell -c "
from integrations.repository import IntegrationRepository
from tenants.models import Tenant
t = Tenant.objects.get(slug='finestar')
cfg = IntegrationRepository.resolve_eracun_config(t, 'production')
print(cfg.provider if cfg else 'None')
"
# Očekivano: direct
```

### 3. Rollback (hitno)

Postaviti u `.env`:

```
USE_SUPER_ERACUN_FALLBACK=True
```

Zatim u Admin → IntegrationConfig:

- deaktivirati DIRECT production
- aktivirati SUPER production (ako `SuperTenantConfig` još postoji)

### 4. Prvi live outbound

1. Admin → Računi → odabrati račun → **Pošalji eRačun**
2. Provjera audit timeline: `document_built → … → signed → outbound_sent → fiscalized`
3. Ops dashboard: `/admin/integrations/integrationconfig/ops/`

### 5. Prvi live inbound

1. Partner šalje UBL na `POST /api/fiscal/as4/inbound/` (Domibus push)
2. Admin → Troškovi → provjera AS4 UBL linka + audit timeline
3. Akcija **Odobri ulazni eRačun i knjiži**

### 6. Stability window

Pokrenuti tjedni ritam iz [stability-window.md](stability-window.md):

- Ops CSV export (ponedjeljak)
- Dead-letter review (srijeda)
- SLO snapshot (petak)

**Kraj prozora:** 2026-08-01

## MASTER_CHECKLIST ažuriranje

Nakon uspješnog cutovera i stability windowa:

| # | Modul | Cilj |
|---|-------|------|
| 28 | AS4 (Domibus) | `[x]` produkcijski inbound+outbound |
| 30 | Fiskalizacija 2.0 / CIS | `[x]` CIS prod JIR |
| 31 | SUPER eRačun | `[x]` uklonjen / deprecated |
| 48 | Fiskalizacija compliance | `[x]` |
| 49 | eRačun legal compliance | `[x]` produkcijski cutover |

## Reference

- PTS runbook: [pts-fiskalizacija-test.md](pts-fiskalizacija-test.md)
- Interoperabilnost: [interoperability-matrix.md](interoperability-matrix.md)
- ORR: [operational-readiness-review.md](operational-readiness-review.md)
