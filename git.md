### FQA

1. How to reove a file from git repository without deleting it from the local filesystem?
```Bash
git rm --cached mylogfile.log # remove a single file
git rm --cached -r mydirectory # remove a single directory
```

2. How to remove an entry in `git config`?
```Bash
git config --global --unset XXX
```

3. How to store user and password without ssh key?
```Bash
git config credential.helper store
```

4. How to rename remote repository's name?
```Bash
git remote rename origin destination # rename origin to destination
```
