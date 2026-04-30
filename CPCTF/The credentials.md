# The credentials

## Summary

The distributed ZIP contained a Git repository after `git filter-branch` was used to remove `flag.txt` from history. The cleanup was incomplete: `git filter-branch` left a backup ref at `.git/refs/original/refs/heads/master`, and the reflog also retained the old commit IDs.

## Recon

The archive contained a full `.git` directory. Listing refs showed both the rewritten branch and the backup ref:

```bash
cat .git/refs/heads/master
cat .git/refs/original/refs/heads/master
```

`git log --all` revealed commits from both histories:

```bash
git log --all --decorate --oneline --stat
```

The original initial commit `1b9465c83a7d8c48b9ef5eedad4493c5900bb475` still contained `flag.txt`.

## Exploit

Extract the removed file directly from the old commit:

```bash
git show 1b9465c83a7d8c48b9ef5eedad4493c5900bb475:flag.txt
```

## Flag

```text
CPCTF{n3ver_c0mmit_y0ur_cr3dential5}
```
