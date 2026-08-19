# ployed-demo-todo

Full-stack todo application built by two Ployed agents, each shipping its own
reviewable pull request.

- `server/` — Express 4 API over an in-memory store *(backend-dev-agent)*
- `public/` — vanilla HTML/CSS/JS UI *(frontend-dev-agent)*

The contract both agents build against is [`docs/REQUIREMENTS.md`](./docs/REQUIREMENTS.md).

## Run it

```bash
npm install
npm test      # backend tests
npm start     # then open http://localhost:3000
```

State lives in process memory, so restarting the server clears the list. That is
intentional — see the requirements.
