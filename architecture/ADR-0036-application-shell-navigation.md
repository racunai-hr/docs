# ADR-0036 — Application Shell & Navigation System (NAV-01)

```text
Status: Proposed
Date: 2026-11-28
Type: Domain
Supersedes: —
Related: ADR-0010, ADR-0011, ADR-0020, ADR-0021, ADR-0022, ADR-0032, FAZA3_SALDAKONTI_SPEC.md
```

## Status

**Proposed** — produktne odluke D01–D09 potvrđene su od vlasnika projekta. Tehnički ugovor D10, konačna matrica ruta i persistence rješenje čekaju NAV-01.0 preflight i prihvaćanje. Ovaj ADR ne predstavlja autorizaciju implementacije ili deploya.

## Context

RacunAI ima vertikalnu navigaciju koja s rastom broja poslovnih funkcionalnosti postaje sve manje praktična. Potreban je jedinstven operativni okvir za desktop i mobilne uređaje, s poslovnim kontekstom, personalizacijom i postojećim RBAC-om. Navigacija je korisnički pogled na ERP, ne preslika backend domena. Postojeći Documents-centric workflow, kanonski URL-ovi i poslovna pravila ostaju očuvani.

Repozitoriji: `racunai-hr/app`, `racunai-hr/api`, `racunai-hr/docs`; radna grana `develop`. Provjereni frontend manifest: Next.js 15.5.24, React 19.0.0, TypeScript 5.8.x, Vitest 4.1.x. Lucide nije zasebna ovisnost u pregledanom manifestu; postojeći mehanizam ikona treba inventarizirati.

## Decision

### D01 — Hibridna navigacija (potvrđeno)

Glavni poslovni moduli u horizontalnoj navigaciji, funkcionalnosti aktivnog modula u lijevom Sidebaru. Ova odluka zamjenjuje raniji izbor dvostrukog Sidebara u raspravi, ali ne supersedira postojeći ADR. Zaglavlje sadrži poslovni kontekst i globalne korisničke kontrole. Radna površina ostaje neovisna o navigaciji.

### D02 — Glavni moduli (potvrđeno: ikona iznad naziva)

Predloženi zadani redoslijed, podložan provjeri stvarnih ruta u NAV-01.0:

| ID | Naziv | Lucide ikona |
|---|---|---|
| overview | Pregled | layout-dashboard |
| documents | Dokumenti | files |
| finance | Financije | book-open |
| partners | Partneri | users |
| banking | Bankarstvo | landmark |
| assets | Imovina | building-2 |
| payroll | Plaće | contact |
| tax | Porezi | calculator |
| reports | Izvještaji | chart-no-axes-combined |

Postavke, profil i pomoć izvan glavnih poslovnih modula. Overflow izbornik „Više” za module koji ne stanu. Budući moduli (Prodaja, Nabava, Skladište, CRM, Projekti, Proizvodnja) ne prikazuju se dok nisu implementirani i dostupni. Razdvajanje Dokumenti/Prodaja/Nabava nije autorizirano ovim ADR-om.

### D03 — Proširivi podizbornici (potvrđeno)

Najviše dvije razine u Sidebaru (grupa i stavka), više grupa može biti otvoreno istodobno. Grupa aktivne stranice automatski je otvorena pri navigaciji. Stanje grupa se pamti; mali moduli mogu imati ravne stavke. Sljedeća matrica predstavlja informacijski prijedlog, ne dokaz postojećih ruta:

| Modul | Grupe / kandidati |
|---|---|
| Pregled | Nadzorna ploča, obavijesti, zadaci |
| Dokumenti | Izlazni računi; ulazni računi, troškovi; službeni dokumenti |
| Financije | Knjigovodstvo: glavna knjiga, temeljnice, kontni plan; saldakonti: potraživanja, obveze, otvorene stavke; analitika: mjesta troška, bruto bilanca |
| Partneri | Svi partneri, kupci, dobavljači |
| Bankarstvo | Računi, transakcije, izvodi; usklađivanje, nalozi za plaćanje |
| Imovina | Dugotrajna imovina, vozila; amortizacija |
| Plaće | Zaposlenici; obračuni plaća, JOPPD |
| Porezi | PDV, PDV-S, ZP, U-RA/I-RA; ostali obrasci |
| Izvještaji | Bilanca, RDG; analitika partnera i mjesta troška |

