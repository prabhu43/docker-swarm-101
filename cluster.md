## Create virtual machines using docker-machine:
docker-machine create -d virtualbox manager1
docker-machine create -d virtualbox worker1
docker-machine create -d virtualbox worker2
docker-machine create -d virtualbox worker3

## Setup Swarm cluster
- The cluster management and orchestration features embedded in the Docker Engine are built using swarmkit. Swarmkit is a separate project which implements Docker’s orchestration layer and is used directly within Docker.
- Kind of nodes: 
    - manager: 
        - Many managers can run in a swarm for HA. Manager nodes elect a single leader to conduct orchestration tasks. Uses RAFT consensus to make sure that all the manager nodes that are in charge of managing and scheduling tasks in the cluster, are storing the same consistent state. 
        - Performs orchestration and cluster management functions required to maintain the desired state of the swarm
        - Manager node can also run tasks by default. But, can be made manager-only node. 
        - To deploy your application to a swarm, you submit a service definition to a manager node. The manager node dispatches units of work called tasks to worker nodes

    - worker: receive and execute tasks dispatched from manager nodes
- Docker Engine uses a declarative approach to let you define the desired state of the various services in your application stack
- Task: basic unit of swarm, running container which is part of swarm service. Similar to pod in k8s.

#### Create master node
* Initialize swarm using a manager node
```
docker@manager1:~$ docker swarm init --advertise-addr 192.168.99.101
Swarm initialized: current node (s1xltwwabdkbz8ev15sb55utm) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-32ywq14cfcphqlurus8lejgvs9bczo9gubta297zse8b6nipbo-71rm5ns42gwf238xoi3p119l7 192.168.99.101:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```
* List the nodes in swarm
```
docker@manager1:~$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
s1xltwwabdkbz8ev15sb55utm *   manager1            Ready               Active              Leader              18.09.5
```

* Get the command to join a node as manager in swarm.
Only the manager node can run this command
```
docker@manager1:~$ docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-32ywq14cfcphqlurus8lejgvs9bczo9gubta297zse8b6nipbo-ax3hj7kmuh3lphffb1h8w6lo2 192.168.99.101:2377
```

* Get the command to join a node as worker in swarm.
Only the manager node can run this command
docker@manager1:~$ docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-32ywq14cfcphqlurus8lejgvs9bczo9gubta297zse8b6nipbo-71rm5ns42gwf238xoi3p119l7 192.168.99.101:2377

#### Join 
```
docker@worker1:~$ docker swarm join --token SWMTKN-1-32ywq14cfcphqlurus8lejgvs9bczo9gubta297zse8b6nipbo-ax3hj7kmuh3lphffb1h8w6lo2 192.168.99.101:2377
This node joined a swarm as a manager.
```

* List nodes in a swarm

Manager Status: 
    - Leader: leader of manager nodes
    - Reachable: manager & non-leaders
    - Empty: worker

```
docker@manager1:~$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
s1xltwwabdkbz8ev15sb55utm *   manager1            Ready               Active              Leader              18.09.5
lw7ux5n5j5w36cjatwx1cfhqi     worker1             Ready               Active              Reachable           18.09.5
yo897e7oph4h2tittuc1szq4k     worker2             Ready               Active              Reachable           18.09.5
3l9phhc7shvpk6ygzgc2hq733     worker3             Ready               Active              Reachable           18.09.5
```

```
docker@manager1:~$ docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
s1xltwwabdkbz8ev15sb55utm *   manager1            Ready               Active              Leader              18.09.5
lw7ux5n5j5w36cjatwx1cfhqi     worker1             Ready               Active                                  18.09.5
yo897e7oph4h2tittuc1szq4k     worker2             Ready               Active                                  18.09.5
3l9phhc7shvpk6ygzgc2hq733     worker3             Ready               Active                                  18.09.5
```


* `docker info` in a node shows its config in a swarm
docker@manager1:~$ docker info
```
Swarm: active
 NodeID: s1xltwwabdkbz8ev15sb55utm
 Is Manager: true
 ClusterID: 3dtq50aoxyhs3mvyzx4hgt0t1
 Managers: 4
 Nodes: 4
 Default Address Pool: 10.0.0.0/8
 SubnetSize: 24
 Orchestration:
  Task History Retention Limit: 5
 Raft:
  Snapshot Interval: 10000
  Number of Old Snapshots to Retain: 0
  Heartbeat Tick: 1
  Election Tick: 10
 Dispatcher:
  Heartbeat Period: 5 seconds
 CA Configuration:
  Expiry Duration: 3 months
  Force Rotate: 0
 Autolock Managers: false
 Root Rotation In Progress: false
 Node Address: 192.168.99.101
 Manager Addresses:
  192.168.99.101:2377
  192.168.99.102:2377
  192.168.99.103:2377
  192.168.99.104:2377
```

* Cluster management commands fail in worker nodes
```
docker@worker3:~$ docker node demote worker2
Error response from daemon: This node is not a swarm manager. Worker nodes can't be used to view or modify cluster state. Please run this command on a manager node or promote the current node to a manager.
```

* Demote a worker node to manager
```
docker@manager1:~$ docker node promote worker1
Node worker1 promoted to a manager in the swarm.
```

* Promote a manager node to worker
```
docker@manager1:~$ docker node demote worker1
Manager worker1 demoted in the swarm.
```

#### Ports required:
- TCP port 2377 for cluster management communications
- TCP and UDP port 7946 for communication among nodes
- UDP port 4789 for overlay network traffic

## Scheduling
- Resource Awareness: SwarmKit is aware of resources available on nodes and will place tasks accordingly
- Spread strategy: schedule tasks on the least loaded nodes, provided they meet the constraints and resource requirements
- Contraints: Operators can limit the set of nodes where a task can be scheduled by defining constraint expressions

## Services

Service Types:
    - Replicated
    - Global(daemonset in K8s)

### Update services
- Default: lockstep update. Update all tasks at same time
- Configurations:
    - Paralleism - how many updates can be performed at the same time.
    - Delay - minimum delay between updates


## References:
https://docs.docker.com/engine/swarm/
https://docs.docker.com/engine/swarm/raft/
https://github.com/docker/swarmkit/
