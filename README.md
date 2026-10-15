# compose-to-k8s

An incident reporting portal, similar to an IT helpdesk ticket form, built
with FastAPI, PostgreSQL and Redis. The project starts as a Docker Compose
application and grows into a deployment on a hardened VPS with k3s, GitOps
(Argo CD) and HTTPS: a hands-on DevSecOps learning project.

## Project status

- [x] Repository setup, branch protection, architecture decisions
- [ ] Application: form, report list, PostgreSQL, Redis, rate limiting
- [ ] Docker Compose: one-command local run
- [ ] Continuous integration: tests, image build, vulnerability scan
- [ ] Hardened VPS
- [ ] Kubernetes (k3s)
- [ ] GitOps: Argo CD and Sealed Secrets
- [ ] Public exposure with HTTPS
- [ ] Kubernetes and application hardening

## Getting started

Requirements: Git and Docker with Compose.

    git clone https://github.com/HakeemMHB/compose-to-k8s.git
    cd compose-to-k8s
    docker compose up

Then open http://localhost:8000. No configuration file is needed for a
local run.

The commands to inspect the data in each database will be described in
`docs/demo.md`.

## Stack

| Layer | Tools |
|---|---|
| Application | Python, FastAPI, Jinja2, Pydantic |
| Data | PostgreSQL, Redis |
| Containers | Docker, Docker Compose |
| Continuous integration | GitHub Actions, Trivy, GitHub Container Registry (GHCR) |
| Deployment | VPS, k3s, Kustomize |
| Continuous deployment | Argo CD, Sealed Secrets |
| Exposure | Traefik, cert-manager, Let's Encrypt |

## Why two databases

- **PostgreSQL is the source of truth.** Every report is stored in full and
  durably, in a table whose schema enforces the expected structure.
- **Redis is the fast layer.** It holds the latest reports, counters per
  severity and per-IP rate limiting. Atomic counters (`INCR`) and automatic
  expiry (`EXPIRE`) make rate limiting simple, with no cleanup job.

MongoDB and Neo4j were considered and rejected: reports share a fixed
structure and have no relationships between them. See
[ADR 001](docs/adr/001-use-postgresql-and-redis-for-data-storage.md).

## Security by layer

| Layer | Measures |
|---|---|
| Server | SSH by key only, password and root login disabled, minimal firewall, fail2ban, automatic security updates |
| Cluster | Default-deny NetworkPolicies, Pod Security `restricted`, encrypted secrets in Git |
| Application | Input validation, per-IP rate limiting, output escaping, parameterized SQL queries, security headers |

## Delivery workflow

- **Continuous integration:** every pull request runs linting and tests. On
  `main`, the image is built, scanned with Trivy and published to GHCR.
- **Continuous deployment:** Argo CD watches the repository and applies every
  change merged on `main` to the cluster. Nobody changes the server by hand.

### Why pull requests on a solo project

- `main` always works: anyone can clone the project at any time.
- Tests run on the pull request, before the change reaches `main`.
- Reviewing the full diff before merging catches mistakes.
- Each change keeps its *what* and *why* in the pull request description,
  linked from the commit on `main`.

## Documentation

- [Architecture decisions](docs/adr/)
- Architecture overview, demo procedure and troubleshooting journal: coming
  with the next steps

## License

[MIT](LICENSE)
