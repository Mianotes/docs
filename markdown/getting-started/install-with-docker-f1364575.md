# Installing with Docker

Created: 2026-06-07T14:20:31Z

## Note

The Docker install uses one Mianotes container. The container runs the web service and the dashboard together.

## Requirements

Install Docker Desktop, Docker Engine, or another Docker-compatible runtime that supports Docker Compose. You do not need to install Python, Node.js, FFmpeg, FLAC, or Tesseract on the host machine. Those tools are provided inside the Mianotes container.

## Folder layout

Create a folder for Mianotes:

```bash
mkdir mianotes
cd mianotes
```

Download the official Compose file and start Mianotes:

```bash
curl -fsSL https://raw.githubusercontent.com/Mianotes/install/main/docker-compose.yml -o docker-compose.yml
docker compose up -d
```

Open:

```text
http://localhost:8201
```

If Mianotes is running on another machine, replace `localhost` with the server address. Mianotes stores persistent data in a `data/` folder next to `docker-compose.yml`:

```text
mianotes/
  docker-compose.yml
  data/
    system.db
    workspaces/
    markdown/
    html/
```

The `data/` folder is mounted into the container at `/data`. This means files written by Mianotes inside the container are saved on your computer or server outside the container.

## docker-compose.yml

The downloaded `docker-compose.yml` should look like this. You can edit it before starting Mianotes if you need to change ports, volumes, or environment variables.

```yaml
services:
  mianotes:
    image: ghcr.io/mianotes/mianotes:latest
    container_name: mianotes
    restart: unless-stopped
    ports:
      - "8200:8200"
      - "8201:8201"
    volumes:
      - ./data:/data
    environment:
      MIANOTES_HOST: 0.0.0.0
      MIANOTES_PORT: 8200
      MIANOTES_DATA_DIR: /data
      MIANOTES_STORAGE_CONFIG_PATH: /data/workspaces.json
```

If you edited the file manually, start Mianotes with:

```bash
docker compose up -d
```

## View logs

```bash
docker compose logs -f
```

## Stop Mianotes

```bash
docker compose down
```

This stops and removes the container, but it does not delete your `data/` folder.

## Update Mianotes

```bash
docker compose pull
docker compose up -d
```

## Back up Mianotes

Back up the `data/` folder:

```bash
tar -czf mianotes-backup.tar.gz data
```

The backup includes your system database, workspace databases, Markdown files, source records, and published HTML.

## Remove Mianotes

Stop the container:

```bash
docker compose down
```

Then remove the Mianotes folder if you no longer need the data:

```bash
cd ..
rm -rf mianotes
```

Only delete the folder if you are sure you no longer need your notes.
