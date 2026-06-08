# CA Manager Demo

CA Manager Demo is a single-container demo distribution of CA Manager for non-critical lab and evaluation environments. The image starts CA Manager in production mode, starts a bundled MySQL 8 database, initializes the schema on first run, and can preload an Acme Corp sample PKI inventory.

This repository does not contain the CA Manager application source code. It contains only the public demo run files and documentation. The Docker image is built from the private CA Manager source repository and published separately to Docker Hub.

## What is included

- CA Manager API and web UI in one container
- MySQL 8 bundled in the same container
- First-run schema initialization
- Optional Acme Corp demo seed data
- Persistent database volume
- Demo UI/API on port `8585`

The Acme Corp seed creates:

- `Acme Corp Lab Root CA G1`
- `Acme Corp Lab Issuing CA G1`
- `api.dev.acme.com`
- `portal.lab.acme.com`
- `vault.ops.acme.com`
- `acme-lab-automation-client`
- `vpn-user01.acme.com`
- `alice.admin@acme.com`
- `Acme Lab Build Signing`

## Quick start

1. Copy the example environment file:

```bash
cp .env.example .env
```

2. Edit `.env` and set `CA_MANAGER_DEMO_IMAGE` to the Docker Hub image you published, for example:

```bash
CA_MANAGER_DEMO_IMAGE=apimonster/ca-manager-demo:latest
```

3. Start the demo:

```bash
docker compose up -d
```

4. Open the UI:

```text
http://localhost:8585
```

For localhost HTTP demos, keep `SESSION_COOKIE_SECURE=false` so the browser can store the login session cookie.

Default demo login:

```text
Email: admin@acme.com
Password: ChangeMe!8585
```

Change `CA_MANAGER_DEMO_ADMIN_EMAIL`, `CA_MANAGER_DEMO_ADMIN_PASSWORD`, and `CA_MANAGER_DEMO_ADMIN_NAME` in `.env` before first startup if you want different credentials. The demo CA/certificate inventory is loaded if `CA_MANAGER_DEMO_SEED=true`.

## Run without Compose

```bash
docker run -d \
  --name ca-manager-demo \
  -p 8585:8585 \
  -v ca-manager-demo-data:/var/lib/mysql \
  -e SESSION_SECRET="change-this-demo-session-secret-at-least-32-characters" \
  -e KEY_ENCRYPTION_SECRET="change-this-demo-key-secret-at-least-32-characters" \
  -e MYSQL_ROOT_PASSWORD="change-this-demo-root-password" \
  -e MYSQL_PASSWORD="change-this-demo-db-password" \
  -e SESSION_COOKIE_SECURE=false \
  -e CA_MANAGER_DEMO_SEED=true \
  apimonster/ca-manager-demo:latest
```

## Reset the demo data

The MySQL data is stored in the `ca-manager-demo-data` Docker volume. To reset the demo completely:

```bash
docker compose down -v
docker compose up -d
```

This removes the database volume and causes the next start to initialize a fresh schema and rerun the Acme Corp seed.

## Blank database mode

To start with an empty schema instead of Acme Corp sample data, set this in `.env` before the first run:

```bash
CA_MANAGER_DEMO_SEED=false
```

If the database volume already exists, changing this value will not remove existing data. Reset the volume first if you want a clean blank instance.

## Documentation

- [User Manual](docs/USER_MANUAL.md)
- [Marketing Flyer](docs/ca-manager-lab-flyer.pdf)
- [Feature Brief](docs/ca-manager-feature-brief.pdf)

## Demo scope

This image is intended for demonstrations, training, and lab evaluation. It deliberately trades operational separation for easy startup by running the app and MySQL in one container. For production-style deployments, run CA Manager with separate application and database services.

## Useful commands

Check status:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs -f
```

Stop the demo while keeping data:

```bash
docker compose down
```

Stop and delete demo data:

```bash
docker compose down -v
```
