# ADR-0029 implementation plan — OfficialDocument Knjiži (A+B)

Status: **APPROVE / GO** (2026-08-24). Implementation landed as Gate 1 `api`, Gate 2 `app`, Gate 3 living docs.

Decision record: [`ADR-0029-official-document-posting.md`](ADR-0029-official-document-posting.md). This file is the execution checklist, not a second ADR.

## Locked invariants

- `posting_profile` is mutable only before successful A+B. After `[official_document_posted]` JE **and** AP, writes return **409 `profile_locked`**.
- Legacy 0028 `link-journal` (JE without AP) stays a separate state and **does not** lock the profile. Knjiži on that state is **409 `already_linked_manual_posting`**.
- Engine fail-fast: `OfficialDocument` only through `official_document_posted`; `Expense` only through `expense_*`; `Invoice` only through `invoice_*`.
- If a `SubledgerItem` exists, it is SSOT for payment status. JE + bank-match “paid” is only a fallback when AP is absent.
- Pagination refactor stays out of OfficialDocument commits.

## Gates

| Gate | Repo | What landed |
|------|------|-------------|
| 0 | `api` / `app` | ADR-0028 evidencija-only (already on `develop`) |
| 1 | `api` | Profile catalog, `official_document_posted`, Knjiži A+B, ADR-0025 allowlist, read-model SSOT |
| 2 | `app` | Profile picker, Knjiži, badges, bank-close CTA; `next dev` only |
| 3 | `docs` | `DATA_ARCHITECTURE` §4b, ADR-0029 follow-up, `uvoz-vozila-knjizenje.md` |

## Fine Star PPMV acceptance

Document `#1` `UP/I-410-22/26-09/49557`: set `ppmv_vehicle_acquisition`, Knjiži once → exactly one JE (D 0373 / K 2201) and one payable `SubledgerItem`. Second Knjiži is a no-op. Do **not** `link-journal` this document.
