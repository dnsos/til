Server administratio cheat sheet

Check last server reboots:

```sh
last reboot
```

Also show shutdowns:

```sh
last -x | grep -E "reboot|shutdown"
```
