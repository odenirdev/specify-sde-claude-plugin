# LangGraph — Engineering Reference

## Best Practices

### Mental Model: StateGraph
LangGraph models agents as directed graphs with three primitives:
- **State**: shared typed object (e.g., `{ messages: BaseMessage[] }`) passed between nodes
- **Nodes**: functions that read state and return partial updates
- **Edges**: transitions between nodes; can be conditional

The graph maintains and updates state throughout execution. Compile the graph once and invoke with a thread config.

### ReAct Agent Loop
The canonical ReAct pattern uses two nodes in a cycle:

```ts
import { StateGraph, MessagesAnnotation } from "@langchain/langgraph";
import { ToolNode } from "@langchain/langgraph/prebuilt";

const graph = new StateGraph(MessagesAnnotation)
  .addNode("agent", callModel)
  .addNode("tools", new ToolNode(tools))
  .addEdge("__start__", "agent")
  .addConditionalEdges("agent", shouldContinue)
  .addEdge("tools", "agent")
  .compile({ checkpointer });
```

The `shouldContinue` function routes to `"tools"` when there are tool calls, or `"__end__"` otherwise.

### Checkpointers for Persistence
Always compile with a checkpointer for multi-turn conversations and fault tolerance:

```ts
import { InMemorySaver } from "@langchain/langgraph";
import { PostgresSaver } from "@langchain/langgraph-checkpoint-postgres";

// Development
const checkpointer = new InMemorySaver();

// Production
const checkpointer = PostgresSaver.fromConnString(process.env.DATABASE_URL!);
await checkpointer.setup();

const graph = workflow.compile({ checkpointer });
```

### Thread IDs for Conversation History
Always pass a unique `thread_id` per conversation to maintain state:

```ts
const config = { configurable: { thread_id: "user-session-123" } };
const result = await graph.invoke({ messages: [new HumanMessage(input)] }, config);
```

### ToolMessage Handling
Tool results must be returned as `ToolMessage` to close the tool call loop:

```ts
import { ToolMessage } from "@langchain/core/messages";

return new ToolMessage({
  content: JSON.stringify(result),
  tool_call_id: toolCall.id,
});
```

---

## Anti-Patterns

| Pattern | Problem | Fix |
|---|---|---|
| Non-serializable objects in state | Checkpointer fails to persist state | Store primitives, strings, or plain objects |
| Deeply nested graphs without reason | Hard to debug, hard to trace execution | Keep ReAct loop flat: 2–3 nodes maximum |
| Missing checkpointer in multi-turn flows | State lost between invocations | Always compile with checkpointer |
| No thread_id in config | All users share the same conversation history | Generate unique thread_id per session |
| Direct state mutation in nodes | Causes race conditions in parallel execution | Always return state updates, never mutate |

---

## Review Criteria

- [ ] State type is explicitly typed (use `Annotation` or `MessagesAnnotation`)
- [ ] `thread_id` is defined per conversation in the config
- [ ] Checkpointer is configured (InMemorySaver for dev, PostgresSaver for prod)
- [ ] Conditional edge has explicit `"__end__"` case to terminate the loop
- [ ] Tool results returned as `ToolMessage` with matching `tool_call_id`
- [ ] Graph compiled once at module initialization, not per request

---

## Trade-offs

**InMemorySaver vs PostgresSaver**: InMemorySaver is zero-setup and ideal for development or single-invocation Lambda executions. PostgresSaver persists state across Lambda cold starts and concurrent executions — required for production multi-turn agents.

**Single-agent vs Multi-agent graph**: Single-agent graphs are easier to reason about. Multi-agent (supervisor + subgraphs) adds coordination overhead. Only split when tool count exceeds ~10 or when domains are truly independent.

**Streaming vs batch invocation**: Use `.stream()` for real-time token delivery in HTTP contexts; use `.invoke()` for Lambda async handlers where the full response is needed before returning.

---

## Implementation Notes

- Import from `@langchain/langgraph` for graph primitives; `@langchain/langgraph/prebuilt` for `ToolNode` and `createReactAgent`
- `createReactAgent` is a shortcut for simple ReAct loops — use `StateGraph` directly when you need custom state or conditional logic
- For Lambda: compile the graph outside the handler (module scope) to reuse across warm invocations
- Time-travel debugging: use `graph.getState(config)` to inspect state at any checkpoint