Ne stvarati nove poslovne stranice samo radi matrice. Zadržati Documents-centric saldakonti workflow iz FAZA3_SALDAKONTI_SPEC.md (bez uvođenja zasebne `/saldakonti` rute); za porezne evidencije poštovati postojeća značenja i workflowe. Konačne oznake, rute, grupiranje i zadane stranice verificirati u preflightu.

### D04 — Dizajnerski sustav (potvrđeno)

| Token | Vrijednost |
|---|---|
| Glavna navigacija | `#14263E` |
| Aktivni modul | `#3266AA` |
| Sidebar | `#F3F6FA` |
| Aktivna stavka | `#DCE8F8` |
| Tekst navigacije | `#35465D` |
| Radna površina | `#FFFFFF` |
| Font | Inter; tabularne znamenke za financijske iznose |
| Ikone | Lucide, jednobojne |
| Zaglavlje / moduli | 56 / 72 px (ukupno 128 px) |
| Sidebar | 240 px |
| Ikone modula / stavki | 24 / 18 px |
| Stavka / radius / razmak grupa | 36 / 6 / 16 px |
| Tipografija | naslov stranice 24/600; naslov Sidebara 18/600; grupa 14/600; stavka 14/400–500; modul 12/500 |

Integrirati u postojeće CSS tokene; bez paralelnog dizajnerskog sustava. Lokalno posluživanje Inter fonta je tehnička preporuka. Provjeriti kontrast, fokus i čitljivost u svim stanjima.

### D05 — Sidebar Engine (potvrđeno)

| Stanje | Širina | Ponašanje |
|---|---:|---|
| expanded | 240 px zadano, 200–360 px | nazivi, ikone, proširive grupe, resize ručica |
| compact | 56 px | ikone, tooltip i popup grupa dostupan klikom i tipkovnicom |
| hidden | 0 px | radna površina puna, dostupna kontrola za ponovno otvaranje |

Pamćenje stanja, širine, grupa i posljednje stranice po modulu, uz provjeru pristupa prije vraćanja. Gornja navigacija ostaje vidljiva. Predloženi prečaci Ctrl+B, Ctrl+Shift+B i Esc; provjeriti sukobe s preglednikom i ne presretati unos teksta. Resize mora imati tipkovničku alternativu. Privremena responsive promjena ne prepisuje desktop preferencije.

### D06 — Poslovni kontekst (potvrđeno)

Globalni Tenant Switcher i odabir poslovne godine u zaglavlju. Tvrtke: samo ovlaštene, pretraživanje po nazivu i OIB-u, omiljene i nedavne, logo ili inicijali, pamćenje zadnje tvrtke. Godina se pamti po tvrtki, prikazuje se stvarni status iz backend izvora (ne izmišljati otvorenost/zaključenost). Promjena tvrtke zadržava aktivni modul ako je dostupan, osvježava ovlasti i stvarni API kontekst; inače bira dostupnu sigurnu početnu stranicu. Ne prenositi identifikatore dokumenata iz jednog tenanta u drugi. Nespremljene izmjene zahtijevaju upozorenje prije promjene konteksta.

Godina je poslovni kontekst i zadani vremenski filter gdje ima smisla: Pregled, Dokumenti, Financije, Bankarstvo, Plaće, Porezi i Izvještaji; Imovina djelomično; Partneri i Postavke neovisni. Odabir godine ne mijenja sistemski datum, ne otvara razdoblja i ne zaobilazi postojeća računovodstvena zaključavanja. Ne pretpostaviti da sve godine počinju 1.1. ako stvarni model razdoblja dopušta drukčije. Godišnji filter ne smije proizvoljno uklanjati matične podatke ni otvorene stavke iz prethodnih razdoblja. Konačna semantika po ruti predmet je preflighta.

