# OTP Sandbox — E2E test runbook

Operativni vodič za PSD2 sandbox testiranje s OTP bankom na racunAI platformi.

> Prije izvođenja testova preporučeno pročitati [`docs/architecture/banking-v2.md`](../architecture/banking-v2.md) — dijagram toka (connect → AIS → PIS → posting) i odgovornosti komponenti.

## TPP 166 (Finestar-api-test)

Aktivni sandbox TPP: `PSD-SANDBOX-ID166`. Certifikati ID164 se **ne koriste**.

Portal → generiraj certove → import u IAM → kreiraj Application → preuzmi `ClientCertificate166.zip`:

```bash
cd /opt/stacks/racunai.hr/erp
./scripts/otp_install_sandbox_tpp.sh 166 \
  /opt/stacks/racunai.hr/.temp/ClientCertificate166.zip \
  '<client_secret>'
docker compose restart django celery-worker
```

PSU test korisnici slijede `OTP_CLIENT_ID` (npr. `166.company.no1` za TPP 166).

## Preduvjeti

Za **CAMT.053 bulk import** (Fine Star OTP izvodi) vidi [`camt053-bulk-import-finestar.md`](camt053-bulk-import-finestar.md) — jednokratno kreirati `BankAccount` s IBAN `HR6124070001100204771`.

```bash
cd /opt/stacks/racunai.hr/erp
docker compose exec django python manage.py otp_healthcheck
docker compose exec django python manage.py otp_diagnose
```

### OTP smoke (preporučeni deploy entry point)

Jedan command za read-only provjeru sandbox integracije (healthcheck → AIS → PIS → posting summary):

```bash
# Read-only smoke (default — Payment preskočen)
docker compose exec django python manage.py otp_smoke --tenant otp-company-no1

# Pun deploy check (testno plaćanje + knjiženje)
docker compose exec django python manage.py otp_smoke \
  --tenant otp-company-no1 \
  --submit-test-payment \
  --order-id 2

# Debug samo posting korak
docker compose exec django python manage.py otp_smoke \
  --from=posting --order-id 2 --tenant otp-company-no1
```

Posting očekuje `PaymentOrder` s payment FK; sandbox može trebati `PIS_SANDBOX_ALLOW_POSTING_ON_AUTHORISED=true`.

Redoslijed pojedinačnih commanda: **healthcheck → diagnose → provision → connect (SCA) → accounts → sync → otp_smoke (ili otp_ais_e2e) → reconciliation → celery → PIS**.

## AIS E2E provjera (nakon connecta)

```bash
docker compose exec django python manage.py otp_ais_e2e \
  --tenant otp-company-no1 \
  --run-sync
```

Vidi [`otp-readiness.md`](otp-readiness.md) za puni compliance checklist.

## Brzi reset tijekom razvoja

```bash
docker compose exec django python manage.py provision_otp_test_tenants \
  --tenant otp-company-no1 \
  --reset \
  --admin-user <username>
```

## Provision tenant-a

```bash
# Default: otp-company-no1 + otp-company-no2
docker compose exec django python manage.py provision_otp_test_tenants --admin-user <username>

# Jedan tenant
docker compose exec django python manage.py provision_otp_test_tenants \
  --tenant otp-company-no1 --admin-user <username>

# Svih 8 PSU (nakon E2E na no1)
docker compose exec django python manage.py provision_otp_test_tenants \
  --all --admin-user <username>
```

| slug | OTP PSU (ovisi o `OTP_CLIENT_ID`) |
|------|-----------------------------------|
| `otp-company-no1` | `{tpp}.company.no1` |
| `otp-company-no2` | `{tpp}.company.no2` |
| `otp-co1-emp1` | `{tpp}.co1.emp1` |
| `otp-co1-emp2` | `{tpp}.co1.emp2` |
| `otp-co2-emp1` | `{tpp}.co2.emp1` |
| `otp-co2-emp2` | `{tpp}.co2.emp2` |
| `otp-retail1` | `{tpp}.retail1` |
| `otp-retail2` | `{tpp}.retail2` |

