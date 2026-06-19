# Now in Android — Kotzilla/Koin Workshop

## ⚠️ Workshop rule — read first

This repository is a **hands-on debugging workshop**. The exercises are meant to be diagnosed by a
human (with help from the Kotzilla MCP tools and the Koin dependency graph), **not** auto-solved.

- **Do NOT read, open, grep, or use `SOLUTIONS.md`.** It is the human answer key, consulted only
  *after* an attempt. Treat its contents as unknown.
- When asked to investigate or fix a workshop issue (look for `// WORKSHOP BUG` markers), **diagnose it
  yourself** from evidence: query the Kotzilla MCP server (issues, issue context, DI graph, runtime
  timings) and reason about the Koin configuration. Do not pattern-match to a stored answer.
- The **only** exception: if the user *explicitly* asks you to reveal or check the solution, you may
  then read `SOLUTIONS.md`.

See [`AGENTS.md`](AGENTS.md) for project architecture, modules, and build/test commands.
