# WordPress Docker Compose

Multi-container setup that runs WordPress together with a MySQL database using Docker Compose.

## Table of Contents

- [Description](#description)
- [Quickstart](#quickstart)

## Description

This repository contains everything needed to run a WordPress site in two containers:

- `wordpress` serves the site and is published on port `8080`.
- `db` runs the MySQL database that stores the WordPress content.

Both services share one network and keep their data in Docker volumes, so content survives restarts. Credentials are not part of the repository; they are passed to the containers through a local `.env` file.

### Repository contents

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
