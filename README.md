# Docker + Nginx Lab

A containerized website on a hardened CentOS Stream 9 server (AWS EC2):
custom Nginx image, reverse proxy, load balancing and container hardening.

## Architecture

Browser -> proxy (Nginx, port 80, the only published service)
        -> web (custom Nginx image, serves the site)
        -> api (demo service, scaled to 3 replicas during testing)

All services share a private Docker network and run as non-root users.

## What I built

- Custom image (web/Dockerfile): non-root Nginx on Alpine, security headers,
  hidden version, /healthz endpoint and a Docker HEALTHCHECK
- Docker Compose stack (compose.yaml): reverse proxy, health-based startup
  order, restart policies, private network
- Load balancing demo: scaled the API to 3 containers and verified round-robin
- Hardening: read-only filesystems, cap_drop ALL, no-new-privileges,
  memory and PID limits, log rotation, only the proxy publishes a port
- Scanning: scanned the image with Trivy, rebuilt on the latest base image
  and reached 0 vulnerabilities (a HIGH libexpat finding in the previous build was fixed)
- Operations: volume backup and restore, container checks added to my health script

## Troubleshooting drills

| Symptom | Cause | Fix |
|---|---|---|
| Proxy would not start | Missing semicolon in Nginx config | Read logs, fixed config |
| 502 Bad Gateway | api container stopped | Started api, reloaded proxy |
| host not found in upstream | Wrong service name | Corrected name in config |

## Challenges and what I learned

- Docker published ports bypass host firewall rules, so I rely on the AWS
  Security Group (source: my IP) and publish only what is needed.
- SSH rejected my private key because Windows file permissions were too open;
  I fixed it with icacls and found the correct key pair for the instance.

## Run it

docker compose up -d --build

## Screenshots

See the screenshots/ folder.

## Tools

CentOS Stream 9, Docker Engine, Docker Compose, Nginx, Trivy, AWS EC2
