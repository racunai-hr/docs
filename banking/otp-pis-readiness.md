# OTP PIS — readiness checklist

Payment Initiation Service (PIS) prema Berlin Group / OTP sandbox API.

> Za cjelokupnu arhitekturu i odgovornosti komponenti pogledati [`docs/architecture/banking-v2.md`](../architecture/banking-v2.md).

## Pokretanje provjera

```bash
cd /opt/stacks/racunai.hr/erp

# OTP smoke — jedan entry point za sandbox readiness (preporučeno)
docker compose exec django python manage.py otp_smoke --tenant otp-company-no1

# Pun deploy check (testno plaćanje + knjiženje)
docker compose exec django python manage.py otp_smoke \
  --tenant otp-company-no1 \
  --submit-test-payment \
  --order-id 2

# Debug samo posting
docker compose exec django python manage.py otp_smoke \
  --from=posting --order-id 2 --tenant otp-company-no1

# PIS preduvjeti (cert, connected veza, consent, sync streak)
docker compose exec django python manage.py pis_healthcheck \
  --tenant otp-company-no1

# PIS E2E (bez slanja naloga)
docker compose exec django python manage.py otp_pis_e2e \
  --connection-id 10

# PIS E2E s testnim SEPA nalogom (1 EUR, sandbox)
docker compose exec django python manage.py otp_pis_e2e \
  --tenant otp-company-no1 \
  --submit-test-payment

# PIS knjiženje E2E (authorised nalog + idempotentni drugi publish)
docker compose exec django python manage.py otp_payment_posting_e2e \
  --order-id 2

# Pytest (live sandbox)
docker compose exec django pytest banking/tests/test_otp_pis_e2e.py -m otp_pis_integration -v
```

## Faze razvoja

| Faza | Opis | Status |
|------|------|--------|
| 1 | `POST /v1/payments/sepa-credit-transfers`, `PaymentOrder`, SCA redirect | ✅ MVP |
| 2 | `confirmationCode` callback, PUT authorisation | ✅ osnova |
| 3 | `GET .../status`, mapiranje OTP → ERP statusi | ✅ |
| 4 | Invoice → PaymentOrder → JournalEntry | ✅ (PR2: idempotentno knjiženje) |

### Sandbox knjiženje na `authorised`

OTP sandbox često ne simulira `executed` nakon SCA. Za E2E knjiženje u sandboxu postavite:

```bash
PIS_SANDBOX_ALLOW_POSTING_ON_AUTHORISED=true
```

**Samo sandbox.** Kad je flag `false` (default), `PaymentExecuted` se emitira tek na prijelaz u `executed`.
Produkcija ne koristi ovaj flag — knjiženje ide na stvarno izvršenje plaćanja.

## Compliance checklist

| Komponenta | Sandbox | Produkcija |
|------------|---------|------------|
| AIS consent (`allPsd2`) aktivan | ✅ | ⏳ |
| PIS gate (`sync_success_streak ≥ 3`) | ✅ | ⏳ |
| `PaymentOrder` model | ✅ | ⏳ |
| `POST sepa-credit-transfers` (OTP sandbox; `domestic-payment` nije podržan) | ✅ | ⏳ |
| SCA redirect (`scaRedirect`) | ✅ | ⏳ |
| `confirmationCode` → PUT authorisation | ✅ | ⏳ |
| `GET payment status` | ✅ | ⏳ |
| ERP knjiženje nakon izvršenja | ✅ (sandbox: `PIS_SANDBOX_ALLOW_POSTING_ON_AUTHORISED`, PR2 FK guard) | ⏳ |
| `pis_healthcheck` | ✅ | ⏳ |
| `otp_smoke` | ✅ | ⏳ |
| `otp_pis_e2e` | ✅ | ⏳ |
| `otp_payment_posting_e2e` | ✅ | ⏳ |

## Statusi `PaymentOrder`

| ERP status | OTP (primjer) | Značenje |
|------------|---------------|----------|
| `draft` | — | Lokalni nacrt |
| `submitted` | RCVD | Poslan u banku |
| `sca_required` | PDNG, ACTC | Čeka SCA |
| `authorised` | SCA finalised | PSU potvrdio |
| `accepted` | ACSP, ACCP | Banka prihvatila |
| `executed` | ACSC | Izvršeno |
| `rejected` | RJCT | Odbijeno |
| `failed` | CANC / greška | Neuspješno |

### OTP Sandbox ograničenje

Nakon SCA (`scaStatus: finalised`) sandbox često **ostaje** na `transactionStatus: RCVD`
i ne simulira `ACSP` / `ACSC`. U tom slučaju završni ERP status je **`authorised`**.
Produkcija bi trebala nastaviti lanac do `accepted` → `executed`.

## Arhitektura (paralelno s AIS)

```
AIS                          PIS
├── otp_healthcheck          ├── pis_healthcheck
├── otp_ais_e2e              ├── otp_pis_e2e
├── otp-readiness.md         ├── otp-pis-readiness.md
├── otp_integration          └── otp_pis_integration
└── otp_smoke (umbrella) ────────▶ healthcheck + AIS + PIS + posting summary
```

`otp_smoke` je preporučeni deploy/CI entry point — delegira na service sloj (`run_otp_healthcheck`, `run_ais_e2e_checks`, `run_pis_e2e_checks`, `run_payment_posting_e2e`) bez `call_command()`.

## Reference

- [OTP AIS readiness](otp-readiness.md)
- [OTP Sandbox runbook](otp-sandbox-test-runbook.md)
