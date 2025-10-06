# Case Study 001: Resolving Dev Environment Issues

**Date:** 2025-09-30  
**Category:** Investigation  
**Tags:** #Docker, #WSL, #MySQL, #Nginx, #Django, #TLS  
**Status:** Completed  
**Severity:** S2 – High (blocked full local environment startup)

## 1. Summary
Local development environment repeatedly failed to start due to permission, configuration, and data consistency issues across Docker, WSL, and MySQL. The root cause was an interplay between Windows NTFS permission constraints and Linux-based TLS requirements within containers. Environment was stabilized after migrating volumes, reinstalling certificates within WSL, and reinitializing the database.

## 2. Context
This case involved a full-stack local environment composed of Dockerized services (MySQL, Django backend, Nginx, Celery, and React frontends) running under **Windows Subsystem for Linux (WSL 2)**.  
The setup relied on local TLS certificates generated via `mkcert`, containerized MySQL with volume bindings, and local service ports (`8013–8123`).

The initial goal was to restore a complete development environment for backend, admin, and client UIs after cloning the repository.

## 3. Problem Statement
Multiple containers exited immediately after `docker compose up`.  
MySQL refused to start due to permission errors.  
Nginx crashed on certificate initialization.  
Django returned **502 Bad Gateway** from the admin interface, preventing login and database migrations.

These issues completely blocked local development and debugging.

## 4. Investigation

### Hypotheses
- **H1:** MySQL TLS initialization failed due to `mkcert` certificate misconfiguration.  
- **H2:** File system permissions mismatched between Windows NTFS and Linux volumes.  
- **H3:** Django migration state corrupted after partial container restarts.

### Steps Taken
1. Inspected MySQL logs showing permission failures on `ca.pem` and `private_key.pem`.  
2. Verified `mkcert` installation path; discovered it was installed on Windows host, not inside WSL.  
3. Reinstalled `mkcert` within WSL — partially fixed certificate generation errors.  
4. Noticed `private_key.pem` still failed due to NTFS permission limitations on `/var/lib/mysql`.  
5. Switched MySQL data directory to use a **named Docker volume**, not a bind mount.  
6. Restarted environment — MySQL initialized successfully.  
7. Re-ran Django migrations; identified residual `django_migrations` table conflict.  
8. Dropped the table manually, then successfully re-ran all migrations.  
9. Increased Docker memory allocation from 12GB → 20GB to stabilize concurrent containers.  
10. Retried environment startup fully within WSL context — all core services came online.

### Key Evidence
**MySQL error logs:**
```
mysqld: Cannot change permissions of the file 'ca.pem' (OS errno 1 - Operation not permitted)
[ERROR] Could not set file permission for private_key.pem
[ERROR] The designated data directory /var/lib/mysql/ is unusable
```

**Diagnosis:**
- NTFS filesystem prevents Unix permission changes (`chmod`, `chown`).
- Docker bind mounts pointing to Windows paths inherit NTFS permission model.

## 5. Decision
Use **Docker-managed named volumes** for MySQL storage instead of host-bound directories to ensure Linux-compatible permissions.  
Reinstall all certificates and developer tooling within WSL rather than Windows.  
Manually reset database migration state to ensure Django schema consistency.

## 6. Risk & Tradeoff Analysis

**Option 1 – Continue using bind mounts**
- *Benefit:* Easy file inspection via Windows Explorer.  
- *Risk:* MySQL crash on startup due to NTFS permission model.  
- *Decision:* ❌ Rejected — inconsistent and unstable.

**Option 2 – Use named Docker volumes**
- *Benefit:* Proper Unix permission control, persistent data, stable MySQL startup.  
- *Risk:* Slightly harder to inspect data directly.  
- *Decision:* ✅ Accepted — resolved all permission-related crashes.

**Option 3 – Move development to native Linux**
- *Benefit:* Avoids WSL abstraction issues entirely.  
- *Risk:* Requires dual-boot or VM; not portable for all contributors.  
- *Decision:* Deferred — not required after stable WSL configuration achieved.

## 7. Mitigation & Handoff
- Updated `docker-compose.yaml` to use named volumes:

```yaml
  volumes:
    - mysql_data:/var/lib/mysql
  volumes:
    mysql_data:`
``` 

- Reinstalled mkcert within WSL.
- Increased WSL Docker memory allocation.
- Cleaned and reseeded database via bin/seed.
- Documented setup differences between Windows and WSL.

## 8. Outcome & Learnings

### Outcome:

- Backend, database, and admin UI successfully restored.
- Frontend containers (Storybook, client portal) remained in partial setup pending further debugging.
- All TLS, migration, and login errors resolved.

### Key Learnings:

- NTFS file systems do not fully support Unix file permissions required by MySQL TLS initialization.

- mkcert must be installed in the same environment as the running containers.

- Dropping stale migration tables can fix half-initialized Django databases.

- Docker resource allocation under WSL directly impacts container stability.

## 9. Attachments

N/A

## 10. Reflection

This investigation clarified the complexity of hybrid Windows–Linux dev setups. It emphasized the importance of matching certificate context, isolating platform-specific quirks early, and preferring container-native volumes for consistency. The experience reinforced environment reproducibility as a first-class engineering concern.