# Procedura — FINA Demo aplikacijski certifikat za fiskalizaciju

Izvor: [FINA — izdavanje certifikata](https://www.fina.hr/poslovni-digitalni-certifikati/poslovni-certifikati-za-fiskalizaciju/izdavanje-certifikata-za-fiskalizaciju)

## Korak 00 — PKI preduvjet (prvi Finin certifikat)

- [x] Izvadak sudskog registra — [`../../finestar-sudski-registar-2026-05-27.pdf`](../../finestar-sudski-registar-2026-05-27.pdf)
- [ ] Obavijest DZS NKD 2025. — [journal](../journal/2026-06-12-dzs-nkd-molba.md)
- [x] OI skrbnika/zastupnika

## Korak 01 — Preuzimanje i popunjavanje dokumentacije (Demo)

- [x] Preuzeti Zahtjev za izdavanje Demo certifikata (obrazac D20)
- [x] Popuniti Zahtjev (Fine Star d.o.o., skrbnik **Ante Vrcan**, zastupnik **Toni Šupe**)
- [x] Pripremiti presliku OI skrbnika (prednja + stražnja strana)

Datoteke: [`../../porezna/zahtjevi/`](../../porezna/zahtjevi/)

## Korak 02 — Predaja dokumentacije

**Digitalno slanje:** kopiju potpisane dokumentacije poslati na **certifikati-fiskalizacija@fina.hr**

- [x] Email poslan — vidi [journal](../journal/2026-06-12-fina-demo-certifikat.md)
- [~] FINA traži ispravak — vidi [ispravak](../journal/2026-06-12-fina-demo-certifikat-ispravak.md)

**Alternativa:** predaja u najbližoj poslovnici Fine (nije korišteno).

## Korak 03 — Preuzimanje Demo certifikata

Nakon obrade dokumentacije u Fini:

1. Skrbnik certifikata prima **aktivacijske podatke odvojeno** putem **e-pošte** i **SMS-a**
2. Kombinacijom podataka — **online preuzimanje** na portalu za preuzimanje demo certifikata
3. Postaviti lozinku za `.p12` datoteku (poznata samo skrbniku)

- [ ] Aktivacijski podatci primljeni
- [ ] Certifikat preuzet (`.p12`)
- [ ] Certifikat sigurno pohranjen na serveru

## Parametri certifikata (Fine Star)

| Parametar | Vrijednost |
|-----------|------------|
| CA | Fina Demo CA 2020 |
| Profil | Demo aplikacijski certifikat (NCP) |
| Naziv | FISKAL |
| OIB subjekta | 36619131370 |
| Skrbnik | Ante Vrcan (punomoć Toni Šupe) |
| Zastupnik | Toni Šupe |

## Produkcijski certifikat (kasnije)

**Blokator:** NKD prijepis od DZS-a (korak 00).

Kad NKD stigne → vidi [`fina-produkcijski-certifikat.md`](fina-produkcijski-certifikat.md).

Demo ≠ produkcija. Produkcijski certifikat za go-live nakon PTS DEMO testa; obrasci i uplata opisani u produkcijskoj proceduri.
