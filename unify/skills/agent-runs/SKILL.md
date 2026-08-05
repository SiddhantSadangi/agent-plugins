---
name: agent-runs
description: Run Unify agent tasks with the core run_agent → poll_agent → answer_question → read_agent_results loop. Use before starting any Unify run - covers writing effective task briefs, polling cadence, relaying clarification questions, reading results, and error recovery.
---

# Running the Unify agent

Every Unify task is one lifecycle: start, poll, (maybe) answer questions, read.

## 1. Start: `run_agent({ prompt })`

Returns `{ runId, threadId, threadLink }` immediately. Write the prompt as a
brief to a skilled GTM operator who cannot see your conversation. Include:

- **Entity type and deliverable**: companies or people; inline answer or a DataTable ("return the results as a DataTable" for anything more than a couple of records).
- **Hard filters vs. intent**: exact constraints (industry, headcount, geography, funding stage, titles) separately from fuzzy persona intent ("technical buyers of data-infrastructure tooling"). Geography defaults to US, so say otherwise explicitly.
- **Scale**: target count ("20 companies", "2–3 contacts per company").
- **Budget**: if spend matters: "prefer free sources (Universal Data, CRM)" or "cap credit spend at N".
- **Approval semantics** for outreach: whether to stop at previews or the user has approved enrolling/sending. Never authorize sending unless your user explicitly did.

Pre-answer the obvious clarification dimensions above; the agent pauses to ask
when scope is ambiguous or expensive.

**Show `threadLink` as soon as the run starts.** The run happens in the
background on Unify's side, so the link is what lets the user watch it live —
intermediate steps, tool calls, and partial results stream into that thread while
you poll. Send one short message right after `run_agent` returns that says the
run is underway and includes the **bare URL** on its own line (most terminals and
plain-text clients auto-linkify a raw URL; Markdown link syntax only renders
where Markdown is supported). This is the one user-facing message to send before
a terminal status — everything else about polling stays internal.

`threadId` identifies the thread the agent ran in. Pass it back as
`run_agent({ prompt, threadId })` for a follow-up task that should build on the
same context (same thread, same link) instead of starting cold.

## 2. Poll: `poll_agent({ runId })`

Statuses: `PENDING` (keep polling), `CLARIFICATION_NEEDED`, `READY`, `ERROR`,
`NOT_FOUND`. Runs commonly take one to several minutes; poll every ~10 seconds
initially and back off toward ~30 seconds for long runs. Don't give up; complex
list-building runs can run 10+ minutes.
Treat polling as internal tool work. Do not send user-facing messages that
merely announce polling attempts, waits, cadence changes, or unchanged `PENDING`
statuses. Beyond the initial "run started, here's the link" message, only message
the user when input is required, the run reaches a terminal state, or an
actionable blocker occurs.

## 3. Clarifications: `answer_question({ runId, answers })`

On `CLARIFICATION_NEEDED`, `poll_agent` returns `questions` (up to 4, each with
2–4 options). Rules:

- Answer every question in one call, in the same order, with `question` copied
  **exactly** as returned; the server rejects mismatches.
- `answer` may be an option label or free text.
- If your conversation already contains the answer, answer directly; otherwise
  present the questions and options to your user verbatim and relay their choices.
- Response is `{ runId, alreadyResuming }`; keep polling either way. If the call
  fails with a retryable message, retry with the same answers.

## 4. Read: `read_agent_results({ runId })`

Only for `READY` or `ERROR` runs. Returns
`{ status, finalAnswer, content, errorMessage, threadLink }`.
`finalAnswer` is plain text. The result's **structured content** (`content`)
carries typed resource references with the exact IDs the loader tools require. A
DataTable reference includes `tableId` + `versionId` (both needed by
`load_datatable`), a List reference includes its `listId` (for `load_list`), and
so on. Take IDs from there rather than parsing them out of the prose.

`threadLink` is the same clickable thread URL `run_agent` returned, and comes
back for both `READY` and `ERROR` runs. Present it again after relaying results
so the user can inspect the full thread, intermediate steps, and any generated
resources in the UI.

**The result is the response; the link is only supporting context.** Never send
only the thread link or make the user open it to learn the outcome. Lead with
the substantive `finalAnswer` (or a faithful summary when it is long), include
the useful records or resource details you loaded, and state errors plainly.
Put the link last in a short, visually de-emphasized footer such as:

> Run details: <threadLink>

Keep the bare URL visible so terminals and plain-text clients can auto-link it.
In clients known to support inline HTML, `<sub>Run details: <threadLink></sub>`
is also acceptable, but do not rely on font sizing or hide the URL behind
Markdown link text.

## Errors

- "workspace does not have chat funding available" → billing gate; surface to the user.
- Rate-limit errors → wait and retry `run_agent` once; don't hammer.
- `ERROR` status → read `errorMessage`, report it plainly; a fresh, more specific brief often succeeds where a vague one failed.
- Runs are visible only to the user who created them; don't ask about runIds from other sessions or users.
