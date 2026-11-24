# Deploy LifeLink to Render

This repository includes a Render Blueprint (`render.yaml`) with Gunicorn, a health check, a generated secret, and a persistent disk mounted at `/var/data`.

## Deploy
1. Push the contents of this directory to a GitHub repository.
2. In Render, select **New + → Blueprint**, connect the repository, and apply the Blueprint.
3. Confirm the persistent disk is attached. It is required because SQLite data on an ephemeral filesystem can be lost.
4. Wait for deployment, then visit `/healthz` on the service URL; expect `{"status":"ok"}`.
5. Open the Render Shell and run `python -m flask --app app create-admin` once.
6. Sign in through the admin login and test with dummy donor/patient accounts.

## Important plan note
The included Blueprint uses Render's Starter web-service plan because persistent disks are a paid feature. Do not switch to an ephemeral free service while using SQLite if you need durable donor/request data.

## Environment
`render.yaml` sets `RENDER=true`, `LIFELINK_HTTPS=1`, `LIFELINK_DB=/var/data/lifelink.sqlite3`, and generates `LIFELINK_SECRET_KEY`.

## Production caveat
For larger deployments, migrate SQLite to managed PostgreSQL and implement verified backups, restore tests, audit logging, external notification delivery, and a privacy/security review. This app is an academic prototype and is not a replacement for a hospital or licensed blood bank.
