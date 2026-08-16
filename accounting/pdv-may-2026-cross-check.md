# Cross-check — svibanj 2026 (Fine Star, OIB 36619131370)

Audit tablica usklađenosti izvora prije odluke o polju 610. Ažurirati nakon svakog koraka Faze 1.

**Korak 2 (2026-07-06):** Službena semantika polja 610–615 i II.7 dokumentirana u [`pdv-pu-uputa-2026-citations.md`](pdv-pu-uputa-2026-citations.md). EU stjecanje ide u **II.7** (`Podatak207`), ne u 610; 610 = ispravak pretporeza (VIII.1).

**Source Matrix Review (2026-07-06):** ERP build vs službeni element — [`pdv-source-matrix-review-2026.md`](pdv-source-matrix-review-2026.md). Konflikt `610` vs `207`/`307` dokumentiran; `erp-fix-610-rule` rasprava ◐, implementacija LOCKED.

| Izvor | Dobavljač / polje | Osnovica (EUR) | Status |
|---|---|---:|---|
| GL (0373 + RC) | Automobile Hadžić | 23.882,35 | ✅ |
| PDV XML | box **610** (`payload.json`) | 23.882,35 | ✅ |
| PDV-S | **DE229674882** (1 red) | 23.882,35 | ✅ generirano |
| ePorezna | upload PDV-S + PDV | T1: 610→17.911,77; II.7=0; VI.1=17.911,77; 400=−7,98 | ✅ T1 ([Korak 3](pdv-korak3-eporezna-test-2026.md)); T3 ⏳ 1b |

## PDV-S generirani XML (Korak 1a)

| Provjera | Rezultat |
|---|---|
| Datoteka | [`.temp/PDV-S/PDV-S_36619131370_20260501-20260531.xml`](../../.temp/PDV-S/PDV-S_36619131370_20260501-20260531.xml) |
| XSD | ✅ |
| Agregacija | 1 red — DE / 229674882, I1=23.882,35 |
| Potpis | ❌ (unsigned draft — očekivano prije predaje) |

Regeneracija:

```bash
cd /opt/stacks/racunai.hr/erp
docker compose exec -T django python manage.py generate_pdv_s_xml \
  --tenant finestar --year 2026 --month 5 \
  --output /opt/stacks/racunai.hr/.temp/PDV-S/PDV-S_36619131370_20260501-20260531.xml
```

## PDV-S predaja na ePoreznu (Korak 1b)

1. [ePorezna](https://e-porezna.porezna-uprava.hr/) → Fine Star → **Obrazac PDV-S** → **05/2026**
2. **Učitaj XML datoteku** → gornja arhivska datoteka
3. Provjeri: **DE / 229674882 / 23.882,35 / 0,00** (1 agregirani red)
4. **Provjeri** → **Potpiši i pošalji**
5. **Preuzmi XML** s portala nakon predaje

Arhiviranje potpisanog exporta:

```bash
docker compose exec -T django python manage.py archive_pdv_s_submission \
  --tenant finestar --year 2026 --month 5 \
  --xml /path/to/signed/PDV-S_36619131370_20260501-20260531.xml
```

Nakon uspješne predaje ažurirati retk **ePorezna** u tablici iznad na ✅ i zabilježiti datum predaje.
