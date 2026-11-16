# WordPress Docker Compose

Multi-container setup that runs WordPress together with a MySQL database using Docker Compose.

The repository contains the Compose file and an environment template to start both services on any Docker host. Data is kept in Docker volumes, and credentials are passed in through a local `.env` file.

## Table of Contents

- [Quickstart](#quickstart)
- [Usage](#usage)
- [Repository Contents](#repository-contents)
- [Testing](#testing)
- [Security](#security)

## Quickstart

### Prerequisites

- [Docker Engine](https://docs.docker.com/engine/install/) with the Docker Compose plugin
- Port `8080` reachable on the host

### Steps

1. Clone the repository and change into it:

   ```bash
   git clone git@github.com:DaKo1312/wordpress-docker-compose.git
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

## Repository Contents

| File | Purpose |
| --- | --- |
| `docker-compose.yaml` | Defines the `wordpress` and `db` services, their volumes and the shared network. |
| `.env.example` | Template for the `.env` file that holds the credentials. |
| `.gitignore` | Keeps the `.env` file and other local artifacts out of the repository. |
| `README.md` | This documentation. |

## Testing

Replace `<your-server-ip>` with the address of your host.

**Reachable on port 8080.** Run this from a machine that is not the server:

```bash
curl --head http://<your-server-ip>:8080/wp-login.php   # expect: HTTP/1.1 200 OK
```

**Admin login.** Open `http://<your-server-ip>:8080/wp-admin`, log in with the admin user created during installation and navigate the dashboard.

**Data survives a restart.** Change something in WordPress, for example the site title, then recreate the containers and reload the page:

```bash
docker compose down && docker compose up -d
```

The change and the admin account are still there, because both volumes are kept.

**Containers restart after a failure.** `docker kill` counts as a deliberate stop and does not trigger the restart policy, so end the main process from inside the container:

```bash
docker compose exec wordpress sh -c "kill 1"
docker compose exec db sh -c "kill 1"
docker compose ps   # both services are "Up" again after a few seconds
docker inspect --format '{{.Name}} restarts={{.RestartCount}}' $(docker compose ps -q)
```

`RestartCount` is `1` for both containers.

## Security

- No passwords, user names or IP addresses are tracked; the documentation uses placeholders such as `<your-server-ip>`.
- Credentials reach the containers only through the untracked `.env` file. Restrict it with `chmod 600 .env`.
- Only the WordPress port is published. The database has no published port and is reachable only inside the `wordpress_net` network.
- Both services use the official images with pinned version tags.
- The setup serves plain HTTP. For anything beyond testing, put a reverse proxy with TLS in front of it and choose a strong admin password.
