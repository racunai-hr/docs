# 2026-06-13 — PTS tester i demo certifikat

## FINA demo certifikat

- Preuzet: `.certificates/36619131370.F2.2.p12` (FISKAL 2, OIB 36619131370)
- Lozinka: `.certificates/.secret` (nije u gitu)
- Email potvrde: `docs/porezna/Preuzimanje aplikacijskog NRA2 certifikata.eml`

## ePorezna / PTS

- Punomoć 478876 aktivna; Ante Vrcan vidi Fine Star d.o.o. na ePorezni
- PTS tester: https://pts.porezna-uprava.hr/
- Korisničko ime: OIB `11528564544` (Ante Vrcan)
- Admin (ePorezna → PTS): svrha **za sebe** + **eRačun**; tester dodan

## Sljedeći korak (ručno)

1. U PTS adminu proširiti svrhu testiranja → **Fiskalizacija i eIzvještavanje**
2. Testni scenariji → **Fiskalizacija** → Pokreni test
3. `python manage.py fiscal_submit_demo --tenant finestar` s parametrima iz PTS PDF-a
4. Na PTS-u provjeriti status **Uspješan**

**Ne pokretati** Razmjena eRačuna dok nema MPS/AS4 infrastrukture.