### D07 — Ovlasti i dostupnost (potvrđeno)

Navigacija koristi postojeći RBAC, tenant kontekst i stvarnu dostupnost funkcionalnosti. Backend ostaje autoritativan za pristup i mutacije. Ne uvoditi nove role ili proširivati postojeće ovlasti. Ranija ilustrativna matrica owner/accountant/viewer nije odobrenje promjene ovlasti. Moduli bez prava pristupa i neimplementirani moduli su skriveni. Implementirani, neaktivirani komercijalni moduli mogu biti u zasebnom katalogu za ovlaštenog administratora; nema implementacije naplate. Katalog nije mehanizam dodjele ovlasti. Prazne grupe i moduli bez dostupnih stavki uklanjaju se iz operativne navigacije.

### D08 — Personalizacija (potvrđeno)

Omiljene stranice na vrhu Sidebara (zvjezdica u navigaciji i na stranici), promjena redoslijeda modula drag & drop uz tipkovničku alternativu, skrivanje dostupnih modula, globalna tražilica navigacije (Ctrl+K gdje dopušteno), posljednjih 10 različitih posjećenih stranica, gustoća 30/36/44 px, sinkronizacija između uređaja, reset na zadane postavke. Moj profil → Personalizacija. Pretraživanje obuhvaća navigaciju, ne poslovne zapise. Skriveni, ali ovlašteni moduli moraju ostati dostupni kroz Više ili pretragu te se mogu ponovno prikazati. Omiljene i nedavne stranice ponovno se filtriraju prema aktualnim ovlastima. Oduzimanje ovlasti nikad se ne zaobilazi preferencijama.

| Opseg | Preferencije |
|---|---|
| Korisnik | redoslijed modula, desktop stanje i širina, gustoća, prikaz ikona/naziva, omiljene i nedavne tvrtke, zadnja tvrtka |
| Korisnik + tenant | omiljene/nedavne stranice, skriveni moduli, poslovna godina |
| Korisnik + tenant + modul | otvorene grupe, posljednja dostupna navigacijska stranica |

Spremati stabilne ID-jeve navigacijskih stavki, ne konkretne URL-ove s poslovnim identifikatorima. Verzija sheme preferencija, validacija vrijednosti, reset i obrada zastarjelih ID-jeva. Koristiti postojeći UserProfile/API ako prikladno; konačni model, API ugovor i konflikt sinkronizacije definirati nakon preflighta. Lokalni cache može ubrzati prikaz, ali ne smije biti izvor ovlasti niti zamjena za trajnu sinkronizaciju.

### D09 — Responsive navigacija (potvrđeno)

| Širina | Glavni moduli | Sidebar |
|---|---|---|
| ≥1440 px | horizontalni | expanded zadano |
| 1024–1439 px | horizontalni + Više | compact zadano |
| 768–1023 px | hamburger | overlay |
| <768 px | hamburger + donja navigacija | skriven / drawer |

Desktop/laptop poštuju spremljene preferencije gdje prostor dopušta. Tablet/mobitel ne prepisuju desktop stanje. Donja mobilna navigacija: Pregled, Dokumenti, Omiljeno, Traži, Više (uz filtriranje ovlasti). Više prikazuje mrežu dostupnih modula i dostupne globalne kontrole. Tvrtka i godina dostupne na svim uređajima. Dodirne mete najmanje 44×44 px, safe area, portret/landscape, zatvaranje drawera nakon navigacije, bez ovisnosti o hoveru. Postojeća PWA funkcionalnost ostaje očuvana ako postoji. Redizajn poslovnih tablica nije dio NAV-01.

