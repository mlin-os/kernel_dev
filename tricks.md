### Shell-related skills

1. [How to exclude directories from `grep`](https://stackoverflow.com/questions/6565471/how-can-i-exclude-directories-from-grep-r)?

```Bash
alias grep="grep --exclude-dir={*.svn,*.git}"
```
