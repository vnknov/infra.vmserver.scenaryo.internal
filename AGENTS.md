# AGENTS.md

Guidance for AI agents working in this repository.

## Repository Overview

This repository contains infrastructure configuration for `vmserver.scenaryo.internal`.

Current contents:

- `caddy/docker-compose.yml`: Docker Compose service for the Caddy reverse proxy.
- `caddy/Caddyfile`: Caddy routes for Obsidian and n8n services.
- `certs/`: local certificates and private keys, intentionally ignored by git.

## Working Guidelines

- Keep changes minimal and infrastructure-focused.
- Do not commit certificates, private keys, tokens, passwords, or other secrets.
- Treat `certs/` as local-only runtime material.
- Preserve existing service names and ports unless the user explicitly asks to change them.
- Prefer editing existing Compose and Caddy configuration over introducing new tooling.
- Avoid adding backward-compatibility aliases or duplicate routes unless there is a concrete need.

## Caddy Notes

- The Caddy container mounts `./Caddyfile` to `/etc/caddy/Caddyfile:ro`.
- TLS certificates are expected at `/certs/server.crt` and `/certs/server.key` inside the container.
- The configured certificate files come from `../certs/local-ca/vmserver.crt` and `../certs/local-ca/vmserver.key`.
- Use explicit schemes in `reverse_proxy` targets when the upstream protocol matters.
- Be careful with `tls_insecure_skip_verify`; keep it only for known internal upstreams that require it.

## Certificate Inventory

The actual deployment environment currently has these local CA files under `certs/local-ca/`:

- `root-ca.crt`: root CA certificate.
- `root-ca.key`: root CA private key, mode `600`.
- `root-ca.srl`: root CA serial file.
- `vmserver.cnf`: OpenSSL configuration for the vmserver certificate.
- `vmserver.crt`: vmserver certificate mounted into Caddy as `/certs/server.crt`.
- `vmserver.csr`: vmserver certificate signing request.
- `vmserver.key`: vmserver private key mounted into Caddy as `/certs/server.key`, mode `600`.

These files are runtime material only and must remain outside git.

## Docker Compose Notes

- The Caddy service joins an external Docker network named `proxy`.
- Do not remove the external network declaration unless replacing the deployment topology.
- Named volumes `caddy_data` and `caddy_config` store Caddy runtime state and configuration.

## Validation

When possible, validate changes before finishing:

- From `caddy/`, run `docker compose config` to validate Compose syntax.
- If Docker and the Caddy image are available, run `docker compose exec caddy caddy validate --config /etc/caddy/Caddyfile` against a running deployment.
- If the service is not running, note that runtime validation could not be performed.

## Git Hygiene

- Do not revert unrelated user changes.
- Check diffs before summarizing edits.
- Stage or commit only when the user explicitly asks.
