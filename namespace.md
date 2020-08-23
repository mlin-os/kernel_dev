### Commands

1. list existing namespaces on your machine
```Bash`
mu@ustc:~/project/kernel_dev$ lsns
        NS TYPE   NPROCS   PID USER COMMAND
4026531834 time       68  1117 mu   /lib/systemd/systemd --user
4026531835 cgroup     68  1117 mu   /lib/systemd/systemd --user
4026531836 pid        68  1117 mu   /lib/systemd/systemd --user
4026531837 user       68  1117 mu   /lib/systemd/systemd --user
4026531838 uts        68  1117 mu   /lib/systemd/systemd --user
4026531839 ipc        68  1117 mu   /lib/systemd/systemd --user
4026531840 mnt        68  1117 mu   /lib/systemd/systemd --user
4026532008 net        68  1117 mu   /lib/systemd/systemd --user
```

2. find the pid of a running program

```Bash
mu@ustc:~/project/kernel_dev$ pidof bash
3495 1646
```
