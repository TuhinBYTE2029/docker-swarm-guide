# Docker Swarm Production Guide

The complete guide to running Docker Swarm in production: installation, networking, compose files, Dockerfiles, monitoring, and deployment best practices.

### [Read the Guide on Our Website](https://thedecipherist.github.io/docker-swarm-guide/?utm_source=github&utm_medium=readme&utm_campaign=docker-swarm-guide&utm_content=hero-cta) ← Best reading experience

> **TL;DR:** I've been running Docker Swarm in production on AWS for years. This guide covers everything from the Swarm hierarchy to advanced production configurations - DNS that actually works, encrypted overlay networks, multi-stage Dockerfiles, CI/CD versioning, and battle-tested monitoring stacks.

---

## Table of Contents

- [Why Docker Swarm in 2026?](#why-docker-swarm-in-2026)
- [What's Covered](#whats-covered)
- [Quick Start](#quick-start)
- [Key Topics](#key-topics)
- [Contributing](#contributing)
- [License](#license)

---

## Why Docker Swarm in 2026?

Docker Swarm is still incredibly relevant for small-to-medium teams who want container orchestration without the complexity overhead:

- Native Docker integration (no YAML hell beyond compose files)
- Significantly lower learning curve than Kubernetes
- Perfect for 2-20 node clusters
- Built-in service discovery and load balancing
- Rolling updates out of the box
- Works with your existing Docker Compose files

If you're not running thousands of microservices across multiple data centers, Swarm might be exactly what you need.

---

## What's Covered

| Topic | Key Takeaway |
|-------|--------------|
| Swarm Hierarchy | Swarm → Nodes → Stacks → Services → Tasks |
| VPS Requirements | Infrastructure presets and cost planning |
| Installation | Production-ready setup on Ubuntu |
| DNS Configuration | The #1 cause of Swarm networking issues |
| Overlay Networks | Encrypted inter-node communication |
| Compose Files | Production-ready with deploy, resources, rollback |
| Dockerfiles | Multi-stage builds, health checks, security |
| Monitoring | Prometheus + Grafana + cAdvisor stack |
| Management Platforms | Portainer, Dokploy, Coolify, CapRover, Dockge |
| CI/CD | Versioning and deployment workflows |
| Secrets | Stop using environment variables |

---

## Quick Start

```bash
# Initialize Swarm on manager (use your internal IP)
docker swarm init --advertise-addr 10.10.1.141:2377 --listen-addr 10.10.1.141:2377

# Create encrypted overlay network
docker network create \
  --opt encrypted \
  --subnet 172.240.0.0/24 \
  --gateway 172.240.0.254 \
  --attachable \
  --driver overlay \
  mynetwork

# Deploy a stack
docker stack deploy -c docker-compose.yml mystack

# Check service status
docker service ls
docker stack ps mystack
```

---

## Key Topics

### The Swarm Hierarchy

```
Swarm → Nodes → Stacks → Services → Tasks (Containers)
```

- **Swarm**: Your entire cluster
- **Nodes**: Manager and Worker hosts
- **Stacks**: Groups of related services (from compose files)
- **Services**: Manage replicas, updates, health
- **Tasks**: Individual containers

### DNS Configuration (Critical!)

90% of Swarm networking problems are DNS issues. Configure internal DNS on each node:

```ini
# /etc/systemd/resolved.conf
[Resolve]
DNS=10.10.1.122 8.8.8.8
Domains=~yourdomain.io
```

### Required Ports

Ensure these ports are open between nodes:
- **TCP 2377**: Cluster management
- **TCP/UDP 7946**: Node communication
- **TCP/UDP 4789**: Overlay network traffic

### Production Compose Example

```yaml
services:
  app:
    image: "yourregistry/app:latest"
    init: true
    deploy:
      replicas: 6
      placement:
        max_replicas_per_node: 3
      update_config:
        parallelism: 2
        delay: 10s
        failure_action: rollback
      resources:
        limits:
          cpus: '0.50'
          memory: 400M
```

---

## Repository Contents

```
docker-swarm-guide/
├── GUIDE.md                    # The complete guide
├── README.md                   # This file
├── docs/                       # Website assets
│   ├── index.html
│   ├── app.js
│   ├── styles.css
│   └── banner.webp
└── .gitignore
```

---

## Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Add your improvements
4. Submit a PR with description

### Ideas for Contributions

- [ ] Example compose files for common stacks (NGINX + Node, etc.)
- [ ] Additional monitoring configurations
- [ ] Troubleshooting guides for specific issues
- [ ] Alternative deployment platform comparisons

---

## License

MIT License - See [LICENSE](./LICENSE)

---

*Built with battle-tested knowledge by [TheDecipherist](https://thedecipherist.com?utm_source=github&utm_medium=readme&utm_campaign=docker-swarm-guide&utm_content=footer)*
