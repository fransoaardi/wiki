- swarm init

```bash
[dockerManager]$ sudo docker swarm init
Swarm initialized: current node (idwe8j9s60bzmyht4xgk1rz6w) is now a manager.
To add a worker to this swarm, run the following command:
docker swarm join \
--token SWMTKN-1-4jlc1t4i5e4f5o49a2a0vuqsphm9i4iywcxmocqbacw45b9f43-dlg05iws33lno43bwg35arsge \
11.22.33.44:2377
To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

- swarm join (노드별 반복한다)
```bash
[dockerWorker]$ sudo docker swarm join \
> --token SWMTKN-1-4jlc1t4i5e4f5o49a2a0vuqsphm9i4iywcxmocqbacw45b9f43-dlg05iws33lno43bwg35arsge \
> 11.22.33.44:2377
This node joined a swarm as a worker.
``` 

- node list
```bash
[dockerManager]$ sudo docker node ls
ID HOSTNAME STATUS AVAILABILITY MANAGER STATUS
3uhu9uat5jhbl7v3yshshvouz dockerjohn-2bcd82b4.node.server.aa Ready Active
idwe8j9s60bzmyht4xgk1rz6w * dockerjohn-eec717cb.node.server.aa Ready Active Leader
jeokaif7f43ng5sqrumexwql4 dockerjohn-85c0a8f9.node.server.aa Ready Active
s2i6z05jl0s2hf92m9jp46s8t dockerjohn-be7f6d00.node.server.aa Ready Active
```

- service create
```
[deploy@dockerjohn-eec717cb ~]$ sudo docker service create -p 9000:9000 --name swarm docker.hub.here/docker/image:dev
ko2z9pu9nmwkibs4cjv9ka0i7
```

- service list
```bash
[dockerManager]$ sudo docker service ls
ID NAME MODE REPLICAS IMAGE
ko2z9pu9nmwk swarm replicated 1/1 docker.hub.here/docker/image:dev
```

- service scale
```bash
[dockerManager]$ sudo docker service scale swarm=7
swarm scaled to 7
```

- image update
```bash
[dockerManager]$ sudo docker service update --image swarm docker.hub.here/docker/image:dev swarm
swarm
```
 

## Docker Swarm에 대한 생각

- 대규모 서버들을 동적으로 관리하는 경우에는 필요한 기능이 있다고 생각이됨
- `docker swarm`이라는 layer 가 한개 추가되어 관리대상이 늘어나는 느낌이 든다
- image update를 빠르게 진행하고 scaling 도 편하고 단순한 설정으로 clustering 을 지원하지만 항상 좋은건 아니고 서비스에 따라 선택해야됨
- 결국 server orchestration 은 `kubernetes` 가 대세이다. 
 
