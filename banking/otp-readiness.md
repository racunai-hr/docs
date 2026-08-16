# OTP PSD2 — readiness checklist

Operativni pregled spremnosti OTP integracije za sandbox i produkciju.
Ažuriraj oznake nakon svake faze razvoja ili E2E provjere.

## Pokretanje E2E provjere

```bash
cd /opt/stacks/racunai.hr/erp

# Statička + live provjera postojeće connected veze
docker compose exec django python manage.py otp_ais_e2e \
  --tenant otp-company-no1

# Uključuje dva uzastopna synca (idempotentnost)
docker compose exec django python manage.py otp_ais_e2e \
  --connection-id 10 \
  --run-sync

# Pytest (live sandbox, ne u default CI)
docker compose exec django pytest banking/tests/test_otp_ais_e2e.py -m otp_integration -v
```

E2E provjera **ne ovisi** o broju sandbox transakcija.

## Compliance checklist

| Komponenta | Sandbox (TPP 166) | Produkcija | Napomena |
|------------|-------------------|------------|----------|
| TPP registracija | ✅ | ⏳ | `PSD-SANDBOX-ID166` |
| QWAC / mTLS certifikat | ✅ | ⏳ | `erp/.certificates/otp/client.p12` |
| QSeal / potpis API poziva | ✅ | ⏳ | `signature.p12` |
| Application (client_id/secret) | ✅ | ⏳ | IAM portal |
| Redirect URI | ✅ | ⏳ | `https://otp-sbx.racunai.hr/oauth/callback/` |
| OAuth Authorization Code | ✅ | ⏳ | `/connect/authorize` → 302 |
| Token exchange (mTLS) | ✅ | ⏳ | `/connect/token` |
| Consent `allPsd2` scope | ✅ | ⏳ | puni AIS (računi + transakcije) |
| SCA redirect + `confirmationCode` | ✅ | ⏳ | PUT authorisation |
| GET `/v1/accounts` | ✅ | ⏳ | pre-step auth, bez `PSU-ID` |
| GET `/v1/accounts/.../transactions` | ✅ | ⏳ | `bookingStatus` obavezan |
| Transaction pagination (`_links.next`) | ✅ | ⏳ | |
| Statement import + deduplikacija | ✅ | ⏳ | `get_or_create` po `external_id` |
| Celery periodični sync | ✅ | ⏳ | 15 min beat |
| E2E automatizirana provjera | ✅ | ⏳ | `otp_ais_e2e` |
| **PIS — domestic payment** | ✅ | ⏳ | OTP sandbox: `sepa-credit-transfers` |
| **PIS — SCA za plaćanje** | ✅ | ⏳ | `confirmationCode` callback |
| **PIS — status plaćanja** | ✅ | ⏳ | `GET .../status` |
| **Mapiranje → ERP knjiženje** | ⏳ | ⏳ | nakon ACSC (faza 4) |
| Produkcijski onboarding | ⏳ | ⏳ | novi TPP, certovi, IAM |

Legenda: ✅ gotovo · ⏳ u planu / nije applicable · ❌ blokirano

## E2E scenarij (`otp_ais_e2e`)

Provjerava:

- ✔ P12 i credentials (`healthcheck_*`)
- ✔ OAuth authorize → HTTP 302
- ✔ `BankConnection.status == connected`
- ✔ Consent authorized + `allPsd2: allAccounts`
- ✔ Accounts count > 0
- ✔ Transactions endpoint HTTP 200
- ✔ `BankStatement` postoji
- ✔ Drugi sync idempotentan (`--run-sync`)
- ✔ `sync_success_streak >= 2`

## Poznata sandbox ograničenja

- Test računi (`ibanNotExisting`, `ibanWithHighVolumes`, …) često vraćaju prazne `booked[]` liste.
- To **nije** greška aplikacije — API vraća 200 s praznim podacima.
- Produkcijski računi s prometom potrebni za test volumena transakcija.

## Reference

- [OTP Sandbox test runbook](otp-sandbox-test-runbook.md)
- OTP User Guide Sandbox v1.2 (`.temp/opt-upute/`)
- Berlin Group NextGenPSD2 v1.3

## Sljedeći razvojni prioriteti

1. **E2E integracijski test** — `otp_ais_e2e` (ovaj dokument)
2. **PIS** — `domestic-payment` → SCA → status → ERP
3. **Automatsko knjiženje** bankovnih izvoda
4. **Produkcijski onboarding** OTP-a