Za TPP 166: `166.company.no1`, itd.

## OAuth + Connect (SCA)

1. Admin → Banking → Bank Connection → odabrati vezu → **Spoji OTP banku**
2. SCA u OTP sandbox portalu (PSU iz tablice)
3. OAuth callback → consent `authorized`, connection `connected`
4. `BankAccount` prelazi `pending_discovery` → `active` nakon GET `/accounts`

## Sync i reconciliation

```bash
# Ručni sync jedne veze (admin action ili Celery)
docker compose exec django python manage.py shell -c "
from banking.tasks import sync_connection_task
from banking.provider_models import BankConnection
c = BankConnection.all_objects.filter(status='connected').first()
if c: print(sync_connection_task(c.pk))
"
```

Celery beat (automatski):
- `banking.sync_all_active_connections` — svakih 15 min
- `banking.check_stale_connections` — svaki sat (>24h bez synca)
- `banking.check_consent_expiry` — dnevno u 07:30

## PIS (Payment Initiation)

PIS je dopušten tek nakon **≥3 uzastopna AIS synca** bez greške (`sync_success_streak ≥ 3`).

```python
connection.is_pis_ready(min_streak=3)  # True → PIS dopušten
```

## Audit predložak

Nakon **svakog** E2E runa kopiraj i popuni:

```yaml
# === Run metadata (reproducibility) ===
date: 2026-07-02T14:30:00Z
environment: sandbox
git_sha: abc1234                    # manage.py otp_audit_metadata ili git rev-parse
migration_version: banking.0008_psd2_lifecycle
otp_portal_version: "..."           # iz IAM metadata ako dostupno
openapi_version: "..."              # iz API discovery
certificate_thumbprint: "sha256:..." # otp_healthcheck output

# === Test context ===
tenant: otp-company-no1
psu: 164.company.no1
phase: connect|accounts|sync|reconciliation|celery|pis
result: pass|fail
duration_seconds: 42

# === Banking IDs ===
connection_id:
consent_id:
authorization_id:
external_account_id:

# === Outcomes ===
transactions_imported:
payments_created:
matches_created:
sync_success_streak:

# === Debug ===
bank_api_call_ids: [123, 124, 125]
error_summary:
notes:
```

### Audit metadata helper

```bash
docker compose exec django python manage.py shell -c "
from banking.utils.audit import get_audit_metadata
import pprint; pprint.pp(get_audit_metadata())
"
```

## Faze E2E testiranja

### Faza 1: `otp-company-no1`
Kompletan flow + audit zapis za svaku fazu.

### Faza 2: `otp-company-no2`
Multi-tenant izolacija — provjeri da tenant A ne vidi podatke tenant B.

### Faza 3: `provision --all`
Preostalih 6 tenant-a nakon uspješne faze 1–2.

### Faza 4: PIS
Jedan tenant, stabilan AIS (`sync_success_streak ≥ 3`), tek tada testirati inicijaciju plaćanja.

## Troubleshooting

| Simptom | Akcija |
|---------|--------|
| healthcheck ✘ P12 | Provjeri `erp/.certificates/otp/client.p12` i `OTP_CERT_PASSWORD` |
| healthcheck ✘ client_secret | Postavi `OTP_CLIENT_SECRET` u `.env`, restart django + celery |
| diagnose ✘ TLS | Cert istekao ili pogrešan sandbox cert |
| Stale connection | Consent istekao ili Celery worker ne radi |
| PIS odbijen | `sync_success_streak < 3` — pričekaj 3 uspješna synca |

## Certifikat

Izvor: `.temp/ClientCertificate164 (1).zip`  
Destinacija: `erp/.certificates/otp/client.p12` (mount: `/run/secrets/otp-cert/client.p12`)

Lozinka u `PfxPassword.txt` unutar ZIP-a → `OTP_CERT_PASSWORD` u `.env`.
