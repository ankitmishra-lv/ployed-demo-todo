# Ployed Demo — Full-Stack Todo Application

**Status:** demo specification · **Owner:** Ankit · **Date:** 2026-08-19

The contract both agents build against. Deliver it either by committing it to the
demo repository or by pasting the relevant sections into each agent's prompt —
see §9. What matters is that both agents work from the *same* text.

---

## 1. Goal

A working todo application demonstrating that two Ployed agents can build
complementary halves of one codebase, each shipping a reviewable pull request.

Deliberately small: the point is the collaboration and the governance trail, not
the feature set.

## 2. Scope

**In scope**
- Rendered HTML UI, no build step
- HTTP API over an in-memory store
- One repository, two agents, two pull requests

**Out of scope** — do not implement
- Persistence beyond process memory, authentication, multi-user support
- Frameworks (React/Vue/Next), bundlers, TypeScript, Docker, CI config
- Pagination, search, tags, due dates, drag-and-drop

## 3. Stack

| Layer | Choice | Why |
|---|---|---|
| Runtime | Node.js 20+ | global `fetch`, built-in test runner |
| Server | Express 4 | smallest credible HTTP layer |
| Store | in-memory `Map` | zero provisioning; state loss on restart is intentional |
| UI | vanilla HTML/CSS/JS + `fetch` | no build step, no bundler |
| Tests | `node --test` | zero extra dependencies |

## 4. File ownership

Ownership is strict. An agent must not create or modify files outside its lane —
this, plus merging between the two tasks, is what keeps the pull requests from
colliding.

```
/README.md              created at repo init  — neither agent edits
/docs/REQUIREMENTS.md   optional seed (§9)    — neither agent edits
/package.json           BACKEND creates it    — FRONTEND must not touch it
/server/                BACKEND only
/public/                FRONTEND only
```

## 5. API contract

Base path `/api`. All request and response bodies are JSON.

| Method | Path | Body | Success | Notes |
|---|---|---|---|---|
| `GET` | `/api/todos` | — | `200` `Todo[]` | newest first |
| `POST` | `/api/todos` | `{ title }` | `201` `Todo` | |
| `PATCH` | `/api/todos/:id` | `{ title?, completed? }` | `200` `Todo` | partial update |
| `DELETE` | `/api/todos/:id` | — | `204` empty | |

### Todo

```json
{
  "id": "3f2a9c14-...",     // uuid, server-generated
  "title": "Buy milk",       // 1..200 chars, trimmed
  "completed": false,
  "createdAt": "2026-08-19T10:00:00.000Z"   // ISO 8601
}
```

### Errors

Every failure returns a JSON body of this exact shape:

```json
{ "error": { "code": "invalid_title", "message": "title must be 1-200 characters" } }
```

| Status | When | `code` |
|---|---|---|
| `400` | missing/empty/over-long `title`, or `completed` not a boolean | `invalid_title` / `invalid_completed` |
| `404` | unknown `:id` | `not_found` |

## 6. Server behaviour

- Serves `/public` as static files at `/`, so `GET /` returns the UI.
- Listens on `process.env.PORT || 3000`.
- Exports the Express `app` separately from the `listen()` call so tests can
  import it without binding a port.
- No request logging, no CORS middleware (same origin).

## 7. UI behaviour

Single page: `public/index.html`, `public/app.js`, `public/styles.css`.

- Lists todos newest-first, each with a checkbox, its title, and a delete control.
- An input plus submit button adds a todo; the field clears on success.
- Toggling the checkbox `PATCH`es `completed`; completed titles render struck through.
- Delete removes the row.
- Empty state reads "Nothing to do yet."
- A failed request shows an inline, dismissible error using the `message` from the
  error body. The page must not go blank on a failed request.
- Accessible basics: a `<label>` for the input, buttons are real `<button>`s,
  visible focus states.

## 8. Acceptance criteria

**Backend**
1. `node --test` passes, covering: list empty, create, reject empty title, reject
   201-char title, patch `completed`, patch unknown id → `404`, delete → `204`.
2. `npm install && npm start` works, and `curl localhost:3000/api/todos` returns `[]`.
3. Nothing added or changed outside `server/` and `package.json`.

**Frontend**
1. `GET /` serves the page; add / toggle / delete each work against the live API.
2. Empty state and the inline error path are both reachable.
3. Nothing added or changed outside `public/` — including no edit to `package.json`
   and no new dependency.

**Both**
- One pull request per agent, opened against `main`, `/autoreview` clean.
- The pull request body states what was verified and how.

## 9. Getting this contract to the agents

**Required setup is only this:** a **public** GitHub repository that already has at
least one commit. Creating it on GitHub with "Add a README" ticked satisfies both.

Public is not optional — a private repo fails checkout (PLO-78). The existing commit
is not strictly required, but without one the runtime writes a `.ployed-init` file to
`main` to manufacture a PR base, and that file stays in your repo (PLO-80).

Then pick one of:

| | how | trade-off |
|---|---|---|
| **A** | Paste §5–§8 into each prompt | nothing to seed; you paste twice, and the two agents diverge if the text differs |
| **B** | Commit this file to `docs/REQUIREMENTS.md`, prompts point at it | one shared source of truth, shorter prompts; costs one commit |
| **C** | Have `product-manager-agent` write the spec from a brief, review and merge it, then build | strongest demo — spec → build → review — but a third task and an agent-authored contract |

B is the low-variance default. A is the smallest setup. C is the best story if you
have appetite for one more moving part.
