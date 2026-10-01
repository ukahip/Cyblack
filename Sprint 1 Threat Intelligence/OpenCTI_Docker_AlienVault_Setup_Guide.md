# OpenCTI + AlienVault OTX on Docker: Installation Guide

**Lab:** CTI Sprint (Desert Oil / Team 6)
**Target OS:** Ubuntu 24.04 LTS (VMware or bare metal)
**Stack:** Docker Engine, Docker Compose v2, OpenCTI, AlienVault OTX connector
**Audience:** SOC / CTI analyst building a local threat-intelligence lab

---

## 0. Before You Start

### 0.1 Minimum resources

| Resource | Bare minimum | Recommended |
|---|---|---|
| vCPU | 4 | 6-8 |
| RAM | 8 GB | **16 GB** |
| Disk | 50 GB | 100 GB (OTX ingestion grows quickly) |
| Network | Internet access | NAT or bridged |

> **Lesson learned:** OpenCTI with Elasticsearch, RabbitMQ, Redis, MinIO and workers will exhaust an 8 GB VM once the OTX connector starts a large backfill. Give it 16 GB if you can. On 8 GB, follow the 8 GB profile in Section 4.3 and keep the OTX start date recent. Several teammates on 8 GB VMs could not complete the install with default settings.

### 0.2 Version policy (pin everything)

Never use `latest` for a lab you want to reproduce or show to recruiters. Look up the current stable release at:

`https://github.com/OpenCTI-Platform/opencti/releases`

and pin it in `.env` (Section 4). The OpenCTI platform, worker and every connector image **must use the same version tag**. A mismatch between platform and worker is a common cause of "connector registered but nothing imports".

### 0.3 What you need in hand

- A Ubuntu 24.04 machine with `sudo`
- A free **AlienVault OTX account** and your OTX API key (Section 5.1)
- A browser that supports **WebGL 2** (see Troubleshooting)

---

## 1. Prepare the Host

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y ca-certificates curl gnupg git jq uuid-runtime openssl
```

### 1.1 Kernel setting required by Elasticsearch

Elasticsearch will crash-loop without this.

```bash
# Apply now
sudo sysctl -w vm.max_map_count=1048575

# Persist across reboots
echo "vm.max_map_count=1048575" | sudo tee /etc/sysctl.d/99-opencti.conf
sudo sysctl --system

# Verify
sysctl vm.max_map_count
```

Expected: `vm.max_map_count = 1048575`

### 1.2 (Optional) Add swap on small VMs

```bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
free -h
```

---

## 2. Install Docker Engine and Compose (Official Repository)

Do **not** use the `docker.io` or `docker-compose` packages from Ubuntu's default repos. They are older and the legacy `docker-compose` v1 binary is end-of-life. Use Docker's official apt repository.

### 2.1 Remove conflicting packages

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
  sudo apt remove -y $pkg 2>/dev/null
done
```

### 2.2 Add Docker's GPG key and repository

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
```

### 2.3 Install

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 2.4 Start on boot and allow your user to run Docker

```bash
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker        # or log out and back in
```

### 2.5 Verify

```bash
docker --version
docker compose version
docker run --rm hello-world
```

Note the syntax: **`docker compose`** (space, v2 plugin), not `docker-compose`.

### 2.6 (Recommended) Cap container log size

Unbounded container logs are a common reason the disk fills up.

```bash
sudo tee /etc/docker/daemon.json > /dev/null <<'EOF'
{
  "log-driver": "json-file",
  "log-opts": { "max-size": "50m", "max-file": "3" }
}
EOF
sudo systemctl restart docker
```

---

## 3. Get the Official OpenCTI Docker Files

Using the official repository guarantees the compose file matches the version you pin.

```bash
cd ~
git clone https://github.com/OpenCTI-Platform/docker.git opencti-docker
cd opencti-docker
ls -la
```

You should see `docker-compose.yml`, `.env.sample` and supporting files.

> To lock the compose file to a specific release, list the tags with `git tag --sort=-v:refname | head` and `git checkout <tag>` if your chosen release has one. Otherwise, the compose file reads the version from `.env`.

---

## 4. Configure OpenCTI (`.env`)

### 4.1 Generate secrets

```bash
# Admin password, tokens, and service credentials
echo "ADMIN_PASSWORD:      $(openssl rand -base64 18)"
echo "ADMIN_TOKEN (UUIDv4): $(uuidgen)"
echo "HEALTHCHECK_KEY:     $(uuidgen)"
echo "MINIO_ROOT_PASSWORD: $(openssl rand -hex 16)"
echo "RABBITMQ_PASS:       $(openssl rand -hex 16)"
echo "CONNECTOR_*_ID (one per connector):"; for i in 1 2 3 4 5; do uuidgen; done
```

Save these in a password manager. **Never commit `.env` to GitHub.**

### 4.2 Create `.env`

```bash
cp .env.sample .env

