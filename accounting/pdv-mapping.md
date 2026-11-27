# PDV mapiranje — ERP izvor → VATBox → polje obrasca

Jedini izvor istine za definicije polja je `accounting/services/tax_forms/pdv/boxes.py` (`VAT_BOX_REGISTRY`). Ovaj dokument se mora držati sinkroniziranim s registryjem.

| Kod | Oznaka | Kategorija | PDV polje | Tip | Aktivan | Implementiran | Rezerviran | Pravilo |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 000 | Obračun PDV-a u obavljenim transakcijama dobara i usluga — ukupno (I. + II.) | other | 000 | scalar | da | ne | ne | — |
| 100 | I. Transakcije koje ne podliježu oporezivanju, oslobođene i oporezive po stopi 0% — ukupno | other | 100 | scalar | da | ne | ne | — |
| 101 | Isporuke u RH za koje PDV obračunava primatelj (tuzemni prijenos porezne obveze) | other | 101 | scalar | da | ne | ne | — |
| 102 | Isporuke dobara obavljene u drugim državama članicama | other | 102 | scalar | da | ne | ne | — |
| 103 | Isporuke dobara unutar EU | other | 103 | scalar | da | da | ne | Invoice EU outbound goods, 0% PDV, I-RA izlazni |
| 104 | Obavljene usluge unutar EU | other | 104 | scalar | da | da | ne | Invoice EU outbound services (datum usluge), 0% PDV, I-RA izlazni |
| 105 | Obavljene isporuke osobama bez sjedišta u RH | other | 105 | scalar | da | da | ne | Invoice third-country services (datum usluge), 0% PDV, I-RA izlazni |
| 106 | Sastavljanje i postavljanje dobara u drugoj državi članici EU | other | 106 | scalar | da | ne | ne | — |
| 107 | Isporuke novih prijevoznih sredstava u EU | other | 107 | scalar | da | ne | ne | — |
| 108 | Tuzemne isporuke | other | 108 | scalar | da | ne | ne | — |
| 109 | Izvozne isporuke | other | 109 | scalar | da | da | ne | Invoice third-country goods, 0% PDV, I-RA izlazni |
| 110 | Ostala oslobođenja | other | 110 | scalar | da | ne | ne | — |
| 111 | Isporuke po stopi 0% | other | 111 | scalar | da | ne | ne | — |
| 200 | II. Oporezive transakcije — ukupno | output | 200 | pair | da | ne | ne | — |
| 201 | Isporuke dobara i usluga u RH po stopi 5% | output | 201 | pair | da | da | ne | Invoice item, stopa 5%, I-RA izlazni |
| 202 | Isporuke dobara i usluga u RH po stopi 13% | output | 202 | pair | da | da | ne | Invoice item, stopa 13%, I-RA izlazni |
| 203 | Isporuke dobara i usluga u RH po stopi 25% | output | 203 | pair | da | da | ne | Invoice item, stopa 25%, I-RA izlazni |
| 204 | Primljene isporuke u RH za koje PDV obračunava primatelj (tuzemni prijenos porezne obveze) | output | 204 | pair | da | da | ne | JE K 2401/24010/24011 (tuzemni prijenos porezne obveze) — II.4 |
| 205 | Stjecanje dobara unutar EU po stopi 5% | output | 205 | pair | da | da | ne | Porezna činjenica: stjecanje dobara 5%; JE K 24020 (PDV), osnovica s 0373 — II.5 |
| 206 | Stjecanje dobara unutar EU po stopi 13% | output | 206 | pair | da | da | ne | Porezna činjenica: stjecanje dobara 13%; JE K 24021 (PDV), osnovica s 0373 — II.6 |
| 207 | Stjecanje dobara unutar EU po stopi 25% | output | 207 | pair | da | da | ne | Porezna činjenica: stjecanje dobara 25%; JE K 24022 (PDV), osnovica s 0373 — II.7 |
| 208 | Primljene usluge iz EU po stopi 5% | output | 208 | pair | da | da | ne | Porezna činjenica: primljene EU usluge 5%; JE K 24030 — II.8 |
| 209 | Primljene usluge iz EU po stopi 13% | output | 209 | pair | da | da | ne | Porezna činjenica: primljene EU usluge 13%; JE K 24031 — II.9 |
| 210 | Primljene usluge iz EU po stopi 25% | output | 210 | pair | da | da | ne | Porezna činjenica: primljene EU usluge 25%; JE K 24032 — II.10 |
| 211 | Primljene isporuke dobara i usluga od poreznih obveznika bez sjedišta u RH po stopi 0% i 5% | output | 211 | pair | da | ne | ne | — |
| 212 | Primljene isporuke dobara i usluga od poreznih obveznika bez sjedišta u RH po stopi 13% | output | 212 | pair | da | ne | ne | — |
| 213 | Primljene isporuke dobara i usluga od poreznih obveznika bez sjedišta u RH po stopi 25% | output | 213 | pair | da | ne | ne | — |
| 214 | Naknadno oslobođenje izvoza u okviru osobnog putničkog prometa | output | 214 | pair | da | ne | ne | — |
| 215 | Obračunani PDV pri uvozu | output | 215 | pair | da | ne | ne | — |
| 300 | III. Obračunani pretporez — ukupno | input | 300 | pair | da | ne | ne | — |
| 301 | Pretporez od primljenih isporuka u tuzemstvu po stopi 5% | input | 301 | pair | da | ne | ne | — |
| 302 | Pretporez od primljenih isporuka u tuzemstvu po stopi 13% | input | 302 | pair | da | ne | ne | — |
| 303 | Pretporez od primljenih isporuka u tuzemstvu po stopi 25% | input | 303 | pair | da | da | ne | Expense s PDV 25% / JE D 1400, U-RA ulazni |
| 304 | Pretporez od primljenih isporuka u RH za koje PDV obračunava primatelj | input | 304 | pair | da | da | ne | JE D 1401/14010/14011 (tuzemni prijenos porezne obveze) — III.4 |
| 305 | Pretporez od stjecanja dobara unutar EU po stopi 5% | input | 305 | pair | da | da | ne | Porezna činjenica: stjecanje dobara 5%; JE D 14020 — III.5 |
| 306 | Pretporez od stjecanja dobara unutar EU po stopi 13% | input | 306 | pair | da | da | ne | Porezna činjenica: stjecanje dobara 13%; JE D 14021 — III.6 |
| 307 | Pretporez od stjecanja dobara unutar EU po stopi 25% | input | 307 | pair | da | da | ne | Porezna činjenica: stjecanje dobara 25%; JE D 14022 — III.7 |
| 308 | Pretporez od primljenih usluga iz EU po stopi 5% | input | 308 | pair | da | da | ne | Porezna činjenica: primljene EU usluge 5%; JE D 14030 — III.8 |
| 309 | Pretporez od primljenih usluga iz EU po stopi 13% | input | 309 | pair | da | da | ne | Porezna činjenica: primljene EU usluge 13%; JE D 14031 — III.9 |
| 310 | Pretporez od primljenih usluga iz EU po stopi 25% | input | 310 | pair | da | da | ne | Porezna činjenica: primljene EU usluge 25%; JE D 14032 — III.10 |
| 311 | Pretporez od primljenih isporuka od obveznika bez sjedišta u RH po stopi 5% | input | 311 | pair | da | ne | ne | — |
| 312 | Pretporez od primljenih isporuka od obveznika bez sjedišta u RH po stopi 13% | input | 312 | pair | da | ne | ne | — |
| 313 | Pretporez od primljenih isporuka od obveznika bez sjedišta u RH po stopi 25% | input | 313 | pair | da | ne | ne | — |
| 314 | Pretporez pri uvozu | input | 314 | pair | da | ne | ne | — |
| 315 | Ispravci pretporeza | input | 315 | scalar | da | ne | ne | — |
| 400 | IV. Obveza PDV-a u obračunskom razdoblju: za uplatu (II. − III.) ili za povrat (III. − II.) | adjustment | 400 | scalar | da | ne | ne | — |
| 500 | V. Iznos godišnjeg razmjernog odbitka pretporeza (%) | adjustment | 500 | scalar | da | ne | ne | — |
| 610 | Za ispravak pretporeza — ukupno (VIII.1.1–1.5) | adjustment | 610 | scalar | da | da | ne | VIII.1 zbroj 611+612+613+614+615 — agregat iz ledgera |
| 611 | Nabava nekretnina (VIII.1.1) | adjustment | 611 | scalar | da | da | ne | JE D nekretnine (05x, 026) — nabava VIII.1.1 |
| 612 | Nabava osobnih automobila i drugih sredstava za osobni prijevoz (VIII.1.2) | adjustment | 612 | scalar | da | da | ne | JE D osobni automobili (032x) — nabava VIII.1.2 |
| 613 | Prodaja osobnih automobila i drugih sredstava za osobni prijevoz (VIII.1.3) | adjustment | 613 | scalar | da | da | ne | JE K osobni automobili (032x) — prodaja VIII.1.3 |
| 614 | Nabava ostale dugotrajne imovine (VIII.1.4) | adjustment | 614 | scalar | da | da | ne | JE D ostala DI (030–031x) ili EU Expense 0% bez RC JE — VIII.1.4 |
| 615 | Prodaja ostale dugotrajne imovine (VIII.1.5) | adjustment | 615 | scalar | da | da | ne | JE K ostala DI (030–031x, 05x) — prodaja VIII.1.5 |
| 620 | Otuđenje/stjecanje gospodarske cjeline ili pogona | other | 620 | scalar | ne | ne | da | — |
| 630 | Ukupno primljene usluge od poreznih obveznika bez sjedišta u RH (EU + treće zemlje) | other | 630 | scalar | da | ne | ne | — |
| 640 | Ukupno obavljene usluge poreznim obveznicima bez sjedišta u RH (EU + treće zemlje) | other | 640 | scalar | da | ne | ne | — |
| 650 | Primljena dobra iz EU u okviru trostranog posla | other | 650 | scalar | da | ne | ne | — |
| 660 | Postupak oporezivanja prema naplaćenim naknadama | other | 660 | bool | da | ne | ne | — |
| 701 | Donacije hrane po stopi 5% (nabavna vrijednost bez PDV-a) | other | 701 | pair | da | ne | ne | — |
| 702 | Donacije hrane po stopi 13% (nabavna vrijednost bez PDV-a) | other | 702 | pair | da | ne | ne | — |
| 703 | Donacije hrane po stopi 25% (nabavna vrijednost bez PDV-a) | other | 703 | pair | da | ne | ne | — |
| 704 | Donacije hrane — ukupno (nabavna vrijednost bez PDV-a) | other | 704 | pair | da | ne | ne | — |
