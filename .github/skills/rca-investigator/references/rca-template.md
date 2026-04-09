# RCA Document Template

Use this template to produce the RCA. Replace all `{placeholders}` with actual values from the investigation.

---

```markdown
# RCA: {brief title}

**Date**: {date} | **Trace**: `{trace ID or error message}`
**Severity**: 🔴 Critical / 🟠 High / 🟡 Medium / 🟢 Low

---

## TL;DR
{2 sentences max. First sentence: what broke and why, in plain language anyone can understand. Second sentence: what was done to fix it. No file names or technical jargon.}

---

## What happened
{Max 2 sentences. Plain language. What broke, for whom, user-visible impact.}

## Root cause
{Max 2 sentences. The specific technical reason.}

## Code path
**Step 1 — {description}**
[filename.rb](path/to/file.rb) (line X-Y) — {what this code does in context of the error}

**Step 2 — {description}**
[filename.rb](path/to/file.rb) (line X-Y) — {what this code does}

## Evidence
{3-5 most relevant log lines, trimmed}

## Suggested fix
{Exactly 1 recommendation sentence. Reference the file and line from Code path above.}
```