# Install Sublime Text once (skip if already installed)
sudo snap install sublime-text --classic

# Edit as your normal user (not with sudo)
subl .env
```

> **Sublime Text tips for config files:** set **View -> Indentation -> Indent Using Spaces** (width 2) before editing `docker-compose.yml`. YAML does not allow tabs, and a single tab will break the file. Do not open these files with `sudo subl`, because root-owned saves cause permission problems later.

Set (or confirm) at least the following. Use the values you generated above.

```ini
# --- Version (must match for platform, worker and all connectors) ---
OPENCTI_VERSION=<PINNED_VERSION_FROM_GITHUB_RELEASES>

# --- Platform ---
OPENCTI_BASE_URL=http://localhost:8080
OPENCTI_ADMIN_EMAIL=admin@desertoil.lab
OPENCTI_ADMIN_PASSWORD=<generated>
OPENCTI_ADMIN_TOKEN=<generated UUIDv4>
OPENCTI_HEALTHCHECK_ACCESS_KEY=<generated UUIDv4>

# --- Object storage (MinIO) ---
MINIO_ROOT_USER=opencti
MINIO_ROOT_PASSWORD=<generated>

# --- Message broker (RabbitMQ) ---
RABBITMQ_DEFAULT_USER=opencti
RABBITMQ_DEFAULT_PASS=<generated>

# --- Elasticsearch heap (see 4.3) ---
ELASTIC_MEMORY_SIZE=4G

