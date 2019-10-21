# Find Port in Use

## osx

```bash
$ lsof -i -P -n | grep TCP
```

## linux

```bash
$ netstat -ntlp | grep 8080
```
