# Pravila kontnog plana

Ovo pravilo vrijedi za **cijeli ERP** — knjiženje, osnovna sredstva, banku, izvještaje.

---

## Pravilo (obavezno)

> **Sve kontne oznake (0373, 032001, 029x, 430x, 1000, …) moraju biti konfiguracija tenant-a putem `ChartOfAccounts`. Poslovna logika nikada ne smije ovisiti o šifri konta.**

### Što to znači u praksi

| Dozvoljeno | Zabranjeno |
|------------|------------|
| FK na `ChartOfAccounts` (npr. `preparation_account`, `BankAccount.ledger_account`) | `if account_code == '0373':` u servisima |
| Tenant/postavke koje referenciraju konto po ID-u ili FK-u | Hardkodirane šifre u poslovnoj logici |
| `resolve_account(tenant, code)` samo na granici importa/setupa | Pretpostavka da svaki tenant ima istu šifru |
| Dokumentacija i primjeri s RRiF šiframa | RRiF šifre u runtime grananju |

Isti ERP mora raditi s **bilo kojim kontnim planom** tenant-a.

### Iznimke

- **RRiF predložak** (`RRiF-RP2025.csv`) i **provision** kontnog plana — setup, ne runtime
- **DEFAULT_POSTING_RULES** pri provisioning-u — kopiraju se u `PostingRule` po tenantu; runtime koristi pravila iz baze
- **Testovi** — mogu koristiti poznate šifre za finestar fixture
- **Dokumentacija** — primjeri (T-Cross, uvoz vozila) s tipičnim RRiF šiframa

---

## Povezani dokumenti

- [manual-journal-bank-matching.md](manual-journal-bank-matching.md)
- [fixed-assets-architecture.md](fixed-assets-architecture.md)
- [uvoz-vozila-knjizenje.md](uvoz-vozila-knjizenje.md)
