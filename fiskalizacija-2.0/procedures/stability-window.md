# 30-day stability window — Fine Star M1.8

**Početak:** 2026-07-02 (conditional go iz [operational-readiness-review.md](operational-readiness-review.md))  
**Kraj:** 2026-08-01  
**Cilj:** 30 dana stabilnog B2B eRačuna prije potpunog zatvaranja M1.8 za produkciju

## Tjedni ritam

| Dan | Aktivnost | Odgovoran |
|-----|-----------|-----------|
| Ponedjeljak | Ops CSV export | Ops |
| Srijeda | Dead-letter pregled (outbox `dead`) | Ops + dev |
| Petak | SLO snapshot (7d rolling) | Ops |
| Po potrebi | PTS regresija (lookup + dry-run CIS) | Dev |

## Export procedure

```bash
# 1. Ops dashboard metrike
# Admin → Integracije → Ops pregled → Export CSV
# URL: /admin/integrations/integrationconfig/ops/export/

# 2. Dead-letter pregled
docker compose exec django python manage.py shell -c "
from integrations.models import IntegrationOutboxMessage
dead = IntegrationOutboxMessage.all_objects.filter(status='dead').order_by('-updated_at')
print('dead count:', dead.count())
for m in dead[:20]:
    print(m.pk, m.correlation_id, m.last_error[:120])
"

# 3. Synthetic health
docker compose exec django pytest fiscal_gateway/tests/test_pts_commands.py -m pts_integration -q
```

## Tjedni log

### Tjedan 1 (2026-07-02 — 2026-07-08)

| Metrika | Vrijednost | Napomena |
|---------|------------|----------|
| Outbound sent / failed (7d) | — | baseline tek započet |
| Dead-letter count | 0 | |
| Inbound audit failures | 0 | |
| MPS `/health` uptime | 100% | ručna provjera |
| CIS S003 | aktivan | Track 0 |
| Kod cutover | ✅ | migracija 0008 + M1.6 admin UI (2026-07-10) |
| Ops export | ☐ | |
| Dead-letter review | ☐ | |

### Tjedan 2 (2026-07-09 — 2026-07-15)

| Metrika | Vrijednost | Napomena |
|---------|------------|----------|
| Outbound sent / failed (7d) | | |
| Dead-letter count | | |
| Ops export | ☐ | |
| Dead-letter review | ☐ | |

### Tjedan 3 (2026-07-16 — 2026-07-22)

| Metrika | Vrijednost | Napomena |
|---------|------------|----------|
| Outbound sent / failed (7d) | | |
| Dead-letter count | | |
| Ops export | ☐ | |
| Dead-letter review | ☐ | |

### Tjedan 4 (2026-07-23 — 2026-07-29)

| Metrika | Vrijednost | Napomena |
|---------|------------|----------|
| Outbound sent / failed (7d) | | |
| Dead-letter count | | |
| Ops export | ☐ | |
| Dead-letter review | ☐ | |

### Završni tjedan (2026-07-30 — 2026-08-01)

| Aktivnost | Status |
|-----------|--------|
| 7d SLO izračun (outbound ≥ 99%) | ☐ |
| Nema neobjašnjenih dead-letter | ☐ |
| ORR potpis updated | ☐ |
| M1.8 milestones `[x]` | ☐ |

## Kriterij zatvaranja M1.8

- [ ] 30 dana bez P1 incidenta na outbound/inbound
- [ ] Outbound SLO ≥ 99% (7d rolling u zadnjem tjednu)
- [ ] Svi dead-letter razriješeni ili dokumentirani
- [ ] PTS matrica Verified bez aktivnih blokera (S003 riješen)
- [ ] ORR potpis: **Go**

## Eskalacija

| Uvjet | Akcija |
|-------|--------|
| dead-letter > 0 | Incident runbook + root cause u 24h |
| outbound failure > 1% / 7d | Error budget eskalacija |
| MPS/Domibus down > 5 min | Infra incident + chaos drill replay |
