# intro 

elasticsearch install command 모음 

# references

## 설치관련
- https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html
- https://www.elastic.co/guide/en/kibana/current/rpm.html

## elasticsearch.yml network 설정 관련 
- https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-network.html

# how-to 
## elasticsearch rpm install
```console
$ curl -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.5.2-x86_64.rpm

$ sudo rpm --install elasticsearch-7.5.2-x86_64.rpm
```

## config.yml modification 

```yml
/etc/elasticsearch/elasticsearch.yml  수정 

network.host: 0.0.0.0
discovery.seed_hosts: ["0.0.0.0"]
```

## systemctl set
```console
$ sudo /bin/systemctl daemon-reload
$ sudo /bin/systemctl enable elasticsearch.service

$ sudo systemctl start elasticsearch.service
$ sudo systemctl stop elasticsearch.service
```

## kibana rpm install
```console
$ curl -O https://artifacts.elastic.co/downloads/kibana/kibana-7.5.2-x86_64.rpm

$ sudo rpm --install kibana-7.5.2-x86_64.rpm
```

## config.yml modificiation 
```yml
/etc/kibana/kibana.yml 파일 수정

server.host: 0.0.0.0
```

## systemctl set
```console
$ sudo /bin/systemctl daemon-reload
$ sudo /bin/systemctl enable kibana.service

$ sudo systemctl start kibana 
```


