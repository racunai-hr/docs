# Korak 0.5 — Forenzika patcha `610 = 17.911,77` (svibanj 2026)

**Datum izvještaja:** 2026-07-06  
**Razdoblje:** Fine Star d.o.o., PDV 05/2026 (OIB 36619131370)  
**Metoda:** reproducibilni pregled datoteka na disku, git zapisa i agent transkripta u workspaceu.

---

## Forensics Report (kratki izlaz)

| Stavka | Rezultat | Dokaz |
|--------|----------|-------|
| **Timeline** | v1 ERP draft → v2 ERP draft (`610=23.882,35`) → upload na ePoreznu → portal ispravlja 610 → ručni patch XML → deploy u media | vidi § Timeline |
| **Izvor patcha** | **Agent transcript + Django shell** (nije git commit, nije PR, nije repo skripta) | transcript `488f369a-7523-4d0c-8d35-217563e8c011`, linije 194–203 |
| **Obrazloženje** | Doslovni citat korisnika + agentov komentar uz patch | vidi § Obrazloženje |
| **Dokazna snaga** | **D** (autoritativna podloga za ERP) / **C** (portal observed behaviour kao kontekst) | vidi § Razine |
| **Zaključak** | Patch **nije dokazano** imao autoritativnu osnovu za promjenu ERP logike; temelji se na ePorezninom prikazu ispravka i korisničkom zahtjevu za XML korekciju | vidi § Zaključak |

---

## § Timeline

| # | Događaj | `610` | Timestamp / metapodaci | Dokaz |
|---|---------|-------|------------------------|-------|
| 1 | ERP draft **v1** (prije EU knjiga) | `0.00` | `mtime` 2026-07-05 13:21:29 UTC; XML `<ns1:Datum>` 2026-07-05T15:21:29+02:00; Identifikator `497ad087…` | `erp/media/vat_returns/finestar/2026/05/v1/unsigned.xml` |
| 2 | RC temeljnice + regeneracija knjiga; ERP draft **v2** | `23.882,35` | `mtime` 2026-07-05 21:15:55 UTC (payload.json i v2 birth); transcript linija 141–144 | `erp/media/…/v2/payload.json` (`610`: `"23882.35"`); SHA256 `110037b453d6553b…`; transcript `488f369a…` L144 |
| 3 | Kopija v2 u `.temp/pdv_obrazac/` (nepatchirana) | `23.882,35` | Birth 2026-07-05 21:16:15 UTC; transcript linija 143 (`cp …/v2/unsigned.xml`) | transcript `488f369a…` L143 |
| 4 | Upload na ePoreznu; portal prikazuje ispravak | ERP `23.882,35` → portal `17.911,77` | Korisnik screenshot ~2026-07-06 00:20 CEST; transcript L182–184 | transcript `488f369a…` L182–184; agent: „ePorezna ispravlja 610 s 23.882,35 na 17.911,77" |
| 5 | **Patch** XML-a (samo polje 610) | `17.911,77` | XML `<ns1:Datum>` 2026-07-06T00:28:25+02:00; Identifikator `e527c920…`; `mtime` `.temp` 2026-07-05 22:28:37 UTC | transcript `488f369a…` L194–195; `.temp/pdv_obrazac/PDV_36619131370_20260501-20260531.xml` |
| 6 | Deploy patchiranog XML-a u ERP media v2 | `17.911,77` | `mtime` unsigned.xml 2026-07-05 22:35:42 UTC | transcript `488f369a…` L203, L209; `erp/media/…/v2/unsigned.xml` SHA256 `e5406bdd89d31b…` |
| 7 | **Inkonzistencija** nakon deploya | payload.json ≠ unsigned.xml | payload.json `mtime` nepromijenjen (21:15:55); `payload_Qh34ziN.json` 22:35:47 s `610=17911.77` | `payload.json` `610=23882.35`; `payload_Qh34ziN.json` `610=17911.77`; Django storage potvrda 2026-07-06 |

---

## § Izvor patcha (pregledani izvori)

| Izvor | Pregledano | Rezultat |
|-------|------------|----------|
| **Git** `git log -p` na `v2/unsigned.xml`, `.temp/pdv_obrazac/…xml` | ✅ | **Nije pronađeno** — datoteke nisu u git povijesti (media izvan repoa / `.gitignore` `media/temp/`) |
| **Git commit / PR** s promjenom `610` | ✅ | **Nije pronađeno** |
| **Repo skripta / CI** koja mijenja `Podatak610` | ✅ | **Nije pronađeno** |
| **Agent transcript** `488f369a-7523-4d0c-8d35-217563e8c011` | ✅ | **Pronađeno** — patch izvršen Django `manage.py shell` (L195, L203) |
| **Ticket / bilješka računovođe** u repou | ✅ | **Nije pronađeno** |
| **Službena PU uputa** arhivirana u repou kao temelj patcha | ✅ | **Nije pronađeno** |

**Tko je napravio patch (činjenica):** Cursor AI agent u sesiji [PDV 610 EU boxovi](488f369a-7523-4d0c-8d35-217563e8c011), na izričit korisnički zahtjev (transcript L193). XML metapodaci `<ns1:Autor>`: „Toni Šupe" — autor obrasca, ne dokaz izvršitelja patch operacije.