### D10 — Tehnička arhitektura (predloženo)

Jedan frontend Navigation Registry s tipiziranim, stabilnim ID-jevima modula, grupa i stavki, oznakama, ikonama, redoslijedom, kanonskim rutama, potrebnim capabilities, ključnim riječima, pravilima poslovne godine i mogućnošću favoriziranja. Navigation Resolver kombinira registry, stvarne backend ovlasti, dostupnost modula i korisničke preferencije. Isti rezultat koriste Top Navigation, Sidebar, Mobile Navigation, Command Palette, Favorites, Recent Pages i Breadcrumbs. Personalizirano skrivanje je odvojeno od autorizacijskog filtriranja.

Predložena organizacija: `src/components/navigation/` (ApplicationShell, TopHeader, ModuleNavigation, Sidebar, SidebarGroup/Item, ResizeHandle, Drawer, MobileNavigation, TenantSwitcher, FiscalYearSwitcher, CommandPalette, Favorites, RecentPages, Breadcrumbs); `src/lib/navigation/` (registry, types, resolver, permissions, preferences, route-matcher, search); hooks prema postojećoj konvenciji. Putanje nisu obvezujuće do audita. Ponovno koristiti postojeće komponente i ukloniti redundantne tek nakon regresijske provjere.

URL je autoritativan za aktivnu stranicu. Podržati postojeće deep links, detaljne rute, browser Back/Forward i postojeće query parametre bez preimenovanja poslovnih ruta. Pri promjeni tenanta spriječiti prikaz starih podataka: tenant-scoped cache, poništavanje ili ignoriranje zastarjelih zahtjeva, osvježavanje konteksta. Ako prava još nisu učitana, ne prikazivati neprovjerene stavke. Preferencije nikad ne mijenjaju autorizaciju. Ako je API ugovor proširen, uskladiti OpenAPI i generirane TypeScript tipove.

## Alternatives considered

| Alternativa | Razlog odabira/odbijanja |
|---|---|
| Klasični vertikalni Sidebar | Odbijen: rast broja modula produžuje izbornik |
| Dvostruki Sidebar | Prvotno odabran, naknadno zamijenjen hibridnim prema odluci vlasnika |
| Hibridni | Odabran: jasna podjela modula i funkcionalnosti |
| Zasebni popisi ruta po uređaju | Odbijen: drift i neujednačena vidljivost |
| Novi RBAC ili novi billing sustav | Izvan opsega; ponovno koristiti postojeće ugovore |

## Consequences

### Prednosti

- Jedinstveno navigacijsko iskustvo, skalabilno s rastom ERP-a.
- Više radne površine za računovodstvene tablice i brži pristup omiljenim stranicama.
- Konzistentan tenant kontekst, dostupnost i korisničke preferencije na uređajima.
- Manje dupliciranja navigacijskog koda.

### Rizici / trade-off

- Zaglavlje visine 128 px smanjuje vertikalni prostor na manjim ekranima.
- Globalna poslovna godina može promijeniti semantiku filtera ako se ne verificira po ruti.
- Sinkronizacija preferencija, promjena tenanta i zastarjeli zahtjevi zahtijevaju provjeru izolacije.
- Tipkovnički prečaci mogu biti rezervirani u preglednicima; potrebne su vidljive alternative.
- Trenutni popis ruta, ovlasti, fontova i API-ja nije u potpunosti auditiran.
- Potreban je pristupačan drag & drop, popup i drawer (fokus, Esc, povrat fokusa, semantički nazivi).

## Follow-up — NAV-01 plan

