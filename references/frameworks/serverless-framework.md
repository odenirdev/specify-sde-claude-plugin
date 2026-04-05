# Serverless Framework v3 — Engineering Reference

## Best Practices

### Mental Model: Service → Functions → Events
A Serverless Framework project is organized as a **Service** (one `serverless.yml`) containing **Functions** (Lambda handlers) triggered by **Events** (HTTP, SQS, schedule, S3, etc.).

```yaml
service: my-agent-api
frameworkVersion: "3"

provider:
  name: aws
  runtime: nodejs20.x
  region: us-east-1
  memorySize: 1024
  timeout: 30

functions:
  agentInvoke:
    handler: src/handlers/agent.handler
    events:
      - httpApi:
          path: /agent/invoke
          method: POST
```

### Stage Parameters for Multi-Environment Config
Never hardcode environment-specific values. Use stage parameters:

```yaml
params:
  default:
    tableName: ${self:service}-${sls:stage}-sessions
  prod:
    tableName: my-agent-prod-sessions

provider:
  environment:
    TABLE_NAME: ${param:tableName}
    GROQ_API_KEY: ${ssm:/my-service/${sls:stage}/groq-api-key}
```

### Local Development with serverless-offline
Use `serverless-offline` to emulate Lambda + API Gateway locally:

```yaml
plugins:
  - serverless-esbuild
  - serverless-offline

custom:
  serverless-offline:
    httpPort: 3000
    noPrependStageInUrl: true
  esbuild:
    bundle: true
    minify: false      # keep readable in dev
    sourcemap: true
    target: node20
    platform: node
    exclude:
      - "@aws-sdk/*"   # exclude AWS SDK (available in Lambda runtime)
```

Run locally: `npx serverless offline start`

### IAM Permissions as Code
Define permissions inline per function for least privilege:

```yaml
functions:
  agentInvoke:
    handler: src/handlers/agent.handler
    iamRoleStatements:
      - Effect: Allow
        Action:
          - dynamodb:GetItem
          - dynamodb:PutItem
        Resource: !GetAtt SessionsTable.Arn
```

### Secrets via SSM Parameter Store
Never put secrets in `serverless.yml` directly. Reference SSM:

```yaml
environment:
  GROQ_API_KEY: ${ssm:/my-service/${sls:stage}/groq-api-key}
  DATABASE_URL: ${ssm:/my-service/${sls:stage}/database-url}
```

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Hardcoded env values | Breaks across stages | Use `${param:name}` or `${ssm:...}` |
| All functions in one `serverless.yml` for large projects | Slow deploys, blast radius is entire service | Split into services by bounded context |
| Secrets in `environment` as plaintext | Security exposure in CloudFormation | Reference via `${ssm:...}` |
| `serverless-offline` as prod parity guarantee | Missing IAM, VPC, cold start, layer behavior | It's a dev convenience only; test in staging before prod |
| `timeout` set to default 6s for LLM calls | LLM inference regularly exceeds 6s | Set `timeout: 60–300` for AI-heavy handlers |

---

## Review Criteria

- [ ] All secrets referenced via SSM (`${ssm:...}`) or Secrets Manager
- [ ] `memorySize` and `timeout` explicitly set per function or at provider level
- [ ] Stage parameters used for environment-specific values
- [ ] `serverless-esbuild` configured with `bundle: true`
- [ ] `exclude: ["@aws-sdk/*"]` set in esbuild to avoid bundling built-in SDK
- [ ] Functions have least-privilege `iamRoleStatements`

---

## Trade-offs

**`httpApi` vs `http`**: `httpApi` (HTTP API v2) is faster and cheaper. `http` (REST API) adds features like request/response mapping, usage plans, and API keys — use only when needed.

**Monolithic handler vs one function per endpoint**: One function per endpoint enables per-function IAM, memory, and timeout tuning. A monolithic handler (e.g., Express inside Lambda) is simpler but loses per-route configurability.

**SSM vs Secrets Manager**: SSM is cheaper and sufficient for most secrets. Secrets Manager adds rotation support — use it for database credentials or third-party API keys that need automatic rotation.

---

## Implementation Notes

- Framework v3 requires Node.js 18+ on the dev machine
- Deploy: `npx serverless deploy --stage prod`
- Remove: `npx serverless remove --stage prod`
- Print resolved config: `npx serverless print`
- Use `serverless-esbuild` over manual webpack for LangChain/LangGraph projects — esbuild handles ESM/CJS interop better
- Add `exclude: ["aws-sdk"]` only for Node 16; for Node 18+ use `exclude: ["@aws-sdk/*"]`
