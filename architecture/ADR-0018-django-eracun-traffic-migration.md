# ADR-0018 — Django eRačun Traffic Migration to Fiscal Gateway

```text
Status: Accepted
Date: 2026-08-17
Type: Integration
Supersedes: —
Related: ADR-0017-fiscal-gateway-model-a.md, FISCAL_GATEWAY_CANONICAL_API.md
```

## Status

**Accepted** — Django više ne smije slati novi eRačun promet izravno na Super. Ovaj ADR zaključava cutover, vlasništvo dokumenta i rollback. Ne mijenja ADR-0017. Prihvaćanje ne odobrava Django implementaciju; prvi outbound slice čeka zaseban impl. plan.

## Context

ADR-0017 je zaključao granicu: `/opt/stacks/racunai.hr/intermediary` je jedini Fiscal Gateway; Super ostaje prvi vanjski posrednik kroz `SuperAdapter`. Kanonski API v1 je specificiran. `SuperAdapter` je implementiran i staging-verificiran.

Django `racunai-api` još uvijek šalje i vuče eRačune kroz `SuperClient` (`super_integration`). `IntegrationConfig` i dalje bira `SUPER` vs `DIRECT` na razini tenanta. Gateway već veže konfiguraciju uz OIB, ne uz tenant dropdown.

Ako Django ostane na `SuperClient` nakon što gateway počne slati isti OIB, nastaju dvostruki pokušaji, dvostruki inbound i nemoguć rollback. Ako se cutover svede na boolean „koristi gateway“, timeout prema `/v1` lako postaje slijepi fallback na Super.

Ovaj ADR uređuje samo **Django promet** prema već prihvaćenom gatewayu. Ne otvara Super sandbox, webhook, ni gašenje `SuperClienta` prije ispunjenih uvjeta.

## Decision

### 1. Verzionirana izlazna ruta po OIB-u

Django ne bira gateway booleanom. Za svaki `taxpayer_oib` postoji **outbound route** s eksplicitnim stanjem:

| Stanje | Značenje |
|--------|----------|
| `LEGACY` | Novi izlazni dokumenti idu kroz `SuperClient` |
| `READY` | Gateway je spreman; promet još nije prebačen |
| `ACTIVE` | Novi izlazni dokumenti idu kroz Fiscal Gateway `/v1` |
| `SUSPENDED` | Novi izlazni dokumenti se ne šalju; čeka operatera |
| `RETIRED` | Izlazni legacy put za taj OIB je ugašen |

Nedostajuća `outbound_route` je fail-safe `LEGACY`.

Obavezna polja rute, najmanje:

- `taxpayer_oib`
- `outbound_provider`
- `routing_generation`
- `activated_at`
- `activated_by`
- razlog promjene
- referenca na gateway binding / provider config

`routing_generation` raste pri svakoj promjeni rute. Dokument pamti generaciju pod kojom je prvi put preuzet za slanje.

### 2. Ulaz i izlaz nisu ista odluka

| Pojam | Što odlučuje | Što ne odlučuje |
|-------|----------------|-----------------|
| `inbound_binding` | Adresa zaprimanja i evidencija FiskAplikacije (ADR-0017 §7) | Kako Django šalje nove izlazne dokumente |
| `outbound_route` | Kako Django šalje **nove** izlazne dokumente | Gdje stižu novi ulazni eRačuni |

`outbound_route = ACTIVE` smije se postaviti samo ako su istodobno ispunjeni:

- provider konfiguriran
- credential dostupan
- capability `outbound_send` postoji (`GET /v1/providers/{provider}/capabilities`)
- readiness je u redu
- OIB je ovlašten za taj način

Evidencija FiskAplikacije **nije** sama po sebi izlazna routing odluka.

### 3. Vlasništvo dokumenta nastaje pri prvom claimu

Vlasnik transporta **nije** trenutak kreiranja nacrta.

| Stanje dokumenta u trenutku cutovera | Transport |
|--------------------------------------|-----------|
| Već pokušan ili poslan kroz `SuperClient` | Ostaje `LEGACY_SUPER_CLIENT` do kraja životnog ciklusa |
| Nacrt bez ijednog provider pokušaja | Smije uzeti gateway rutu nakon aktivacije |
| Nakon prvog dispatch claima | Ruta se više ne mijenja |

`gateway_document_id` **jest** kanonski `document_id` koji izdaje racunAI. Nije drugi identitet.

Na dokumentu se trajno čuva, i to **atomski u istoj transakciji** prvog dispatch claima kad je vlasnik gateway:

- `transport_owner = FISCAL_GATEWAY`
- `routing_generation`
- `bound_provider = super`
- kanonski `document_id`
- send `Idempotency-Key`

Zabranjeno je stanje u kojem dokument ima gateway UUID, a još nema zaključanog vlasnika ili `routing_generation`.

Outbound send ima svoj trajni idempotency ključ. Svaki payment ili druga zasebna naredba ima svoj stabilni command id / idempotency ključ. Svi koriste isti `document_id` i isti `transport_owner`. Query/poll ne stvara novi send ključ niti novi attempt.

```text
document_id
├── send_idempotency_key
├── payment_1_idempotency_key
├── payment_2_idempotency_key
└── status queries
```

### 4. Atomarni routing pod konkurencijom

Prije bilo kojeg mrežnog poziva Django mora:

1. Zaključati dokument za dispatch
2. Pročitati trenutni `outbound_route` za OIB (nedostajuća = `LEGACY`)
3. U **jednoj** transakciji upisati cijeli skup iz §3 (`transport_owner`, `routing_generation`, `bound_provider`, `document_id`, send `Idempotency-Key`)
4. Commitati
5. Tek tada ići na `/v1`
6. Kasniji zadaci čitaju zapis dokumenta, ne trenutni OIB flag