# --- Connector IDs (each must be a unique UUIDv4) ---
CONNECTOR_EXPORT_FILE_STIX_ID=<uuid>
CONNECTOR_EXPORT_FILE_CSV_ID=<uuid>
CONNECTOR_EXPORT_FILE_TXT_ID=<uuid>
CONNECTOR_IMPORT_FILE_STIX_ID=<uuid>
CONNECTOR_IMPORT_DOCUMENT_ID=<uuid>
```

> **Variable names differ slightly between OpenCTI releases.** Always compare against the `.env.sample` that shipped with the version you cloned, and keep any extra variables it defines.

### 4.3 Memory tuning and the 8 GB profile

Setting `ELASTIC_MEMORY_SIZE` only limits one container. The whole stack has to fit in RAM together with the operating system, and on an 8 GB VM it usually does not without changes.

#### Where the memory goes (approximate)

| Component | Approx. real RAM use |
|---|---|
| Ubuntu Desktop and browser | 1.5-2.5 GB |
| Elasticsearch (2G heap plus off-heap overhead) | ~3 GB |
| OpenCTI platform (Node.js) | 1-2 GB, grows under load |
| RabbitMQ | 0.5-1 GB |
| Workers (official compose runs 3 replicas) | 0.3-0.5 GB each |
| Redis + MinIO | ~0.5 GB combined |
| Each connector | 0.1-0.3 GB |

The total is roughly 8-10 GB. On an 8 GB VM the Linux out-of-memory killer then stops a container, usually Elasticsearch or the platform, and the install looks broken.

#### Heap sizing by VM size

| VM RAM | `ELASTIC_MEMORY_SIZE` | Workers | Notes |
|---|---|---|---|
| 8 GB | `2G` | 1 | Use the 8 GB profile below. Borderline even then. |
| 12 GB | `3G` | 1-2 | Comfortable for the sprint |
| 16 GB | `4G` | 2-3 | Recommended |
| 32 GB+ | `8G` | 3+ | Large OTX backfills |

Do not set the heap below `2G`; Elasticsearch becomes unstable. Keep heap at or below about half the RAM you give the VM.

#### Step 1: Confirm memory is actually the problem

Run these on the VM that failed:

```bash
docker ps -a --format 'table {{.Names}}\t{{.Status}}'
docker inspect -f '{{.Name}} OOMKilled={{.State.OOMKilled}} ExitCode={{.State.ExitCode}}' $(docker ps -aq)
sudo dmesg | grep -iE "out of memory|killed process" | tail
free -h
```

| Result | Meaning |
|---|---|
| `OOMKilled=true` or exit code `137` | Out of memory. Continue with this section. |
| Elasticsearch exits immediately with a bootstrap error | `vm.max_map_count` not set (Section 1.1) |
| RabbitMQ crash-loops with ulimit errors | Section 4.4 |

#### Step 2: Apply the 8 GB profile

1. **Add 4 GB swap** (Section 1.2).
2. **Set `ELASTIC_MEMORY_SIZE=2G`** in `.env`.
3. **Reduce workers to one.** In the `worker` service of `docker-compose.yml`:

   ```yaml
     worker:
       # ...existing settings...
       deploy:
         mode: replicated
         replicas: 1        # was 3
   ```

4. **Remove optional services** you do not need for the sprint: `connector-export-file-csv`, `connector-export-file-txt` and `connector-import-document`. Keep the platform, worker, MITRE and AlienVault.
5. **Optional safety limits** so one container cannot take the whole VM. Add under the relevant services:

   ```yaml
     elasticsearch:
       mem_limit: 3g
     opencti:
       mem_limit: 2g
       environment:
         - NODE_OPTIONS=--max-old-space-size=1536   # add to the existing environment list
   ```

   Treat these as a safety net, not a fix. If a limit is too low the container is killed instead.
6. **Free OS memory.** Use Ubuntu Server (no desktop), or close other applications and browse from the host machine to `http://<VM-IP>:8080`.
7. **Start in stages** (Section 4.5) instead of bringing everything up at once.

#### Step 3: If 8 GB still fails

- **Raise the VM to 10-12 GB** if the host machine has 16 GB.
- **Share one instance.** One teammate with 16 GB runs OpenCTI and the others use it through the browser at `http://<host-IP>:8080`, each with their own user account (Settings -> Security -> Users). The sprint tasks only need the UI and the data. Restrict access with the firewall rules in Section 9.

### 4.4 Confirm RabbitMQ file-descriptor limits (known issue)

If RabbitMQ restarts repeatedly with `ulimit` or file-descriptor errors, add `ulimits` to the `rabbitmq` service in `docker-compose.yml`:

```yaml
  rabbitmq:
    # ...existing settings...
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
```

### 4.5 Start the platform

```bash
docker compose pull
docker compose up -d
```

**On an 8 GB VM, start in stages** so everything is not initialising at once:

```bash
docker compose up -d redis elasticsearch minio rabbitmq
docker compose ps                 # wait until these show healthy
docker compose up -d opencti      # wait until healthy (several minutes)
docker compose up -d worker
```

Add connectors one at a time afterwards (Section 5).

First start takes **3-10 minutes** while Elasticsearch initialises and the platform builds its schema. Watch progress:

```bash
docker compose ps
docker compose logs -f opencti
```

Ready when you see a line similar to `OpenCTI ... listening on port 8080` and `docker compose ps` shows the services as `healthy` or `running`.

### 4.6 Log in

- URL: `http://localhost:8080` (or `http://<VM-IP>:8080`; with VMware NAT use port forwarding)
- Email: value of `OPENCTI_ADMIN_EMAIL`
- Password: value of `OPENCTI_ADMIN_PASSWORD`

Your API token is under **Profile (top right) -> API access**. It should equal `OPENCTI_ADMIN_TOKEN`.

---

## 5. Integrate AlienVault OTX

