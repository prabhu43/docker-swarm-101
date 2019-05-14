## Deploy a service to swarm

`docker service` commands will work on manager nodes

* To deploy a service 
docker service create --replicas 1 --name helloworld alpine ping docker.com
service name: helloworld
replicas: 1 instance/task/container
`alpine`: image of container
`ping docker.com`: entrypoint for container

* To list all servies in a swarm
docker service ls

* To see details about a service
docker service inspect --pretty helloworld
Note: Remove `--pretty` to print the output in json

* To see details of nodes in which a service is running
docker service ps helloworld

* Scale a service
docker service scale <SERVICE-ID> =<NUMBER-OF-TASKS>
docker service scale helloworld=5

```
docker@manager1:~$ docker service ps helloworld
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE               ERROR               PORTS
qcj8e9y7ufqd        helloworld.1        alpine:latest       manager1            Running             Running about an hour ago
dia3656jopu5        helloworld.2        alpine:latest       worker1             Running             Running 6 seconds ago
uubtz2ykltle        helloworld.3        alpine:latest       worker2             Running             Running 6 seconds ago
i5f1vj21v0xt        helloworld.4        alpine:latest       worker2             Running             Running 6 seconds ago
6qr0r04vxso0        helloworld.5        alpine:latest       manager1            Running             Running 14 seconds ago
```

* Delete a service
docker service rm helloworld 

* Apply rolling updates to a service
docker service create --replicas 3 --name redis --update-delay 10s redis:3.0.6
--update-delay flag configures the time delay between updates to a service task or sets of tasks

docker service update --image redis:3.0.7 redis

The scheduler applies rolling updates as follows by default:

Stop the first task.
Schedule update for the stopped task.
Start the container for the updated task.
If the update to a task returns RUNNING, wait for the specified delay period then start the next task.
If, at any time during the update, a task returns FAILED, pause the update.


* Drain node
docker node update --availability drain worker1

* Make a drained node active again
docker node update --availability active worker1
