# Chaos drill runbook — M1.8 resilience

**Svrha:** Dokazati outbox/retry i operativni oporavak pod kontroliranim kvarovima.  
**Okolina:** staging Docker (`finestar_erp`, `racunai_mps`, `racunai_domibus`)  
**Zadnje izvršenje:** 2026-07-02

## Prije drill-a

1. Ops export: `/admin/integrations/integrationconfig/ops/export/`
2. Zabilježiti stanje outboxa: Admin → Integracijski outbox
3. Koordinacija: obavijestiti operatora (Fine Star)

## Matrica drill-ova

| # | Drill | Injekcija | Očekivano | Dokaz |
|---|-------|-----------|-----------|-------|
| 1 | Domibus down | `docker stop racunai_domibus` | `outbound_failed` + outbox `retrying` → `dead` | audit + outbox admin |
| 2 | CIS timeout | nedostupan endpoint / S003 | `fiscal_failed`, alert | Notification + audit |
| 3 | MPS unreachable | `docker compose stop mps` | lookup fail, jasna greška | CLI/admin audit |
| 4 | Celery worker stop | `docker compose stop celery-worker` | outbox `pending`; oporavak nakon restarta | `process_due_outbox_messages` |

---

## Drill 1 — Domibus down

```bash
docker stop racunai_domibus
# pokreni stvarno AS4 slanje (ne --lookup-only):
docker compose exec django python manage.py pts_eracun_send \
  --xml fiscal_gateway/fixtures/pts_invoice.xml
docker start racunai_domibus
```

**Očekivano:** `As4SendError` → `outbound_failed` u audit logu; outbox `retrying` s backoffom 1m → 5m → 15m → 1h.

**Izvršeno 2026-07-02:** `docker stop racunai_domibus` — lookup-only i dalje prolazi (ne koristi Domibus). Za puni dokaz pokrenuti stvarno slanje ili admin send u test okruženju dok je Domibus down.

**Automatski dokaz (outbox/retry logika):**

```bash
docker compose exec django pytest integrations/tests/test_outbox.py -v
# PASSED: test_transient_failure_enqueues_outbox, test_schedule_next_retry_dead_after_max
```

---

## Drill 2 — CIS greška / certifikat

```bash
docker compose exec django python manage.py fiscal_submit_demo \
  --tenant finestar --cis-env pts \
  --document-number PTS-CHAOS-1 --issue-date 2026-07-02
```

**Očekivano:** `fiscal_failed` u audit logu; alert ako prag prekoračen.

**Izvršeno 2026-07-02:**

| Polje | Vrijednost |
|-------|------------|
| `correlation_id` | `d3a47b7c-5946-4d53-86ff-e7be458dcb82` |
| `cis_request_id` | `1` |
| Greška | `S003` — certifikat nije na pouzdanom popisu |
| Audit korak | `fiscal_failed` (success pipeline do CIS odgovora) |

---

## Drill 3 — MPS unreachable

```bash
docker compose stop mps
docker compose exec django python manage.py pts_mps_ams --action list
docker compose start mps
sleep 3
docker compose exec django curl -s http://racunai_mps:8000/health
```

**Izvršeno 2026-07-02:**

```
CommandError: MPS servis nedostupan (http://racunai_mps:8000/admin/ams/list):
  Failed to resolve 'racunai_mps'
```

Oporavak: `docker compose start mps` → `{"status":"ok"}`.

---

## Drill 4 — Celery worker stop

```bash
docker compose stop celery-worker
# enqueue outbox poruku (admin send s privremenim Domibus down)
docker compose start celery-worker
docker compose exec django python manage.py shell -c \
  "from fiscal_gateway.tasks import process_due_outbox_messages_task; print(process_due_outbox_messages_task())"
```

**Napomena:** `finestar_erp_celery_worker` nije bio pokrenut u stacku tijekom drill-a; `/api/ready/` i dalje prijavljuje broker OK (Redis). Pokrenuti worker prije produkcijskog go/no-go.

---

## Outbox retry dokaz (obavezno za ORR)

Koraci:

1. Simulirati `As4SendError` → outbox enqueue (`integrations/tests/test_outbox.py::test_transient_failure_enqueues_outbox`)
2. Potvrditi backoff: `schedule_next_retry` → 1m, 5m, 15m, 1h (`integrations/retry_policy.py` + `services/outbox.py`)
3. Nakon `max_attempts=5` → status `dead` + audit `outbound_failed` s `dead_letter: true`
4. Admin akcija „Ponovi slanje" → `process_outbox_message_task`

**Rezultat pytest 2026-07-02:** svi outbox testovi PASSED (5/5).

---

## Rollback

Nakon svakog drill-a:

```bash
docker start racunai_domibus racunai_mps
docker compose start celery-worker celery-beat
```

Provjera:

```bash
docker compose exec django pytest fiscal_gateway/tests/test_pts_commands.py -m pts_integration -v
```
