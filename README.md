# WordPress Docker Compose

Multi-container setup that runs WordPress together with a MySQL database using Docker Compose.

This repository contains everything needed to run a WordPress site in two containers:

- `wordpress` serves the site and is published on port `8080`.
- `db` runs the MySQL database that stores the WordPress content.

Both services share one network and keep their data in Docker volumes, so content survives restarts. Credentials are not part of the repository; they are passed to the containers through a local `.env` file.

## Table of Contents

- [Repository Contents](#repository-contents)
- [Quickstart](#quickstart)
- [Usage](#usage)

## Repository Contents

| File | Purpose |
| --- | --- |
| `docker-compose.yaml` | Defines the `wordpress` and `db` services, their volumes and the shared network. |
| `.env.example` | Template for the `.env` file that holds the credentials. |
| `.gitignore` | Keeps the `.env` file and other local artifacts out of the repository. |
| `README.md` | This documentation. |

## Quickstart

### Prerequisites

- [Docker Engine](https://docs.docker.com/engine/install/) with the Docker Compose plugin
- Port `8080` reachable on the host

### Steps

1. Clone the repository and change into it:

   ```bash
   git clone https://github.com/DaKo1312/wordpress-docker-compose.git
   cd wordpress-docker-compose
   ```

2. Create the `.env` file from the template and replace all placeholder values:

   ```bash
   cp .env.example .env
   ```

3. Start the services:

   ```bash
   docker compose up -d
   ```

4. Open `http://<your-server-ip>:8080` in a browser and complete the WordPress installation, where the admin user is created.

## Usage

### Configuration

All settings are read from the `.env` file next to `docker-compose.yaml`.

| Variable | Required | Default | Description |
| --- | --- | --- | --- |
| `MYSQL_ROOT_PASSWORD` | yes | - | Password of the MySQL root user. |
| `MYSQL_USER` | yes | - | Database user that is created for WordPress. |
| `MYSQL_PASSWORD` | yes | - | Password of that database user. |
| `WORDPRESS_DB_USER` | yes | - | User WordPress connects with. Must equal `MYSQL_USER`. |
| `WORDPRESS_DB_PASSWORD` | yes | - | Password WordPress connects with. Must equal `MYSQL_PASSWORD`. |
| `DB_NAME` | no | `wordpress` | Name of the database used by both services. |
| `WORDPRESS_PORT` | no | `8080` | Host port on which WordPress is published. |
| `WORDPRESS_TABLE_PREFIX` | no | `wp_` | Prefix of the WordPress database tables. |

Credentials are only passed through `env_file`; the `environment` blocks in `docker-compose.yaml` contain non-critical values only.

The database is initialized on the first start. Changing `MYSQL_*` values or `DB_NAME` afterwards has no effect on an existing `db_data` volume.

### Common changes

- **Different port:** set `WORDPRESS_PORT=80` in `.env` and run `docker compose up -d`.
- **Different image versions:** change the `image` tags of the services in `docker-compose.yaml`, then run `docker compose pull` and `docker compose up -d`.
- **Different restart behavior:** both services use `restart: unless-stopped`, so they come back after a crash or a reboot. Replace the value, for example with `always` or `on-failure`.

### Operating the setup

```bash
docker compose ps        # show the state of both services
docker compose logs -f   # follow the logs
docker compose restart   # restart both services
docker compose down      # stop and remove the containers, data is kept
```

### Data persistence

| Volume | Mounted in | Content |
| --- | --- | --- |
| `db_data` | `db` at `/var/lib/mysql` | Database files |
| `wordpress_data` | `wordpress` at `/var/www/html` | WordPress files, themes, plugins and uploads |

The volumes survive `docker compose down` and restarts. To delete all data and start from scratch, run `docker compose down -v`.