The OTX connector pulls **pulses** (community threat reports) that you subscribe to and converts them into STIX objects in OpenCTI: reports, indicators, observables, malware, intrusion sets, and relationships.

### 5.1 Get your OTX API key

1. Create or log in to your account at `https://otx.alienvault.com`.
2. Click your username (top right) -> **Settings**.
3. Copy the **OTX Key**.
4. **Subscribe to pulses and users/groups** you want. The connector only imports pulses from your subscriptions, so an account with no subscriptions imports nothing. For the CTI Sprint, subscribe to sources relevant to oil and gas, energy, Middle East, Saudi Arabia, and state-sponsored actors.

Add the key to `.env`:

```ini
CONNECTOR_ALIENVAULT_ID=<new uuidv4>
ALIENVAULT_API_KEY=<your OTX key>
```

### 5.2 Choose the pulse start date (important)

`ALIENVAULT_PULSE_START_TIMESTAMP` controls how far back the first run goes. An old date (for example 2022) queues a very large backlog. That was the cause of the connector appearing "stuck" at a percentage while the VM ran out of disk and RAM.

For the CTI Sprint, tasks look at **the last 3 months**, so start close to that window:

```bash
# Print a date ~90 days ago in the required format
date -u -d '90 days ago' +"%Y-%m-%dT00:00:00"
```

Use that value below. You can move the date earlier later once you have headroom.

### 5.3 Add the connector to `docker-compose.yml`

Append this service under `services:` (same indentation as the other connectors):

```yaml
  connector-alienvault:
    image: opencti/connector-alienvault:${OPENCTI_VERSION}
    environment:
      - OPENCTI_URL=http://opencti:8080
      - OPENCTI_TOKEN=${OPENCTI_ADMIN_TOKEN}
      - CONNECTOR_ID=${CONNECTOR_ALIENVAULT_ID}
      - CONNECTOR_NAME=AlienVault
      - CONNECTOR_SCOPE=alienvault
      - CONNECTOR_LOG_LEVEL=info
      - ALIENVAULT_BASE_URL=https://otx.alienvault.com
      - ALIENVAULT_API_KEY=${ALIENVAULT_API_KEY}
      - ALIENVAULT_TLP=White
      - ALIENVAULT_CREATE_OBSERVABLES=true
      - ALIENVAULT_CREATE_INDICATORS=true
      - ALIENVAULT_PULSE_START_TIMESTAMP=2026-07-01T00:00:00   # set to ~90 days ago
      - ALIENVAULT_REPORT_TYPE=threat-report
      - ALIENVAULT_REPORT_STATUS=New
      - ALIENVAULT_GUESS_MALWARE=false
      - ALIENVAULT_GUESS_CVE=false
      - ALIENVAULT_EXCLUDED_PULSE_INDICATOR_TYPES=FileHash-MD5,FileHash-SHA1
      - ALIENVAULT_ENABLE_RELATIONSHIPS=true
      - ALIENVAULT_ENABLE_ATTACK_PATTERNS_INDICATES=false
      - ALIENVAULT_INTERVAL_SEC=1800
    restart: always
    depends_on:
      opencti:
        condition: service_healthy
```

Notes on the settings:

| Setting | Why |
|---|---|
| `ALIENVAULT_EXCLUDED_PULSE_INDICATOR_TYPES=FileHash-MD5,FileHash-SHA1` | Cuts volume sharply. SHA-256 is kept. |
| `ALIENVAULT_GUESS_MALWARE` / `GUESS_CVE` | Keep `false` to avoid false-positive links from keyword matching. |
| `ALIENVAULT_TLP=White` | Marks imported data TLP:CLEAR/WHITE (OTX is community data). |
| `ALIENVAULT_INTERVAL_SEC=1800` | Polls every 30 minutes. |

> Option names evolve. Check the connector's README for your pinned version: `https://github.com/OpenCTI-Platform/connectors/tree/master/external-import/alienvault`. If the `depends_on` healthcheck condition errors on your compose version, remove that block and rely on `restart: always`.

### 5.4 Start the connector

```bash
docker compose up -d connector-alienvault
docker compose logs -f connector-alienvault
```

