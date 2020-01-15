# docker-compose

## install

> assume docker is properly installed

> may need sudo

```bash
$ curl -L https://github.com/docker/compose/releases/download/1.21.0/docker-compose-`uname -s`-`uname -m` | sudo tee /usr/local/bin/docker-compose > /dev/null
$ chmod +x /usr/local/bin/docker-compose
```

## version check 

```bash
$ docker-compose --version  
```

## update 

```bash
$ curl -L https://github.com/docker/compose/releases/download/1.25.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```
