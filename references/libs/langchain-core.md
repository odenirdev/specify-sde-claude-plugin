# LangChain Core — Engineering Reference

## Best Practices

### Mental Model: LCEL and Runnables
LangChain Core (LCEL — LangChain Expression Language) is a declarative composition system using the pipe operator (`|`). Every component is a **Runnable** with a unified interface: `.invoke()`, `.stream()`, `.batch()`, `.streamLog()`.

```ts
import { ChatPromptTemplate } from "@langchain/core/prompts";
import { StringOutputParser } from "@langchain/core/output_parsers";

const chain = ChatPromptTemplate.fromTemplate("Answer: {question}")
  | llm
  | new StringOutputParser();

const result = await chain.invoke({ question: "What is LangGraph?" });
```

### Tool Definitions with `DynamicStructuredTool`
Define tools with explicit Zod schemas for type-safe input validation:

```ts
import { DynamicStructuredTool } from "@langchain/core/tools";
import { z } from "zod";

const searchTool = new DynamicStructuredTool({
  name: "search_database",
  description: "Search records in the database by query string",
  schema: z.object({
    query: z.string().describe("The search term"),
    limit: z.number().optional().default(10),
  }),
  func: async ({ query, limit }) => {
    const results = await db.search(query, limit);
    return JSON.stringify(results);
  },
});
```

### Structured Output
Use `with_structured_output` to enforce JSON Schema compliance on LLM responses:

```ts
import { z } from "zod";

const responseSchema = z.object({
  answer: z.string(),
  confidence: z.number().min(0).max(1),
  sources: z.array(z.string()),
});

const structuredLlm = llm.withStructuredOutput(responseSchema);
const result = await structuredLlm.invoke("What is the capital of France?");
// result is typed: { answer: string; confidence: number; sources: string[] }
```

### Message Types
Use the correct message types for conversation history:

```ts
import { HumanMessage, AIMessage, SystemMessage, ToolMessage } from "@langchain/core/messages";

const messages = [
  new SystemMessage("You are a helpful assistant."),
  new HumanMessage("What is the weather in São Paulo?"),
  new AIMessage({ content: "", tool_calls: [{ id: "call_1", name: "get_weather", args: { city: "São Paulo" } }] }),
  new ToolMessage({ content: "22°C, partly cloudy", tool_call_id: "call_1" }),
];
```

### Streaming
Prefer `.stream()` for real-time delivery in HTTP handlers:

```ts
const stream = await chain.stream({ question: input });
for await (const chunk of stream) {
  process.stdout.write(chunk);
}
```

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| `LLMChain` usage | Deprecated — no longer maintained | Use LCEL pipe: `prompt \| llm \| parser` |
| Tools without Zod schemas | No input validation, runtime errors | Always use `DynamicStructuredTool` with `schema` |
| Mixing `.invoke()` and `.stream()` in async context | Race conditions, incorrect output | Choose one execution mode per context |
| `any` as tool return type | Type unsafety propagates through the chain | Tools return `string` or `JSON.stringify(obj)` |
| Structured output + streaming together | Not supported simultaneously | Pick one: streaming for real-time, structured output for typed results |

---

## Review Criteria

- [ ] All tools use `DynamicStructuredTool` with Zod `schema`
- [ ] Tool `description` explains what the tool does AND when to use it (LLM reads this)
- [ ] `ToolMessage` responses include matching `tool_call_id`
- [ ] `withStructuredOutput()` used when deterministic response shape is required
- [ ] Chains use LCEL pipe (`|`) instead of legacy `LLMChain`
- [ ] Message types (`HumanMessage`, `AIMessage`, `ToolMessage`) used correctly in conversation history

---

## Trade-offs

**`DynamicStructuredTool` vs `tool()` helper**: `DynamicStructuredTool` is more explicit and compatible with all LangChain versions. The `tool()` helper (v0.2+) is terser. Prefer `DynamicStructuredTool` for clarity and LangGraph compatibility.

**Structured output vs output parser**: `withStructuredOutput()` constraints the LLM via tool call or JSON mode. Output parsers (`StructuredOutputParser`) prompt the LLM and parse text — less reliable. Prefer `withStructuredOutput()` when the model supports it.

**Streaming vs batch in Lambda**: Lambda HTTP handlers benefit from streaming via Lambda response streaming. For SQS/schedule triggers, `.invoke()` is simpler and sufficient.

---

## Implementation Notes

- Import from `@langchain/core` — never from the top-level `langchain` package directly
- `@langchain/core` is the peer dependency; model packages (`@langchain/groq`, `@langchain/openai`) depend on it
- Tool `name` must be unique within a graph — LLM uses the name to select tools
- Tool `description` directly affects LLM tool selection quality — be specific and concrete
- For LangGraph: bind tools to the model with `llm.bindTools(tools)` before passing to the agent node
