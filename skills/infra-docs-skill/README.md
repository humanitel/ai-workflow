# infra-docs

> **Internal packaged skill** — Fuse Finance infrastructure documentation

A skill for searching and answering questions from the Fuse Finance infrastructure documentation. Requires access to internal company resources.

## What it does

Answers questions about infrastructure, DevOps, deployments, cloud setup, CI/CD, networking, Kubernetes, Docker, monitoring, logging, and internal platform/tooling by querying the infra docs site.

## Official source

- **Docs site:** https://docs.infra.fusefinancelab.com
- **GitHub (internal):** https://github.com/FuseFinance/infra-docs

## Installation

Requires the setup script which fetches and indexes the documentation locally:

```bash
bash <skill-path>/scripts/setup-docs.sh
```

> **Note:** This skill is specific to Fuse Finance internal infrastructure. It will not be useful outside of that context.
