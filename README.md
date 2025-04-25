# How Cursor Manages the Context Window  
*(Updated April 2025)*

Cursor rebuilds the **prompt sent to the LLM** on every message.  
This document explains what goes into that prompt, how it’s optimized, and how to minimize unnecessary token usage.

## 1. How Prompt Rebuilding Works

Language‑model APIs are **stateless**: they do not remember anything between calls.  
Cursor must resend everything the model needs — system rules, recent chat turns, tool definitions, code snippets, and your input — with **each request**.

> **Note:** Cursor applies **heavy prompt caching** internally to optimize for unchanged static sections (such as system instructions and tool schemas).

## 2. What Goes Into a Cursor Prompt

| Prompt Component | Purpose |
|------------------|---------|
| **System instructions** | Fixed rules defining assistant behavior, formatting, tone, etc. |
| **Model-specific directives** | Adjustments based on the model in use (e.g., GPT‑4 vs Claude). |
| **Your latest message** | The new text you just typed. |
| **Selected chat history** | Enough recent conversation to provide continuity; older messages may be truncated or summarized after certain limits. |
| **MCP tool metadata** | Name, description, and input schema for every enabled tool (limited to 40 tools maximum). |
| **Code/file context** | Snippets or search results relevant to your input, sometimes drawn from prior conversation turns. |

*Exact token usage varies based on project size, number of enabled tools, and model context limits. No fixed numbers are provided.*

## 3. Static vs Dynamic Parts

- **Static (heavily cached)**  
  • System instructions  
  • Tool definitions  

- **Dynamic (rebuilt or retrieved each turn)**  
  • New user input  
  • Selected chat history (summarized if over size thresholds)  
  • Relevant code snippets (may reuse cached content if already part of conversation)

## 4. Token Budgets & Best Practices

1. **Minimize enabled tools**  
   MCP metadata can add significant weight. Disable tools you aren't actively using.

2. **Focus your questions**  
   Narrow questions require less history and code to be included.

3. **Reference code precisely**  
   Request snippets, not entire files, to keep code context compact.

4. **Use search actions strategically**  
   Target specific definitions instead of asking for broad project-wide context.

## 5. Model Differences

Cursor applies the **same prompt construction strategy** for all backends:

- Full prompt rebuilt every turn.
- Static sections cached when possible.
- Trimmed dynamically based on each model’s maximum context window (e.g., 8k, 32k, 128k, 200k+ tokens).

## 6. Recent Improvements (as of 2025)

- **Tool limit enforcement:** Only the first 40 tools are included per request.
- **Tool metadata optional:** If no external MCP tools are enabled, Cursor skips this section entirely.
- **Smarter history management:** Long conversations are summarized after threshold limits are exceeded.

## 7. Key Takeaways

- Cursor sends a complete prompt to the model **on every user message**.
- Static elements are **cached** where possible; dynamic elements are **rebuilt or recalled** each turn.
- **Prompt size directly impacts model performance.**
- Managing tools, code references, and conversation focus **keeps the context efficient**.

## 8. Further Reading

- Cursor Docs – [Rules (Context)](https://docs.cursor.sh/rules)  
- Cursor Docs – [Model Context Protocol (MCP)](https://docs.cursor.sh/model-context-protocol/overview)  
- Shankar, S. *How Cursor Works* (2025)  
- Cursor Community – [Discussions on context trimming and tool management](https://community.cursor.sh)

*(All references verified as of April 2025.)*