Dva workera ne smiju isti dokument podijeliti između `SuperClienta` i gatewaya.

### 5. Nema legacy fallbacka nakon dvosmislenog `/v1`

Timeout, prekid veze ili 5xx na Django → gateway `/v1` znači: vanjski pokušaj je **moguć**.

Django tada:

- ne šalje isti dokument kroz `SuperClient`
- ne otvara novi `document_id`
- razrješava prethodni **send** pokušaj istim `document_id` i istim send `Idempotency-Key` kroz gateway upit (`GET /v1/outbound/documents/{document_id}`)

Send ključ se ne koristi za payment. Payment razrješava vlastitim command idempotency ključem, istim `document_id` i istim `transport_owner`.

To je ista invarijanta kao ADR-0017 §4 i kanonski ugovor, primijenjena na Django hop.

### 6. Rollback

**Dopušteno**

- prestati aktivirati nove dokumente
- postaviti `outbound_route = SUSPENDED`
- nacrte bez pokušaja ostaviti na čekanju operatera

**Zabranjeno**

- preusmjeriti dokumente u vlasništvu gatewaya natrag na `SuperClient`
- ponovno slati dokument s dvosmislenim `/v1` rezultatom
- mijenjati `transport_owner` nakon claima

Novi dokumenti smiju se vratiti na `LEGACY` samo ako **nema** gateway pokušaja **i nema** dvosmislenog `/v1` zahtjeva.

### 7. Discovery ostaje kanonski ugovor

Ovaj ADR ne izmišlja `GET /v1/providers`. Readiness za `ACTIVE` koristi već specificirani `GET /v1/providers/{provider}/capabilities` (`outbound_send`) plus operativnu potvrdu credentiala. Nema nove discovery rute ni dopune kanonskog ugovora za ovaj korak.

### 8. Životni ciklus po smjeru

| Smjer | Tko vodi dok je dokument legacy | Prvi impl. slice ovog ADR-a |
|-------|----------------------------------|-----------------------------|
| Izlaz: slanje, poll, plaćanje | `SuperClient` | **Da** — samo OIB-ovi čiji je trenutni izlaz `SuperClient` / `outbound_provider=super` |
| Ulaz: pull, UBL, workflow | stari put | Ne — kasniji slice |
| Pravno odbijanje (e-reporting reject) | stari put | Ne — kasniji slice |

Django `DIRECT` / `fine_star_self` ostaje na postojećem putu do kasnijeg slicea. `racunai_direct` za tuđe OIB-ove ostaje izvan opsega.

Dokument ostaje na putu na kojem je prvi put preuzet. Miješanje polla, plaćanja ili rejecta između `SuperClienta` i gatewaya za isti dokument nije dopušteno.

Kad je `transport_owner = FISCAL_GATEWAY`, kasniji poll i payment idu samo na `/v1`, nikad na `SuperClient`. To je pravilo vlasništva, ne dijeljenja send idempotency ključa.

### 9. Gašenje `SuperClienta`

`SuperClient` se smije ukloniti iz API-ja tek kad su **svi** uvjeti ispunjeni:

- nijedan OIB više nije `LEGACY` na izlazu
- nema otvorenih legacy izlaznih životnih ciklusa
- legacy ulaz i pravno odbijanje su migrirani
- nema zakazanih Django Super poll/taskova
- reconciliation je usporedio oba sustava
- credentiali su maknuti iz `SuperTenantConfig`
- monitoring pokazuje definirani tihi period bez poziva starog klijenta

Do tada `SuperClient` ostaje dopušteno prijelazno stanje iz ADR-0017 §3. Ne širi se novim funkcijama.

### 10. Izvan opsega

Ovaj ADR ne implementira:

- Django HTTP klijent, JWT ni admin odabir rute
- modele, migracije baze ni promjenu `IntegrationConfig` tenanta
- gašenje `SuperClienta`
- Super sandbox ni webhook
- public `/v1` na Traefiku
- `racunai_direct` za tuđe OIB-ove
- prvi Django outbound slice (čeka zaseban impl. plan)

## Consequences

### Prednosti

- Cutover je reversibilan na razini novih dokumenata, bez prepisivanja već poslanih
- Dva workera i timeout prema `/v1` ne mogu stvoriti dvostruki vanjski pokušaj
- Ulazna adresa (FiskAplikacija) ostaje odvojena od izlaznog slanja
- ADR-0017 ostaje granica gatewaya; ovaj ADR ostaje granica Django prometa

### Rizici / trade-off

- `READY` i `SUSPENDED` zahtijevaju operativni postupak; boolean cutover bio bi brži i opasniji
- Nacrti mogu čekati dok ruta nije `ACTIVE` ili vraćena na `LEGACY` pod uvjetima §6
- `SuperClient` ostaje u API-ju dok svi smjerovi nisu migrirani
- Prihvaćanje ovog ADR-a ne pokreće Django kod

### Follow-up

- [x] Prihvatiti ovaj ADR prije Django implementacije
- [x] Discovery / readiness — postojeći `GET /v1/providers/{provider}/capabilities`; nema `GET /v1/providers`
- [x] Kanonski ugovor: zaseban `outbound-provider` / `outbound_readiness` — [`FISCAL_GATEWAY_CANONICAL_API.md`](FISCAL_GATEWAY_CANONICAL_API.md) §6.1 / §9.1
- [ ] Implementirati `outbound_route` + atomarni stamp iz §3/§4 + claim
- [ ] Prvi slice: novi izlazni dokumenti kroz `/v1` za postojeći Django `SUPER` put
- [ ] Kasniji sliceovi: Django `DIRECT` / `fine_star_self`, ulaz i pravno odbijanje
- [ ] Ukloniti `SuperClient` tek po §9
