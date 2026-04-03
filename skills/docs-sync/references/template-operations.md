# Operations — [Project Name]

> Deployment, configuration, and operational procedures. Reference for on-call and deployment workflows.

---

## Environments

| Environment | Purpose | Access |
|---|---|---|
| `development` | Local development | Developer machines |
| `staging` | Pre-production validation | CI + team |
| `production` | Live system | Restricted |

---

## Configuration

All configuration is provided via environment variables. No config files committed with secrets.

### Required Environment Variables

| Variable | Description | Example |
|---|---|---|
| `DATABASE_URL` | PostgreSQL connection string | `postgres://user:pass@host/db` |
| `[SERVICE]_API_KEY` | API key for [service] | — |
| `[VAR_NAME]` | [Description] | [Example value, not real] |

### Optional Variables

| Variable | Default | Description |
|---|---|---|
| `LOG_LEVEL` | `info` | Logging verbosity (`debug`, `info`, `warn`, `error`) |
| `PORT` | `3000` | HTTP server port |

---

## Deployment

### Standard Deploy
```bash
# Build
[build command]

# Run migrations
[migration command]

# Start
[start command]
```

### Database Migrations
```bash
# Apply pending migrations (production)
[prisma migrate deploy / flyway migrate / etc.]

# Check migration status
[status command]
```

> **Never** run `migrate dev` in production — it generates new migrations.

### Rollback
1. [Step 1 — e.g., redeploy previous image]
2. [Step 2 — if DB migration was destructive, restore from backup]

---

## Health Checks

| Endpoint | Purpose | Expected response |
|---|---|---|
| `GET /health` | Liveness check | `200 OK` |
| `GET /health/ready` | Readiness check | `200 OK` when DB is connected |

---

## Runbooks

### [Incident: Name]
**Symptoms**: [What the alert or user report looks like]
**Diagnosis**:
1. Check [log/metric]
2. If [condition], do [action]
3. If [other condition], escalate to [team]

**Resolution**: [Steps to resolve]
**Prevention**: [How to prevent recurrence]

---

## Alerts

| Alert | Condition | Action |
|---|---|---|
| [Alert name] | [Threshold] | [First response] |

---

## Backup & Recovery

- **Database**: [Backup frequency, retention, restore procedure]
- **Recovery time objective (RTO)**: [Target]
- **Recovery point objective (RPO)**: [Target]