Healthy output shows the connector registering with OpenCTI and then fetching pulses. Typical lines: connector started, retrieving pulses since the configured timestamp, sending bundles.

### 5.5 Verify in the OpenCTI UI

1. **Data -> Ingestion -> Connectors**: `AlienVault` should be listed with status **Active** (green).
2. **Data -> Ingestion -> Connectors -> AlienVault**: check the **messages in progress** counter. It rises, then falls as workers process bundles.
3. **Analyses -> Reports**: new reports with the `AlienVault` creator appear.
4. **Observations -> Indicators / Observables**: counts increase.
5. **Threats -> Intrusion sets / Malware**: entities appear and link to reports.

> Seeing the "operations in progress" number climb into the hundreds of thousands during backfill is normal. It means the queue is processing. If it **does not move** for 30+ minutes, go to Troubleshooting.

### 5.6 (Recommended) Add the MITRE ATT&CK connector

Task 1 asks for MITRE ATT&CK TTP mapping. Add the MITRE dataset connector:

```yaml
  connector-mitre:
    image: opencti/connector-mitre:${OPENCTI_VERSION}
    environment:
      - OPENCTI_URL=http://opencti:8080
      - OPENCTI_TOKEN=${OPENCTI_ADMIN_TOKEN}
      - CONNECTOR_ID=${CONNECTOR_MITRE_ID}
      - CONNECTOR_NAME=MITRE ATT&CK
      - CONNECTOR_SCOPE=tool,report,malware,identity,campaign,vulnerability,attack-pattern,course-of-action,intrusion-set,relationship,x-mitre-data-component,x-mitre-data-source,x-mitre-tactic,x-mitre-matrix
      - CONNECTOR_LOG_LEVEL=info
      - MITRE_REMOVE_STATEMENT_MARKING=true
      - MITRE_INTERVAL=7
    restart: always
```

Generate `CONNECTOR_MITRE_ID` with `uuidgen` and add it to `.env`. Start with `docker compose up -d connector-mitre`. Load MITRE first and let it finish before the heavy OTX run, because OTX relationships link to ATT&CK objects.

---

## 6. Using the Data for the CTI Sprint

| Sprint task | Where in OpenCTI |
|---|---|
| Task 1: Sector landscape | **Entities -> Sectors -> Oil and Gas** (use the Knowledge tab and the threats/intrusion sets tabs) |
| Task 2: National threats | **Entities -> Locations -> Saudi Arabia**, then the threats targeting it |
| Task 3: Recent victims | **Analyses -> Reports**, filter by date (last 3 months), then open each report's Knowledge graph for Diamond Model and Kill Chain mapping |
| Task 4: Political actor | **Threats -> Intrusion sets / Threat actors**, filter by motivation (ideology, geopolitical) and recent activity |
| Task 5: Board deck | Export graphs and screenshots from the above |

Tip: use **Knowledge -> Graph** view on a report to screenshot relationships for your Diamond Model.

---

## 7. Operations Cheat Sheet

```bash
# Status and resource use
docker compose ps
docker stats --no-stream
df -h /var/lib/docker

# Logs
docker compose logs --tail=100 opencti
docker compose logs --tail=100 connector-alienvault
docker compose logs --tail=100 rabbitmq

# Restart one service
docker compose restart connector-alienvault

# Stop / start the whole stack (data kept in volumes)
docker compose stop
docker compose start

# Clean unused images (safe)
docker image prune -f
```

> **Danger:** `docker compose down -v` deletes all volumes, which means **all OpenCTI data**. Use plain `docker compose down` unless you really want a full reset.

### Backup the volumes (before upgrades)

```bash
docker compose stop
sudo tar czf ~/opencti-volumes-$(date +%F).tar.gz -C /var/lib/docker/volumes .
docker compose start
```

### Upgrade procedure

1. Back up (above).
2. Read the release notes for breaking changes between your version and the target.
3. Change `OPENCTI_VERSION` in `.env` (platform, worker and connectors move together).
4. `docker compose pull && docker compose up -d`.
5. Check logs, then confirm connectors are Active.

---

