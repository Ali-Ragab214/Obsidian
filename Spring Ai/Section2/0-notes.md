![[Pasted image 20260724012646.png]]![[Pasted image 20260723151055.png]]
what are message roles in llm ? 
![[Pasted image 20260723151144.png]]
user ,system ,assistant,function
![[Pasted image 20260723151909.png]]

```
ChatClient  --بيستخدم جواه-->  ChatModel  --بيكلم-->  LLM Provider
```
# Message Roles in LLM APIs

## `system`

Sets the model's behavior, personality, and constraints **before** the conversation starts. It's instructions to the model itself, not something the user said.

```json
{ "role": "system", "content": "You are a support triage agent. Always respond in JSON." }
```

## `user`

The actual input from the person (or from your application on behalf of the person) — the question, request, or data being sent to the model.

```json
{ "role": "user", "content": "Why is my order still pending?" }
```

## `assistant`

The model's own previous responses. You include these when sending conversation history back, so the model has context of what it already said.

```json
{ "role": "assistant", "content": "Let me check that for you..." }
```

## `function` / `tool`

The **result** of a tool/function call the model requested — not something a human typed. When the model asks to call `get_order_status(orderId)`, your code executes it, then you feed the result back with this role so the model can use it in its next response.

```json
{ "role": "tool", "tool_call_id": "call_123", "content": "{\"status\": \"pending\", \"eta\": \"2 days\"}" }
```

(Anthropic's API calls this `tool_result` content inside a `user` message rather than a separate role, but OpenAI uses a distinct `function`/`tool` role — naming differs slightly by provider, concept is the same.)

---

## Quick mental model

```
system   → "here's how you should behave" (set once, rarely repeated)
user     → "here's what I'm asking"
assistant → "here's what you said before"
tool     → "here's what the tool you called returned"
```

Every multi-turn conversation is just this array growing: `[system, user, assistant, user, assistant, tool, assistant, ...]` — the model re-reads the whole thing each time to generate the next `assistant` message.