**Kada (činjenica):** 2026-07-06 ~00:28 CEST (XML Datum) / filesystem `mtime` 2026-07-05 22:28–22:35 UTC.

**Mechanizam (činjenica):** Python u Django shellu — `fields['610'] = Decimal('17911.77')`, rerender, zapis u `.temp/pdv_obrazac/`, zatim `default_storage.save(…/v2/unsigned.xml)` (transcript L195, L203).

---

## § Obrazloženje

### Doslovni citat — korisnik (okidač patcha)

Transcript `488f369a…`, **linija 193** (2026-07-06 00:27 CEST):

> ispravi onda xml pa ću ga ponovno učitati

### Doslovni citat — agent (namjera patcha)

Transcript `488f369a…`, **linija 194**:

> Ispravljam XML prema ePorezninom ispravku: 610 = 17.911,77 i usklađeni 611/614/615.

Transcript `488f369a…`, **linija 195** (Django shell, komentar u kodu):

```python
# ePorezna ispravak: samo 610 (611/614/615 ostaju — RC invariant)
fields['610'] = Decimal('17911.77')
```

Transcript `488f369a…`, **linija 184** (kontekst prije patcha — **agentova hipoteza**, ne citat PU upute):

> ePorezna u praksi radi: **610_ispravljeno = 610_xml − 611**.

### Što **nije pronađeno**

- Službena PU definicija da `610` mora biti `17.911,77` za ovo razdoblje — **nije pronađeno** u dostupnim izvorima.
- Dokumentirano mišljenje računovođe da ERP treba slati `17.911,77` — **nije pronađeno** u dostupnim izvorima.
- Git audit trail patcha — **nije pronađeno**.

---

## § Dokazna snaga

| Razina | Primjena na ovaj patch |
|--------|------------------------|
| **A** (službena PU uputa / zakon) | **Nije pronađeno** opravdanje patcha u arhiviranim PU dokumentima u repou |
| **B** (računovođa, dokumentirano) | **Nije pronađeno** |
| **C** (reproducibilno ponašanje ePorezne) | Portal je pri uploadu ERP XML-a (`610=23.882,35`) prikazao ispravak na `17.911,77` (transcript L182–184; korisnik screenshot). **Ne dokazuje** ispravnost ERP promjene — samo observed behaviour portala. |
| **D** (pretpostavka / eksperiment) | Patch je primijenjen prema tom observed behaviouru + agentovoj formuli `610 − 611` **bez** potvrde semantike polja 610–615 |

**Za Decision Gate:** patch razine **D** **ne smije** služiti kao argument za `erp-fix-610-rule`.

---

## § Zaključak

| Pitanje | Odgovor | Referenca |
|---------|---------|-----------|
| **Tko** je napravio patch? | Cursor AI agent (sesija `488f369a…`), na korisnički zahtjev | L193, L194–203 |
| **Kada**? | 2026-07-06 ~00:28 CEST | XML Datum; `mtime` unsigned.xml |
| **Na temelju čega** (činjenica)? | ePoreznin prikaz ispravka pri uploadu + korisnik: „ispravi xml" | L182–184, L193 |
| **Je li patch imao autoritativnu podlogu** (zaključak)? | **Nije dokazano** da je imao autoritativnu podlogu za ERP — nema PU upute (A) ni računovođe (B); temelj je portal (C) + agentova interpretacija (D) | § Dokazna snaga |

### Operativne posljedice (činjenice na disku)

| Datoteka | `610` | Kanonski ERP? |
|----------|-------|---------------|
| `v2/payload.json` | `23.882,35` | **Da** — odgovara `build_pdv_payload` |
| `v2/unsigned.xml` | `17.911,77` | **Ne** — ručno patchirano |
| `.temp/pdv_obrazac/…20260501-20260531.xml` | `17.911,77` | **Ne** — kopija patchiranog XML-a |
| Admin URL `…/v2/unsigned.xml` | `17.911,77` | **Ne** — servira patchirani file |

**Preporuka za upload:** koristiti nepatchirani ERP XML (`610=23.882,35`) ili regenerirati draft iz ERP builda — ne patchirani `unsigned.xml` s admin URL-a.

---

## Pregledani izvori (reproducibilnost)

1. `stat` / SHA256 na:  
   `erp/media/vat_returns/finestar/2026/05/v2/{payload.json,payload_Qh34ziN.json,unsigned.xml}`  
   `.temp/pdv_obrazac/PDV_36619131370_20260501-20260531.xml`
2. `git log -p -- erp/media/vat_returns/finestar/2026/05/v2/unsigned.xml` → prazno
3. `grep` repozitorij za `17911`, `17.911`, `23882`, `patch.*610` → samo patchirane datoteke + ovaj izvještaj
4. Agent transcript: `/root/.cursor/projects/opt-stacks-racunai-hr/agent-transcripts/488f369a-7523-4d0c-8d35-217563e8c011/488f369a-7523-4d0c-8d35-217563e8c011.jsonl` (linije 141–144, 182–184, 193–195, 199, 203, 209)
5. Django storage provjera: `default_storage` `payload.json` → `610=23882.35` (2026-07-06)

---

*Evidence Matrix red „Podrijetlo patcha": status ✅ (forenzika zatvorena 2026-07-06).*
