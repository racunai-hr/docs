# IZJAVA

**o opsegu usluga koje se pružaju za potrebe provedbe Zakona o fiskalizaciji**

---

Fine Star d.o.o., OIB **36619131370**, matični broj 080885494, sa sjedištem na adresi Bana Josipa Jelačića 58, 22000 Šibenik (u daljnjem tekstu: **Izdavatelj izjave**), putem informacijske platforme **racunAI** (`racunai.hr`), izjavljuje da za potrebe **Zakona o fiskalizaciji** (Narodne novine br. 89/25) pruža sljedeći opseg usluga:

---

## 1. Razmjena eRačuna (AS4)

| Usluga | Opis | Tehnički endpoint |
|--------|------|-------------------|
| **Slanje eRačuna** | Izdavanje i slanje eRačuna u UBL 2.1 formatu (EN 16931, HR CIUS 2025) prema primatelju putem AS4 protokola | Domibus AP, certifikat FINA PKI (CN: FISKAL 2) |
| **Zaprimanje eRačuna** | Zaprimanje eRačuna od drugih sudionika, validacija UBL-a, slanje ApplicationResponse (OK / NOT OK) | `https://as4-test.racunai.hr/EracunAS4/services/msh` (demo) |

---

## 2. Metapodatkovni servis (MPS)

| Usluga | Opis | Tehnički endpoint |
|--------|------|-------------------|
| **MPS** | Objavljivanje metapodataka za identifikaciju primatelja eRačuna (Publisher ID: MPS36619131370) | `https://mps.racunai.hr/EracunMPS/` |

---

## 3. Fiskalizacija eRačuna

| Usluga | Opis | Tehnički endpoint |
|--------|------|-------------------|
| **Fiskalizacija izlaznih eRačuna** | Slanje poruka fiskalizacije na CIS Porezne uprave (demo/PTS/produkcija) | `fiscal_gateway` modul, CIS prema specifikaciji PU |
| **Fiskalizacija ulaznih eRačuna** | Evidentiranje zaprimljenih eRačuna prema specifikaciji | Isto |

---

## 4. eIzvještavanje

| Usluga | Opis | Status |
|--------|------|--------|
| **eIzvještavanje** | Slanje izvještaja (Odbijanje, Ispravak, Naplata) prema specifikaciji eIzvještavanja | **U implementaciji** — planirano u sklopu `fiscal_gateway` |

---

## 5. Popis korisnika usluge

Izdavatelj izjave pruža navedene usluge:

1. **Za vlastite potrebe** — Fine Star d.o.o. (OIB 36619131370) kao porezni obveznik i pristupna točka.
2. **Za potrebe informacijskog posredništva (Faza 2)** — drugim poreznim obveznicima koji sklope ugovor o korištenju platforme racunAI, nakon stjecanja statusa informacijskog posrednika i upisa na Popis PU.

---

## 6. Tehnički identifikatori

| Parametar | Vrijednost |
|-----------|------------|
| OIB posrednika | 36619131370 |
| AP Party ID (AS4) | FISKAL 2 |
| MPS Publisher ID | MPS36619131370 |
| Certifikat | FINA Demo CA 2020 / produkcijski FINA PKI (nakon izdavanja) |
| Infrastruktura | EU (Hetzner, Helsinki) |

---

## 7.

Ovom izjavom Izdavatelj izjave ispunjava obvezu iz **članka 61. stavka 1. točke 4. Zakona o fiskalizaciji**.

---

U Šibeniku, 13. 06. 2026.

&nbsp;

**Fine Star d.o.o.**

Potpis odgovorne osobe: _________________________

**Toni Šupe**, direktor

Pečat tvrtke:
