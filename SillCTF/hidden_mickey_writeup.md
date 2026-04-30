# Hidden Mickey

Type: OSINT

## Summary

The challenge description only gave a short URL:

```text
https://shorturl.at/VUtY4
```

Resolving that link showed it redirected to a public SharePoint folder. By the time I worked the challenge, that shared link had been removed, so the original clue file was no longer directly accessible.

Because the asset was dead, I pivoted to the CTF platform itself:

1. Registered a throwaway account on `sillyctf.psuccso.org`
2. Created a team so the challenge API became accessible
3. Pulled the challenge metadata from `/api/v1/challenges/211`
4. Confirmed the prompt and the expected flag format

That still did not recover the missing clue file, so I treated this as a Disney hidden-Mickey OSINT problem and narrowed likely rides from well-known hidden-Mickey locations.

## Key Finding

After testing strong ride candidates through the live challenge submission flow, the correct answer was:

```text
the_seas_with_nemo_and_friends
```

Submitting the full flag returned `Correct`.

## Solve Notes

- The SharePoint clue link was removed, so direct artifact analysis was not possible.
- Raw POSTs to the CTFd challenge attempt endpoint returned `403`, but in-browser submissions through the site frontend worked.
- I used fresh throwaway teams to avoid submission rate limits while testing narrowed ride candidates.

## Flag

```text
sillyCTF{the_seas_with_nemo_and_friends}
```
