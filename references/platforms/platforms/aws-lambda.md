# AWS Lambda (Node.js) — Engineering Reference

## Best Practices

### Mental Model: Cold Start vs Warm Invocation
- **Cold start**: Lambda allocates a new execution environment (~500–1000ms for Node.js). Happens after idle periods or scale-out.
- **Warm invocation**: handler re-executes in the existing environment. Module-scope code is NOT re-run.
- **Rule**: heavy initialization (DB connections, graph compilation, HTTP clients) must happen at module scope, not inside the handler.

```ts
// CORRECT: initialized once at module scope
import { createGraph } from "./graph";
const graph = createGraph(); // compiled once

export const handler = async (event: APIGatewayProxyEventV2) => {
  const result = await graph.invoke({ ... });
  return { statusCode: 200, body: JSON.stringify(result) };
};
```

### Memory Configuration
Memory directly controls CPU allocation and network throughput. More memory = faster execution:

- Minimum for LangGraph/LangChain workloads: **1024 MB**
- Recommended starting point: **1536–2048 MB** for LLM-heavy handlers
- Use [AWS Lambda Power Tuning](https://github.com/alexcasalboni/aws-lambda-power-tuning) to find optimal memory/cost ratio

```yaml
# serverless.yml
provider:
  memorySize: 1536
  timeout: 60
```

### Timeout Configuration
Set timeout to 3–5x the expected p95 execution time. LLM calls can take 5–30s:

```yaml
functions:
  agentInvoke:
    timeout: 120  # 2 minutes for LLM-heavy agents
```

### Bundling with esbuild (Cold Start Optimization)
Tree-shaking and minification reduce bundle size and cold start time:

```yaml
custom:
  esbuild:
    bundle: true
    minify: true
    sourcemap: false    # disable in prod for smaller bundle
    target: node20
    platform: node
    exclude:
      - "@aws-sdk/*"    # already available in Lambda runtime (Node 18+)
    external:
      - "pg-native"     # native bindings not available in Lambda
```

### Environment Variables and Secrets
```ts
// Always validate at startup
import { z } from "zod";

const envSchema = z.object({
  GROQ_API_KEY: z.string().min(1),
  DATABASE_URL: z.string().url(),
  STAGE: z.enum(["dev", "staging", "prod"]),
});

const env = envSchema.parse(process.env); // throws at cold start if invalid
```

Use SSM Parameter Store or Secrets Manager for sensitive values — never plaintext in `serverless.yml`.

### Lambda Layers for Shared Dependencies
For large shared dependencies (e.g., chromium, ML models), use Lambda Layers to avoid bundle size limits (250MB unzipped):

```yaml
layers:
  sharedDeps:
    path: layers/shared-deps
    compatibleRuntimes:
      - nodejs20.x

functions:
  agentInvoke:
    layers:
      - !Ref SharedDepsLambdaLayer
```

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Heavy initialization inside handler | Repeated on every warm invocation, high latency | Move to module scope |
| Wildcard imports (`import * as _`) | Prevents tree-shaking, bloats bundle | Import named exports only |
| `memorySize < 512 MB` for LLM workloads | CPU-bottlenecked, slower + more expensive | Start at 1024–1536 MB |
| `timeout` close to average duration | Edge-case timeouts cause silent failures | Set to 3–5x expected p95 |
| Secrets as plaintext in environment | Exposed in CloudFormation, logs | Reference via `${ssm:...}` |
| Bundle includes `@aws-sdk/*` | Unnecessarily large bundle (+30MB) | Add to `exclude` in esbuild config |
| Catching and swallowing errors in handler | Silent failures, no alerting | Always rethrow or return error response with status 5xx |

---

## Review Criteria

- [ ] Heavy initialization at module scope (outside handler function)
- [ ] `memorySize` ≥ 1024 for LLM workloads
- [ ] `timeout` set to 3–5x expected p95 execution time
- [ ] `@aws-sdk/*` excluded from esbuild bundle
- [ ] Environment variables validated with Zod at startup
- [ ] Secrets referenced via SSM, not plaintext
- [ ] Errors rethrown or returned with 5xx status — never swallowed
- [ ] No wildcard imports in Lambda handler files

---

## Trade-offs

**Provisioned Concurrency vs cold start mitigation**: Provisioned Concurrency eliminates cold starts but incurs constant cost. For LangGraph agents with user-facing latency requirements, it may be justified in prod. In dev/staging, cold starts are acceptable.

**Lambda vs ECS/Fargate for LLM agents**: Lambda is simpler operationally and cheaper at low throughput. ECS is better for sustained high-throughput, streaming responses, or workloads that require persistent connections (e.g., WebSocket). Lambda with response streaming is a middle ground.

**Single handler vs per-endpoint functions**: Single handler (Express/Fastify adapter) is easier to migrate existing code. Per-endpoint functions allow fine-grained memory/timeout/IAM tuning — preferred for new LangGraph agent APIs.

---

## Implementation Notes

- Node.js 20.x runtime on Lambda supports native `fetch` and top-level `await`
- Max bundle size: 250MB unzipped (50MB zipped for direct upload; use S3 for larger)
- Max memory: 10,240 MB
- Max timeout: 900s (15 minutes)
- CloudWatch Logs: Lambda streams stdout/stderr automatically
- Monitor memory usage via CloudWatch `max_memory_used` metric — if consistently near limit, increase
- For PostgreSQL in Lambda: use connection pooling (PgBouncer or RDS Proxy) to avoid connection exhaustion
