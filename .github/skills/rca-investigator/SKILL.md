---
name: rca-investigator
description: "Investigate production issues by querying Grafana Loki logs, tracing errors to source code, and producing Root Cause Analysis (RCA). Saves the RCA as a local markdown file. Use when: user provides a trace ID, error message, or asks to investigate a production incident, debug logs, create RCA, root cause analysis, post-mortem."
argument-hint: "Provide a trace ID, error message, or describe the incident to investigate"
---

# RCA Investigator

Investigate production issues end-to-end: query Grafana Loki logs, trace errors to source code, and produce a structured RCA document saved as a local markdown file.

## When to Use

- User provides a **trace ID** from Grafana/Loki logs
- User provides an **error message** and asks to trace its origin
- User asks to **investigate a production incident**
- User asks to **create an RCA** or **root cause analysis**
- User asks to **debug production logs**

## Required MCP Servers

- **grafana-loki**: For querying log data (tool: `query_logs`)

## Procedure

### Step 1 — Gather Input

Ask the user for the following (use `mcp_hitl-web-mcp_get_multiline_input` if available):

| Input | Required | Example |
|-------|----------|---------|
| Trace ID or error message | Yes | `abc123-def456` or `NullPointerException in PaymentService` |
| Service name | Helpful | `nocode-rpc`, `payment-service` |
| Time range | Helpful | `last 1h`, `2025-04-09 10:00 to 11:00` |
| Environment | Helpful | `production`, `staging` |

If the user already provided some of these in their message, do not ask again.

### Step 2 — Query Grafana Loki Logs

Use the `grafana-loki` MCP server tool `query_logs` to fetch relevant logs.

**Strategy:**
1. If a **trace ID** is provided, query directly with `{traceID="<value>"}` or filter logs containing the trace ID
2. If an **error message** is provided, search for that message pattern in logs
3. If a **service name** is provided, scope the query to that service label (e.g., `{app="<service>"}`)
4. Start with a broad query, then narrow down based on initial results
5. Look for error-level logs first: `|= "error"` or `|= "ERROR"` or `level="error"`

**Log analysis checklist:**
- Identify the **first occurrence** of the error (timeline origin)
- Collect the **full stack trace** if available
- Note **upstream/downstream service calls** that failed
- Capture **request/response patterns** around the error
- Identify any **repeated patterns** or **cascading failures**

### Step 3 — Trace Error to Source Code

Search the **current workspace** codebase to find the source of the error:

1. **Extract key identifiers** from log lines:
   - Class names, method names, function names
   - File paths mentioned in stack traces
   - Error message strings
   - HTTP endpoints or route patterns

2. **Search the codebase** using `grep_search` and `semantic_search`:
   - Search for exact error message strings
   - Search for class/method names from stack traces
   - Search for the endpoint or route that received the request
   - Follow the call chain from entry point to the error location

3. **Read and analyze the code path**:
   - Read the relevant source files
   - Trace the execution flow step by step
   - Identify the specific line(s) where the error occurs
   - Understand **why** the code fails (missing validation, null reference, race condition, etc.)

4. **Document the code path** as a numbered sequence of steps, each with:
   - File name and line range
   - What that code does in the context of the error

### Step 4 — Produce RCA Document

Fill in the RCA template from [rca-template.md](./references/rca-template.md) with findings:

- **Title**: Brief, descriptive (e.g., "Payment timeout due to unhandled database connection pool exhaustion")
- **Date**: Current date
- **Trace**: The trace ID or error message used to investigate
- **Severity**: Assess based on user impact:
  - 🔴 Critical: Service down, data loss, security breach
  - 🟠 High: Major feature broken, significant user impact
  - 🟡 Medium: Partial degradation, workaround exists
  - 🟢 Low: Minor issue, limited impact
- **TL;DR**: 2 sentences max, plain language, no jargon
- **What happened**: User-visible impact in plain language
- **Root cause**: The specific technical reason
- **Code path**: Step-by-step trace through the code
- **Evidence**: 3-5 most relevant trimmed log lines
- **Suggested fix**: Exactly 1 actionable recommendation

Present the completed RCA to the user for review before publishing.

### Step 5 — Save RCA as Markdown File

Save the completed RCA document as a local markdown file in the workspace:

1. Use the file name format: `{trace-id}-rca.md` (e.g., `abc123-def456-rca.md`)
   - If the input was an error message instead of a trace ID, slugify the error message (e.g., `nullpointerexception-in-paymentservice-rca.md`)
2. Create the file in the workspace root directory using `create_file`
3. Write the full RCA content in Markdown format
4. Show the user the file path and a preview of the content

### Step 6 — Summary

After saving the RCA file:
1. Share the file path of the saved RCA markdown
2. Summarize what was found and the recommended fix
3. Ask if the user wants to:
   - Modify the RCA content
   - Investigate further with different queries

## Tips

- When log output is large, focus on the **first error occurrence** and work forward
- Stack traces are the most valuable — always capture the full trace
- If the codebase doesn't contain the erroring code, note that the error may originate from a dependency or different repository
- Keep the RCA concise — the template enforces brevity by design
