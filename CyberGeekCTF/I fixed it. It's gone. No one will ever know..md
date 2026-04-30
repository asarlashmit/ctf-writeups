# I fixed it. It's gone. No one will ever know.

## Challenge Overview
- Event: CyberGeek CTF
- Category: Forensics / Git
- Prompt: Recover the deleted secret from the cleaned Git repository.
- Final Flag: geek{n0_tr4c3_l3ft_8a92f1}

## Executive Summary
The challenge provided a cleaned To-Do app repository. The description hinted that a junior developer had removed a sensitive configuration file before pushing a "Clean Version". The working tree looked normal, but the `.git` directory was still present, so the deleted secret could still exist in Git objects.

## Solution Walkthrough
### Recon
After extracting the archive, the repository contained a normal Node.js To-Do app:

```bash
ls -la
cd todo
git log --oneline --decorate --all --graph
```

The visible Git history only showed one clean commit:

```text
beac2b7 Official Production Release v1.0
```

Searching the files for sensitive keywords showed that `.gitignore` referenced a config file:

```bash
find . -type f -print0 | xargs -0 grep -nI -E 'geek\{|flag|secret|config|env|token|key'
```

Relevant result:

```text
.gitignore:2:.env.config
```

That matched the challenge description: a sensitive config file was probably deleted from the working tree.

### Finding Deleted Git Objects
Because the `.git` directory was included, I checked for unreachable objects:

```bash
git fsck --full --no-reflogs --unreachable --lost-found
```

This returned several unreachable commits, trees, and blobs. One suspicious blob was:

```text
unreachable blob 5dca9d1f9f77073759e4374b3ce315894c74349b
```

I dumped the blob:

```bash
git cat-file -p 5dca9d1f9f77073759e4374b3ce315894c74349b
```

Output:

```text
SECRET_DB_KEY=geek{n0_tr4c3_l3ft_8a92f1}
```

### Root Cause
The developer deleted the sensitive config file from the working tree, but Git still retained the old blob object. Since the `.git` directory was shipped with the application, the supposedly deleted secret could be recovered from unreachable Git objects.

## Flag
```text
geek{n0_tr4c3_l3ft_8a92f1}
```
