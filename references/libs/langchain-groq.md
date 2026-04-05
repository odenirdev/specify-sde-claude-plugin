# LangChain Groq (ChatGroq) — Engineering Reference

## Best Practices

### Initialization
Always initialize from environment variables. Never hardcode API keys:

```ts
import { ChatGroq } from "@langchain/groq";

const llm = new ChatGroq({
  model: "llama-3.3-70b-versatile",
  apiKey: process.env.GROQ_API_KEY,
  temperature: 0.7,
  maxTokens: 4096,
});
```

Initialize at module scope (outside Lambda handler) to reuse across warm invocations.

### Model Selection
| Model | Use Case |
|---|---|
| `llama-3.3-70b-versatile` | General reasoning, ReAct agents (default recommendation) |
| `llama3-70b-8192` | Fast reasoning with 8K context |
| `mixtral-8x7b-32768` | Long context (32K), document processing |
| `gemma2-9b-it` | Lightweight tasks, low latency |

### Temperature Guidelines
- `0.0–0.3` — Deterministic tasks: code generation, structured output, SQL
- `0.5–0.7` — Balanced: reasoning, ReAct agents, Q&A
- `1.0` — Default balanced creativity
- `1.5+` — Creative: brainstorming, story generation

### Binding Tools for ReAct Agents
```ts
const llmWithTools = llm.bindTools(tools);

async function callModel(state: typeof MessagesAnnotation.State) {
  const response = await llmWithTools.invoke(state.messages);
  return { messages: [response] };
}
```

### Structured Output
```ts
import { z } from "zod";

const schema = z.object({
  category: z.enum(["bug", "feature", "question"]),
  priority: z.number().min(1).max(5),
  summary: z.string(),
});

const structuredLlm = llm.withStructuredOutput(schema);
const result = await structuredLlm.invoke("Classify this issue: app crashes on login");
```

### Streaming
```ts
const stream = await llm.stream([new HumanMessage("Explain LangGraph in 3 sentences")]);
for await (const chunk of stream) {
  process.stdout.write(chunk.content as string);
}
```

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Hardcoded `apiKey` in source | Security exposure | Always use `process.env.GROQ_API_KEY` |
| Structured output + streaming together | Not currently supported by Groq | Use one or the other per invocation |
| Tool use + streaming together | Not currently supported by Groq | Use `.invoke()` for tool-enabled calls |
| Temperature > 1.5 for factual tasks | Inconsistent, hallucination-prone responses | Use 0.0–0.5 for deterministic tasks |
| Initializing `ChatGroq` inside Lambda handler | New instance per cold start, no reuse | Initialize at module scope |
| Not setting `maxTokens` | Runaway responses, unexpected cost | Always set `maxTokens` explicitly |

---

## Review Criteria

- [ ] `apiKey` loaded from environment variable (`process.env.GROQ_API_KEY`)
- [ ] `ChatGroq` instance initialized at module scope (outside handler)
- [ ] `temperature` set explicitly and appropriate for the task
- [ ] `maxTokens` set to limit response length
- [ ] Streaming and structured output not used in the same call
- [ ] Tool use and streaming not used in the same call

---

## Trade-offs

**Groq vs OpenAI**: Groq offers significantly lower latency due to LPU hardware. OpenAI has more model options and stronger tool use reliability with GPT-4. For speed-critical agents, Groq is the better default; for complex tool orchestration, test both.

**`withStructuredOutput` vs prompt-based JSON**: `withStructuredOutput` uses Groq's structured output mode (JSON Schema enforcement via grammar sampling) — more reliable. Prompt-based JSON parsing is a fallback for models that don't support it.

**`llama-3.3-70b-versatile` vs `mixtral-8x7b-32768`**: LLaMA 3.3 70B is stronger for reasoning and tool use. Mixtral has a longer context window (32K vs 128K for LLaMA 3.3) — use Mixtral when document context exceeds 8K tokens.

---

## Implementation Notes

- Set `GROQ_API_KEY` in `.env` locally and SSM Parameter Store in production
- Groq rate limits are per model and per API key — monitor via Groq console
- `ChatGroq` is drop-in compatible with `ChatOpenAI` interface — swap by changing the class
- For Lambda: `@langchain/groq` adds ~2MB to bundle; esbuild tree-shaking keeps it manageable
- Check current model list: https://console.groq.com/docs/models
