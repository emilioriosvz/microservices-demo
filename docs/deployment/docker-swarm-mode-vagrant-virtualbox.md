---
layout: default
---
## Docker Swarm

Please refer to the [new Docker Swarm introduction](http://container-solutions.com/hail-new-docker-swarm/)

### Blockers

Currently, new Docker Swarm does not support running containers in privileged mode.
Maybe it will be allowed in the future.
Please refer to the issue [1030](https://github.com/docker/swarmkit/issues/1030#issuecomment-232299819).
This prevents running Weave Scope in a normal way, since it needs privileged mode.
A work around exists documented [here](https://github.com/weaveworks/scope-global-swarm-service)

Running global plugins is not supported either.

### Overview

This setup includes 3 nodes for Docker Swarm.  
* master1 - is the Docker Swarm manager node  
* node1, node2 - worker nodes


# How-to using Vagrant and VirtualBox

* Navigate to the vagrant directory and launch vagrant boxes using up.sh
* SSH into the master node ```vagrant ssh```
* Launch docker swarm manager
* SSH into node1 and join the swarm
* SSH into node2 and join the swarm
* If everything succeeded, we should have a docker swarm cluster of 3 nodes.
* Now, on the master1 node execute the following script:

This will spawn all the services composing weave-socks app and expose the application on 192.168.11.10:30000
Since the front-end is run in ```--mode global``` it will be available on all nodes.

<!-- deploy-test-start pre-install -->

    apt-get install -yq wget
    echo "deb http://download.virtualbox.org/virtualbox/debian xenial contrib" >> /etc/apt/sources.list
    wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | apt-key add -
    wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | apt-key add -

    apt-get update
    apt-get install -yq vagrant virtualbox-5.1

    ./up.sh
    docker swarm init --secret "" --listen-addr 192.168.11.10:2377
    docker swarm join --listen-addr 192.168.11.11:2377 192.168.11.10:2377
    docker swarm join --listen-addr 192.168.11.12:2377 192.168.11.10:2377
    /vagrant/start-swarmkit-services.sh

<!-- deploy-test-end -->

<!-- deploy-test-start run-tests -->

    docker run weaveworksdemos/load-test -d 60 -h 192.168.11.10:30000 -c 3 -r 10

    STATUSCODE=$(curl --silent --output /dev/stderr --write-out "%{http_code}" http://localhost/health)
    if [ $STATUSCODE -ne 200 ]; then
        echo "$(tput setaf 1)DEPLOY FAILED$(tput sgr0)"
        exit 1
    fi

<!-- deploy-test-end -->
