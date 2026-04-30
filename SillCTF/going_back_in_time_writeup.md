# Going Back In Time

Type: Web

## Summary

The landing page looked static and only referenced a stylesheet at `2000/assets/stylesheet.css`. Visiting `/2000/` revealed the first real clue: the page title changed to `Welcome to 2000!` and it linked to `1999.css`.

The trick was that the challenge did not use top-level year directories like `/1999/`. Instead, each previous year existed as a nested subdirectory:

```text
/2000/
/2000/1999/
/2000/1999/1998/
...
/2000/1999/1998/1997/1996/1995/1994/1993/1992/1991/1990/
```

Walking that chain eventually reached the 1990 page, which contained:

```text
c2lsbHlDVEZ7SV9MMHZlX3RoRV85T3N9
```

That value was Base64.

## Solve

Fetch the final page:

```bash
curl -sS https://goingbackintime.sillyctf.psuccso.org/2000/1999/1998/1997/1996/1995/1994/1993/1992/1991/1990/
```

Decode the string:

```bash
printf 'c2lsbHlDVEZ7SV9MMHZlX3RoRV85T3N9' | base64 -d
```

Output:

```text
sillyCTF{I_L0ve_thE_9Os}
```

## Flag

```text
sillyCTF{I_L0ve_thE_9Os}
```