## 8. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Elasticsearch exits immediately | `vm.max_map_count` too low | Section 1.1 |
| Any container killed (exit 137, `OOMKilled=true`) | Out of memory | Apply the 8 GB profile (Section 4.3): swap, 1 worker, fewer connectors, staged start, or more RAM |
| RabbitMQ restarting, ulimit errors | File descriptor limit | Section 4.4 |
| `opencti` container unhealthy for long | Elasticsearch or Redis not ready | `docker compose logs elasticsearch`, wait, then check RAM |
| Login page loads but graphs are blank / "WebGL" error | Browser has no WebGL 2 (common in VMs) | Enable 3D acceleration in the VM settings, or use Chrome/Firefox on the host to reach `http://<VM-IP>:8080`. Check `chrome://gpu` |
| AlienVault connector shows Active but nothing imports | No OTX subscriptions, wrong API key, or version mismatch | Subscribe to pulses; verify key; match image tags to platform version |
| Connector "stuck" at a percentage | Huge backlog, or workers starved of RAM/disk | Check `docker stats` and `df -h`; shorten `PULSE_START_TIMESTAMP`; scale workers only if RAM allows |
| Disk fills up | Elasticsearch data plus container logs | Log rotation (2.6), shorter OTX window, bigger disk |
| `docker: permission denied` | User not in `docker` group | Section 2.4 |
| `docker-compose: command not found` | Using v2 | Use `docker compose` |
| Connector crash-loops with auth error | Wrong `OPENCTI_TOKEN` | Compare token in UI (Profile -> API access) with `.env` |

### Quick diagnostic sequence

```bash
free -h; df -h; docker compose ps
docker compose logs --tail=50 opencti | grep -iE "error|warn"
docker compose logs --tail=50 connector-alienvault
docker compose exec rabbitmq rabbitmqctl list_queues name messages 2>/dev/null | head
```

### Reducing load on a small VM

- Stop connectors you do not need (`docker compose stop connector-<name>`).
- Keep a single worker (default is fine for a lab).
- Exclude MD5/SHA1 indicators (already in 5.3).
- Let MITRE finish first, then OTX.

---

## 9. Security Hardening Checklist

- [ ] Change the default admin email and use a strong generated password
- [ ] Rotate the OpenCTI token and OTX API key if they ever appear in screenshots, chat, or Git
- [ ] `.env` is in `.gitignore`; never push it
- [ ] Expose port 8080 only to localhost or a trusted network (`ufw allow from <your-IP> to any port 8080`)
- [ ] Do not publish RabbitMQ (15672), Elasticsearch (9200) or MinIO (9000/9001) ports beyond the host
- [ ] Put OpenCTI behind a TLS reverse proxy (Caddy or Nginx) if reachable from anywhere other than your own machine
- [ ] Redact tokens, keys and emails from every portfolio screenshot
- [ ] Keep images pinned and update on a schedule

```bash
# Basic host firewall for a lab VM
sudo ufw default deny incoming
sudo ufw allow ssh
sudo ufw allow from <YOUR_HOST_IP> to any port 8080 proto tcp
sudo ufw enable
```

---

## 10. Portfolio Evidence Checklist

Capture these for the report (blur secrets):

- [ ] `docker --version` and `docker compose version`
- [ ] `docker compose ps` showing all services healthy
- [ ] OpenCTI login and dashboard
- [ ] **Data -> Ingestion -> Connectors** showing AlienVault Active
- [ ] OTX account subscriptions page (without the API key)
- [ ] A report imported by the AlienVault connector, with its knowledge graph
- [ ] Sector entity page for **Oil and Gas**
- [ ] Location page for **Saudi Arabia**

---

## References

- OpenCTI documentation: `https://docs.opencti.io`
- OpenCTI Docker repository: `https://github.com/OpenCTI-Platform/docker`
- OpenCTI connectors (AlienVault, MITRE): `https://github.com/OpenCTI-Platform/connectors`
- Docker Engine install (Ubuntu): `https://docs.docker.com/engine/install/ubuntu/`
- AlienVault OTX: `https://otx.alienvault.com`
