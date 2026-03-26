# cora-docker-tomcat

Production-ready Apache Tomcat Docker image for the [Cora](https://github.com/lsu-ub-uu) project (Uppsala University Library).

## Quick start (development)

Build the image from the `docker/` directory:

```bash
docker build -t cora-docker-tomcat ./docker
```

Run it:

```bash
docker run -p 8080:8080 cora-docker-tomcat
```

## Production deployment

Use the provided production Compose file for a hardened setup:

```bash
# 1. Copy and edit the environment file
cp .env.example .env
#    Set AJP_SECRET to a strong random value
#    Adjust CATALINA_OPTS if needed

# 2. Build and start
docker compose -f docker-compose.prod.yml up -d --build
```

### Key production settings

| Setting | Value | Rationale |
|---|---|---|
| Base image | `tomcat:11.0.13-jre25-temurin-noble` | JRE-only – no compiler in production |
| User | `tomcat` (non-root) | Principle of least privilege |
| Shutdown port | `-1` (disabled) | Prevents remote shutdown attacks |
| AJP secret | Required (`AJP_SECRET` env var) | Prevents AJP ghostcat-style attacks |
| JVM flags | Container-aware (`-XX:+UseContainerSupport`, `MaxRAMPercentage=75%`) | Respects container memory limits |
| GC | G1GC | Low-latency default for server workloads |
| Healthcheck | TCP check on port 8080 every 30 s | Automatic restart on failure |
| Logging | JSON with 10 MB × 5 file rotation | Prevents disk exhaustion |
| Filesystem | Read-only + tmpfs for `work`/`temp` | Reduces attack surface |
| Capabilities | All dropped | Minimal Linux capabilities |
| `no-new-privileges` | Enabled | Prevents privilege escalation |

### Tuning parameters

The following JVM parameters are set via `CATALINA_OPTS` and can be overridden in your `.env` file:

- **`-XX:MaxRAMPercentage=75.0`** – uses 75 % of the container memory limit for heap.
- **`-XX:InitialRAMPercentage=50.0`** – pre-allocates 50 % to reduce GC pauses at startup.
- **`-XX:+UseG1GC`** – G1 garbage collector, a good default for request/response workloads.
- **`-XX:+ExitOnOutOfMemoryError`** – ensures the container restarts cleanly on OOM instead of hanging.
- **`-Djava.security.egd=file:/dev/urandom`** – avoids blocking on entropy in containers.

The Compose file reserves 512 MB and limits to 1 GB of memory (adjust in `docker-compose.prod.yml`).

## Container security checklist

- [x] Non-root user in Dockerfile (`USER tomcat`)
- [x] Read-only root filesystem with writable tmpfs mounts
- [x] All Linux capabilities dropped
- [x] `no-new-privileges` security option enabled
- [x] Default Tomcat webapps removed (manager, examples, docs, host-manager, ROOT)
- [x] Remote shutdown port disabled (`port="-1"`)
- [x] AJP connector requires a shared secret
- [x] Server header masked (`server="Apache"`)
- [x] Base image pinned to a specific version tag
- [x] JRE-only runtime image (no JDK/compiler tools)
- [x] Healthcheck configured for automatic recovery
- [x] Log rotation configured to prevent disk exhaustion
- [ ] TLS termination – handle at the reverse proxy / load balancer level

## License

Copyright 2020 Uppsala University Library. Licensed under the [GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.en.html).
