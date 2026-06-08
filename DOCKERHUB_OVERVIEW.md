# CA Manager Demo

CA Manager Demo is a self-contained Docker image for evaluating CA Manager in lab, training, and non-critical demo environments.

The image runs CA Manager in production mode with a bundled MySQL 8 database. On first startup, it initializes the schema, creates a demo administrator, and preloads an Acme Corp lab PKI inventory so the UI is ready to explore immediately.

Public demo files and documentation: [api-monster/CA-Manager-Demo](https://github.com/api-monster/CA-Manager-Demo)

## What Is Included

- CA Manager web UI and API in one container
- Bundled MySQL 8 database
- First-run schema initialization
- Pre-created demo administrator
- Optional Acme Corp demo PKI data
- Persistent database volume
- UI and API published on port `8585`

## Quick Start With Docker

```bash
docker run -d \
  --name ca-manager-demo \
  -p 8585:8585 \
  -v ca-manager-demo-data:/var/lib/mysql \
  -e SESSION_SECRET='change-this-demo-session-secret-at-least-32-characters' \
  -e KEY_ENCRYPTION_SECRET='change-this-demo-key-secret-at-least-32-characters' \
  -e MYSQL_ROOT_PASSWORD='change-this-demo-root-password' \
  -e MYSQL_PASSWORD='change-this-demo-db-password' \
  -e SESSION_COOKIE_SECURE=false \
  apimonster/ca-manager-demo:latest
```

Open:

```text
http://localhost:8585
```

## Default Demo Login

```text
Email: admin@acme.com
Password: ChangeMe!8585
```

Override these on first run with `CA_MANAGER_DEMO_ADMIN_EMAIL`, `CA_MANAGER_DEMO_ADMIN_PASSWORD`, and `CA_MANAGER_DEMO_ADMIN_NAME`.

For plain `http://localhost:8585` demos, keep `SESSION_COOKIE_SECURE=false` so the browser can store the login session cookie. Use `true` only when serving the demo through HTTPS.

## Docker Compose

```yaml
services:
  ca-manager-demo:
    image: apimonster/ca-manager-demo:latest
    container_name: ca-manager-demo
    restart: unless-stopped
    ports:
      - "8585:8585"
    environment:
      SESSION_SECRET: change-this-demo-session-secret-at-least-32-characters
      KEY_ENCRYPTION_SECRET: change-this-demo-key-secret-at-least-32-characters
      MYSQL_ROOT_PASSWORD: change-this-demo-root-password
      MYSQL_PASSWORD: change-this-demo-db-password
      SESSION_COOKIE_SECURE: false
      CA_MANAGER_DEMO_SEED: "true"
      CA_MANAGER_DEMO_ADMIN_EMAIL: admin@acme.com
      CA_MANAGER_DEMO_ADMIN_PASSWORD: ChangeMe!8585
      CA_MANAGER_DEMO_ADMIN_NAME: Acme Demo Admin
    volumes:
      - ca-manager-demo-data:/var/lib/mysql

volumes:
  ca-manager-demo-data:
```

Run with Compose:

```bash
docker compose up -d
```

## Demo Data

By default, the image seeds an Acme Corp lab PKI with:

- `Acme Corp Lab Root CA G1`
- `Acme Corp Lab Issuing CA G1`
- `api.dev.acme.com`
- `portal.lab.acme.com`
- `vault.ops.acme.com`
- `acme-lab-automation-client`
- `vpn-user01.acme.com`
- `alice.admin@acme.com`
- `Acme Lab Build Signing`

Disable demo data on first run:

```bash
-e CA_MANAGER_DEMO_SEED=false
```

For Docker Compose:

```yaml
environment:
  CA_MANAGER_DEMO_SEED: "false"
```

## Reset Demo Data

The demo administrator and Acme Corp inventory are created only when the MySQL volume is initialized for the first time. If you pull a newer image or change the demo admin environment variables, reset the volume to apply first-run changes.

```bash
docker rm -f ca-manager-demo
docker volume rm ca-manager-demo-data
```

Then run the container again.

For Docker Compose:

```bash
docker compose down -v
docker compose up -d
```

## Required Environment Variables

| Variable | Purpose |
| --- | --- |
| `SESSION_SECRET` | Signs application sessions. Use a long random value. |
| `KEY_ENCRYPTION_SECRET` | Encrypts private key material stored in the database. Use a long random value. |
| `MYSQL_ROOT_PASSWORD` | Root password for the bundled MySQL instance. |
| `MYSQL_PASSWORD` | Password for the CA Manager database user. |
| `SESSION_COOKIE_SECURE` | Set to `false` for localhost HTTP demos. Use `true` only when serving over HTTPS. |
| `CA_MANAGER_DEMO_SEED` | Optional. Defaults to `true`. Set to `false` for a blank schema on first run. |
| `CA_MANAGER_DEMO_ADMIN_EMAIL` | Optional. Demo admin email. Defaults to `admin@acme.com`. |
| `CA_MANAGER_DEMO_ADMIN_PASSWORD` | Optional. Demo admin password. Defaults to `ChangeMe!8585`. |
| `CA_MANAGER_DEMO_ADMIN_NAME` | Optional. Demo admin display name. Defaults to `Acme Demo Admin`. |

## Scope

This image is intended for demos, training, and lab evaluation. It runs the app and database in one container for convenience. For production-style deployments, run CA Manager with separate application and database services.
