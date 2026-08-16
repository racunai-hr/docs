# Dedicated Server Migration Runbook (Issue #1)

## Scope
- App: Django ERP (`django` service)
- DB: PostgreSQL/PostGIS (`postgis` service)
- Out of scope: TLS termination and public ingress (handled by external reverse proxy)

## 1) Prepare destination server
1. Sync project files to destination host (`/srv/finestar`):
   - `erp/app`
   - `media`
   - `static`
   - optional: `logs`
2. Ensure `.env` is production-ready:
   - `DEBUG=False`
   - strong `SECRET_KEY`
   - strict `ALLOWED_HOSTS`
   - strict `CSRF_TRUSTED_ORIGINS`
   - `DATABASE_HOST=postgis`

## 2) Migrate only `finestar_erp_db` from old shared DB
On source host:
```bash
pg_dump -h <OLD_DB_HOST> -U <OLD_DB_USER> -d finestar_erp_db -Fc -f finestar_erp_db.dump
```

Copy dump to destination host.

On destination host (restore into dedicated PostGIS):
```bash
docker compose up -d postgis
cat finestar_erp_db.dump | docker compose exec -T postgis pg_restore -U "$DATABASE_USER" -d "$DATABASE_NAME" --clean --if-exists
```

## 3) Bring up app and align schema
```bash
docker compose up -d --build
docker compose exec django python manage.py check --deploy
docker compose exec django python manage.py migrate
docker compose exec django python manage.py collectstatic --noinput
```

## 4) Validation
- Login and key ERP flows (partners, invoices, payments)
- Media upload/download
- Compare row counts for key tables old vs new DB
- Smoke SQL:
```sql
SELECT NOW();
```

## 5) Cutover
1. Update external reverse proxy upstream to dedicated server.
2. Verify HTTP 200/302 on key routes.
3. Monitor logs for 24h:
```bash
docker compose logs -f django postgis
```

## 6) Rollback
- Point reverse proxy upstream back to old server.
- Keep old DB untouched until production verification is complete.
