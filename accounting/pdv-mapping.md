# PDV mapiranje — ERP izvor → VATBox → polje obrasca

Jedini izvor istine za definicije polja je `accounting/services/tax_forms/pdv/boxes.py` (`VAT_BOX_REGISTRY`). Ovaj dokument se mora držati sinkroniziranim s registryjem.

| Kod | Oznaka | Kategorija | PDV polje | Tip | Aktivan | Implementiran | Rezerviran | Pravilo |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 000 | Ukupni promet u razdoblju oporezivanja | other | 000 | scalar | da | ne | ne | — |
| 100 | Isporuke u RH po stopi 0% (osim izvoza) | other | 100 | scalar | da | ne | ne | — |
| 101 | Isporuke dobara unutar EU | other | 101 | scalar | da | da | ne | Invoice EU outbound goods, 0% PDV, I-RA izlazni |
| 102 | Isporuke dobara u treće zemlje (izvoz) | other | 102 | scalar | da | ne | ne | — |
| 103 | Obavljene usluge unutar EU | other | 103 | scalar | da | da | ne | Invoice EU outbound services (datum usluge), 0% PDV, I-RA izlazni |
| 104 | Obavljene usluge u treće zemlje | other | 104 | scalar | da | ne | ne | — |
| 105 | Isporuke NPS u RH | other | 105 | scalar | da | ne | ne | — |
| 106 | Trosarinske naknade | other | 106 | scalar | da | ne | ne | — |
| 107 | Posebni postupci oporezivanja | other | 107 | scalar | da | ne | ne | — |
| 108 | Ostalo oslobođeno | other | 108 | scalar | da | ne | ne | — |
| 109 | Ostalo neoporezivo | other | 109 | scalar | da | ne | ne | — |
| 110 | Ostale isporuke | other | 110 | scalar | da | ne | ne | — |
| 111 | Ukupno oslobođeno i neoporezivo | other | 111 | scalar | da | ne | ne | — |
| 200 | Oporezive isporuke — ukupno | output | 200 | pair | da | ne | ne | — |
| 201 | Oporezive isporuke po stopi 5% | output | 201 | pair | da | da | ne | Invoice item, stopa 5%, I-RA izlazni |
| 202 | Oporezive isporuke po stopi 13% | output | 202 | pair | da | da | ne | Invoice item, stopa 13%, I-RA izlazni |
| 203 | Oporezive isporuke po stopi 25% | output | 203 | pair | da | da | ne | Invoice item, stopa 25%, I-RA izlazni |
| 204 | Prodaja dobara na daljinu unutar EU | output | 204 | pair | da | da | ne | Invoice EU B2C, vat_procedure=eu_distance, I-RA izlazni |
| 205 | Isporuke NPS unutar EU | output | 205 | pair | da | ne | ne | — |
| 206 | Montaža i instaliranje u drugoj državi članici | output | 206 | pair | da | ne | ne | — |
| 207 | Stjecanje dobara unutar EU po stopi 25% (II.7) | output | 207 | pair | da | da | ne | JE D imovina u pripremi 0373 (osnovica), JE K 24022 (PDV obveza RC) |
| 208 | Trostrani poslovi | output | 208 | pair | da | ne | ne | — |
| 209 | Prijenos porezne obveze | output | 209 | pair | da | da | ne | JE K 2401/24011 (građevinski prijenos, tuzemni RC) — II.9 |
| 210 | Primljene usluge unutar EU (B2B) | output | 210 | pair | da | da | ne | JE K 24032 (EU B2B usluge RC) — II.10 |
| 211 | Prodaja rabljenih dobara, umjetničkih djela i kolekcionarskih predmeta | output | 211 | pair | da | ne | ne | — |
| 212 | Putničke agencije (marža) | output | 212 | pair | da | ne | ne | — |
| 213 | Stjecanje unutar EU s ugradnjom u drugoj državi | output | 213 | pair | da | ne | ne | — |
| 214 | Isporuke putem elektroničkog sučelja | output | 214 | pair | da | da | ne | Invoice marketplace/e-interface, vat_procedure=eu_electronic, I-RA izlazni |
| 215 | Isporuke u okviru posebnog postupka OSS | output | 215 | pair | da | da | ne | Invoice OSS e-trgovina EU, vat_procedure=oss, strana stopa PDV, I-RA izlazni |
| 300 | Ukupno pretporez | input | 300 | pair | da | ne | ne | — |
| 301 | Pretporez od isporuka u RH po stopi 5% | input | 301 | pair | da | ne | ne | — |
| 302 | Pretporez od isporuka u RH po stopi 13% | input | 302 | pair | da | ne | ne | — |
| 303 | Pretporez od isporuka u RH po stopi 25% | input | 303 | pair | da | da | ne | Expense s PDV 25% / JE D 1400, U-RA ulazni |
| 304 | Pretporez — uvoz | input | 304 | pair | da | ne | ne | — |
| 305 | Pretporez — stjecanje dobara unutar EU | input | 305 | pair | da | ne | ne | — |
| 306 | Pretporez — primljene usluge unutar EU | input | 306 | pair | da | da | ne | JE D 14032, pretporez na EU B2B usluge — III.10 |
| 307 | Pretporez od stjecanja dobara unutar EU po stopi 25% (III.7) | input | 307 | pair | da | da | ne | JE D 14022, pretporez na EU stjecanje dobara |
| 308 | Pretporez — obveza u drugoj državi članici | input | 308 | pair | da | da | ne | Expense/JE IOSS, vat_procedure=ioss ili konto 14042, U-RA ulazni |
| 309 | Pretporez — prijenos porezne obveze | input | 309 | pair | da | da | ne | JE D 1401/14011, pretporez tuzemni RC (građevina, B2B usluge) — III.9 |
| 310 | Pretporez — manji prijetvor | input | 310 | pair | da | ne | ne | — |
| 311 | Pretporez — posebni postupci | input | 311 | pair | da | ne | ne | — |
| 312 | Pretporez — ostalo | input | 312 | pair | da | ne | ne | — |
| 313 | Pretporez — ne priznat | input | 313 | pair | da | ne | ne | — |
| 314 | Pretporez — ostalo ulazno | input | 314 | pair | da | ne | ne | — |
| 315 | Nepriznati pretporez | input | 315 | scalar | da | ne | ne | — |
| 400 | Obveza PDV-a (za uplatu) / povrat | adjustment | 400 | scalar | da | ne | ne | — |
| 500 | Ispravak pretporeza iz prethodnog razdoblja | adjustment | 500 | scalar | da | ne | ne | — |
| 610 | Ispravak pretporeza — ukupno (VIII.1) | adjustment | 610 | scalar | da | da | ne | VIII.1 zbroj 611+612+613+614+615 — agregat iz ledgera |
| 611 | Ispravak pretporeza — nabava nekretnina (VIII.1.1) | adjustment | 611 | scalar | da | da | ne | JE D nekretnine (05x, 026) — nabava VIII.1.1 |
| 612 | Ispravak pretporeza — ostalo (VIII.1) | adjustment | 612 | scalar | da | da | ne | JE D osobni automobili (032x) — nabava VIII.1.2 |
| 613 | Ispravak pretporeza — PDV komponenta (VIII.1) | adjustment | 613 | scalar | da | da | ne | JE K osobni automobili (032x) — prodaja VIII.1.3 |
| 614 | Ispravak pretporeza — nabava ostale DI (VIII.1.4) | adjustment | 614 | scalar | da | da | ne | JE D ostala DI (030–031x) ili EU Expense 0% bez RC JE — VIII.1.4 |
| 615 | Ispravak pretporeza — prodaja ostale DI (VIII.1.5) | adjustment | 615 | scalar | da | da | ne | JE K ostala DI (030–031x, 05x) — prodaja VIII.1.5 |
| 620 | Ostale isporuke | other | 620 | scalar | ne | ne | da | — |
| 630 | Prijenos dobara u drugu državu članicu | other | 630 | scalar | da | ne | ne | — |
| 640 | Primljene isporuke — prijenos obveze | other | 640 | scalar | da | ne | ne | — |
| 650 | Obavljene isporuke — prijenos obveze | other | 650 | scalar | da | ne | ne | — |
| 660 | Nema prometa u razdoblju | other | 660 | bool | da | ne | ne | — |
| 701 | Marža — rabljena dobra | other | 701 | pair | da | ne | ne | — |
| 702 | Marža — umjetnička djela | other | 702 | pair | da | ne | ne | — |
| 703 | Marža — kolekcionarski predmeti | other | 703 | pair | da | ne | ne | — |
| 704 | Marža — antikviteti | other | 704 | pair | da | ne | ne | — |