| Paket | Repo | Opseg / izlaz |
|---|---|---|
| NAV-01.0 | app, api, docs | Read-only preflight: HEAD, grana, working tree; inventar layouta, ruta, RBAC-a, tenant origin, razdoblja, preferencija, CSS, ikona, fontova, testova; gap analiza i konačni prijedlog D10 |
| NAV-01.1 | app | Registry, resolver, route matching; parity svih postojećih ruta |
| NAV-01.2 | app | Application Shell, Navy Blue, moduli i grupe, Lucide/Inter |
| NAV-01.3 | app | Sidebar stanja, resize, popup i prečaci |
| NAV-01.4 | app, api | Tenant switcher i poslovna godina; postojeći auth i kontekst |
| NAV-01.5 | app, api | Preferencije, favorites, drag & drop, tražilica, recent, reset |
| NAV-01.6 | app | Responsive, drawer, donja navigacija, Više, pristupačnost |
| NAV-01.7 | app, api, docs | Integracijski testovi, CI, prijavljeni browser smoke, living docs |

NAV-01.0 ne mijenja kod, bazu, ovisnosti, podatke, git stanje ili deploy; ne ispisuje tajne. Isporučuje stvarne putanje datoteka, kompletnu matricu ruta i ovlasti, plan ponovne uporabe komponenti, prijedlog persistence/API ugovora, testove, rizike i blockere. NAV-01.1 počinje nakon prihvaćanja nalaza. Razvojni paketi ostaju na `develop`; stage/prod deploy nije dio ovog ADR-a.

### Definition of Done

- [ ] Svi postojeći dostupni URL-ovi i Documents-centric poslovni workflowi ostaju funkcionalni; nema izmišljenih ruta.
- [ ] Aktivni modul, grupa i stavka ispravni pri direktnom URL-u, refreshu i Back/Forward.
- [ ] Tenant switch mijenja stvarni API kontekst; nema curenja starih podataka ili poslovnih ID-jeva.
- [ ] Poslovna godina koristi stvarne dostupne podatke i poštuje zaključavanje razdoblja.
- [ ] RBAC, favorites, recent, pretraživanje i katalog ne izlažu nedopuštene stavke niti proširuju ovlasti.
- [ ] Preferencije validirane, trajne, sinkronizirane između uređaja; reset i stale ID fallback provjereni.
- [ ] Tri stanja Sidebara, granice resizea, prečaci i sve četiri responsive kategorije provjereni (uključujući prijelaze 767/768, 1023/1024, 1439/1440 px).
- [ ] Mobilne dodirne mete, safe area, tipkovnički fokus, čitljivost i drawer/popup interakcije provjereni.
- [ ] Postojeći i novi unit/integration testovi prolaze; frontend typecheck i build; OpenAPI contract ako je API mijenjan; server Django testovi samo uz `--keepdb`.
- [ ] CI zelen na točnim SHA-ovima; prijavljeni browser smoke odrađen u postojećem razvojnom okruženju.
- [ ] Relevantni living docs i ADR_INDEX ažurirani; nema promjene produkcije bez zasebne autorizacije.

## Out of scope

Nove poslovne domene ili stranice, promjene računovodstvene logike, PDV pipelinea, Banking v2, SubmissionService, poslovnih ovlasti, komercijalne naplate, redizajn poslovnih tablica i produkcijski deploy. Nema ručnih izmjena PDV XML/payload artefakata ni commita tajni.

## References

- [ADR_INDEX.md](ADR_INDEX.md) — numeracija i predložak
- [ARCHITECTURE_GOVERNANCE.md](ARCHITECTURE_GOVERNANCE.md)
- [ARCHITECTURE_PRINCIPLES.md](ARCHITECTURE_PRINCIPLES.md)
- [ADR-0010-domain-architecture.md](ADR-0010-domain-architecture.md)
- [ADR-0011-architecture-freeze-v1.0.md](ADR-0011-architecture-freeze-v1.0.md)
- [FAZA3_SALDAKONTI_SPEC.md](FAZA3_SALDAKONTI_SPEC.md)
- [ADR-0032-payroll-hr-compliance.md](ADR-0032-payroll-hr-compliance.md)
- [Frontend package.json](https://github.com/racunai-hr/app/blob/develop/package.json)
