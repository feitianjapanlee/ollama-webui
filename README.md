# ollama-webui
An ollama + open-webui docker composition, contains a **Docker Compose** setup for running **Ollama** and **Open‑WebUI** side‑by‑side.  
The services are:

| Service | Image | Purpose |
|---------|-------|---------|
| `ollama` | `ollama/ollama` | Runs the Ollama LLM server |
| `open-webui` | `ghcr.io/open-webui/open-webui:main` | Provides a web interface that talks to Ollama |

The Compose file is stored in the repository root as `docker-compose.yml`.

---

## Prerequisites

| Item | Minimum Version | Notes |
|------|-----------------|-------|
| **Docker Engine** | 27.x or newer (tested with 28.4.0) | Install from <https://docs.docker.com/engine/install/> |
| **Docker Compose** | v2 (bundled with Docker Desktop) | `docker compose` command is available |
| **GPU (optional)** | Supported GPU driver | The Compose file reserves a GPU for Ollama (see `deploy.resources.reservations.devices`). If you don't have a GPU, remove that block. |
| **Disk Space** | At least 2 GB | For images and persisted data |

---

## Project Layout

```
.
├─ docker-compose.yml          # Service definitions
├─ ollama/.ollama/             # Persisted Ollama data (will created automatically by docker-compose.yml) 
└─ open-webui/data/            # Persisted Open‑WebUI data (will created automatically by docker-compose.yml)
```

- `ollama/.ollama` is mounted into the Ollama container at `/root/.ollama`.  
- `open-webui/data` is mounted into the Open‑WebUI container at `/app/backend/data`.

---

## Running the Stack

1. **Clone the repository**  
   ```bash
   git clone https://github.com/feitianjapanlee/ollama-webui.git
   cd ollama-webui
   ```

2. **Build and start the services**  
   ```bash
   docker compose up -d
   ```

   - `-d` runs containers in the background.
   - The first run will pull the images and build the containers. Subsequent runs will skip the pull step.

3. **Verify the services are running**  
   ```bash
   docker compose ps
   ```
   You should see both `ollama` and `open-webui` in the `State` column.

4. **Access the web UI**  
   Open a browser and go to `http://localhost:8080`.

---

## Stopping & Cleaning Up

```bash
# Stop the containers
docker compose down
```

If you want to remove the persisted data as well, delete the directories:

```bash
rm -rf ollama/.ollama
rm -rf open-webui/data
```

---

## Data Persistence

- **Ollama data** is stored in `ollama/.ollama`.  
  This includes model files, cache, and any custom configurations.  
  The directory is mounted as a volume inside the `ollama` container, so the data survives container restarts.

- **Open‑WebUI data** is stored in `open-webui/data`.  
  It holds user settings, chat history, and other stateful information.  
  The directory is also mounted as a volume inside the `open-webui` container.

Feel free to back up these directories or move them to a different location if you need to preserve the data across machine migrations.

---

## Environment Variables

The Compose file sets one environment variable for Open‑WebUI:

```yaml
environment: OLLAMA_API_BASE_URL=http://ollama:11434
```

If you change the Ollama service name or port, update this value accordingly.

---

## Customization Tips

- **Changing the model** – Pull or load a new model into the Ollama container and restart it.  
- **Adding more GPUs** – Modify the `deploy.resources.reservations.devices.capabilities` array.  
- **Exposing additional ports** – Add more entries under each service’s `ports` section.  

---

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|--------------|-----|
| `docker compose up` stalls at “Downloading …” | Slow internet or firewall | Check connectivity |
| Open‑WebUI shows “Cannot connect to Ollama” | Port mapping wrong or Ollama not started | Ensure `ollama` is healthy (`docker compose ps`) |
| GPU not allocated | GPU driver missing or wrong Docker settings | Install GPU drivers and enable `--gpus all` in Docker Desktop |

---

