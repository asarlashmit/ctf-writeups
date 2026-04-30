# Six-sevenn

Type: web / source leak

## Summary

The provided source archive includes the full `.git` directory.  
`HEAD` contains a blurred placeholder flag, but earlier commits still contain the original hardcoded flag value returned by the exclusive drink QR flow.

## Recon

Extract the archive and inspect the repository history:

```bash
tar -xzf sixseven_git_4547ca8.tar.gz
git -C sixseven log --oneline --decorate --graph --all
git -C sixseven grep -n "alfa{" $(git -C sixseven rev-list --all)
```

Relevant output:

```text
a28c686...:app/app.py:45:        flag = "alfa{XXXXXXXXXX}"
c67f4b0...:app/app.py:45:        flag = "alfa{i_love_coffee_so_much!!!}"
ab6813d...:app/app.py:41:        flag = "alfa{i_love_coffee_so_much!!!}"
```

The last commit is explicitly named `Fix: blur flag`, which means the previous commit still preserves the real value.

## Flag

```text
alfa{i_love_coffee_so_much!!!}
```
