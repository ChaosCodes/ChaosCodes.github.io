---
title: My first try in OpenFass
date: 2019-06-11 15:54:27
tags: OpenFaas
---

## My first try in OpenFass

> *This post records my first attempt in OpenFass, which contains several problems and corresponding solutions I met.*



### Build Virtual Machine

Due to lack of Raspberry Pis, I turn to Virtual Machine to simulate the environment and begin the tutorial. You can read the post [Steps to build VMs](https://chaoscodes.github.io/2019/03/09/Series-1-of-the-Research-Training/) for specific steps.

Besides the steps in that post，I choose birdge as the network model and install ***oh-my-zsh*** for a more powerful shell.

#### Zsh install

1. **Install zsh packet**

```shell
$ yum -y install zsh
```

2. **Switch zsh as default shell**

```shell
$ chsh -s /bin/zsh
```

3. **Install "on my zsh"**

```shell
curl sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```



###Function as a service

**Functions as a Service** is a framework for building Serverless functions on top of containers. In short "Serverless" is a misnomer - a new architectural pattern for event-driven systems. For this reason serverless functions are often used as connective glue between other services or in an event-driven architecture.



### Create Swarm cluster

#### Install docker

Here I prepare 2 vms to form the cluster, and first of all, install docker.

```shell
$ curl -fsSL https://get.docker.com/ | sh
```

Then, start the docker service

```shell
$ sudo systemctl start docker
```

Now we can see that the docker is active:

```shell
$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since 一 2019-06-10 09:33:50 EDT; 8s ago
     Docs: https://docs.docker.com
 Main PID: 35887 (dockerd)
    Tasks: 8
   Memory: 32.4M
   CGroup: /system.slice/docker.service
           └─35887 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/contain...
```



#### Initialize swarm manager node

To create a swarm cluster, there must be a manage node that can dispatch the task. We can select one of the vms and run command below.(Only need to run in the one vm that you choose as a master.)

```shell
$ docker swarm init
docker swarm init
Swarm initialized: current node (2nqd9opg0c8xw6sg6r7n06vzl) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-1jqi3cvr4ms32r9mha91l5f1m2s1or0r8kzkrv412b8l6u3kub-di2gvhrv38j5jsbti0s68wd39 192.168.1.19:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```



#### Add worker to swarm

From the initial output we can see the join token and the command to type into the other vm to join the swarm. 

So, here we enter the other vm and run command.

```shell
$ docker swarm join --token SWMTKN-1-1jqi3cvr4ms32r9mha91l5f1m2s1or0r8kzkrv412b8l6u3kub-di2gvhrv38j5jsbti0s68wd39 192.168.1.19:2377
This node joined a swarm as a worker.
```

Now we can check all my nodes in the manager.

```shell
$ docker node ls
ID                            HOSTNAME                STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
2nqd9opg0c8xw6sg6r7n06vzl *   localhost.localdomain   Ready               Active              Leader              18.09.6
btzn8j7ds4eo18y5dem8qcwft     localhost.localdomain   Ready               Active                                  18.09.6
```

> **Maybe it will throw an error when you add a worker.**
>
> Error response from daemon: rpc error: code = Unavailable desc = all SubConns are in TransientFailure, latest connection error: connection error: desc = "transport: Error while dialing dial tcp 10.211.55.3:2377: connect: no route to host"
>
> **Solution:**
>
> Just turn off the firewall by run command `sudo systemctl stop firewalld`.



### OpenFaaS

Now I move on to deploying a real application to enable Serverless functions to run on the cluster.



#### Build up OpenFaas

Get into the vm that serves as the manager and clone and deploy OpenFaas.

```shell
$ git clone https://github.com/alexellis/faas/
$ cd faas
$ ./deploy_stack.sh

Attempting to create credentials for gateway..
uzp0ezwhu2i05cl8sujgpn7tq
ukpztvpguicbu0yu7vv8tfslp
[Credentials]\n username: admin \n password: 9c2b624c6f76b0e55374d302c66bbf34f9c5d213b6ab2af1e554d1ba1d2b90a6\n echo -n 9c2b624c6f76b0e55374d302c66bbf34f9c5d213b6ab2af1e554d1ba1d2b90a6 | faas-cli login --username=admin --password-stdin

Enabling basic authentication for gateway..

Deploying OpenFaaS core services
Creating network func_functions
Creating config func_prometheus_config
Creating config func_prometheus_rules
Creating config func_alertmanager_config
Creating service func_gateway
Creating service func_basic-auth-plugin
Creating service func_faas-swarm
Creating service func_nats
Creating service func_queue-worker
Creating service func_prometheus
Creating service func_alertmanager
```



We can get the username and password from the output. Note that carefully, and they will be required in the latter step. The process will take a few minutes, rum command below and we can check their status.

```shell
$ watch 'docker service ls'
Every 2.0s: docker service ls                                                                                                         Mon Jun 10 14:01:16 2019

ID                  NAME                     MODE                REPLICAS            IMAGE                              PORTS
y8v1715n5z95        func_alertmanager        replicated          1/1                 prom/alertmanager:v0.16.1
bjzeek2a0bhr        func_basic-auth-plugin   replicated          1/1                 openfaas/basic-auth-plugin:0.1.1
yeqnmrjdgz70        func_faas-swarm          replicated          1/1                 openfaas/faas-swarm:0.6.2
wqqttp163kpq        func_gateway             replicated          1/1                 openfaas/gateway:0.13.7-rc5        *:8080->8080/tcp
qa73va6nowrh        func_nats                replicated          1/1                 nats-streaming:0.11.2
nf2wu1s1h9w4        func_prometheus          replicated          1/1                 prom/prometheus:v2.7.1             *:9090->9090/tcp
4gocpxua0h83        func_queue-worker        replicated          1/1                 openfaas/queue-worker:0.7.2
```

They will be done when the REPLICAS comes to 1/1. Then we can continue the tutorial.



Because I use Centos to simulate the Raspberry Pi, we run `ip addr` to get the IP address, and we get 192.168.1.19.

```shell
ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:01:3f:72 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.19/24 brd 192.168.1.255 scope global noprefixroute dynamic ens33
       valid_lft 63258sec preferred_lft 63258sec
    inet6 fe80::121a:f125:bbe9:4416/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```



### Web Servive in Virtual Machine

But at the first time, I can not get access to the server in my Host machine. I try to curl, but get a refuse message.

```shell
$ curl 192.168.1.19:8080
curl: (28) Operation timed out.
```



In order to find the problem, I create a new vm to set up a similar environment. I set up a flask server and manage to find the reason.

#### Difference Between 127.0.0.1 and 0.0.0.0

But what takes me a lot time to figure out first is my wrong setting to the listenning IP host. 

##### 127.0.0.1

127.0.0.1 is the loopback Internet protocol (IP) address also referred to as the *localhost*. The address is used to establish an IP connection to the same machine or computer being used by the end-user.

##### 0.0.0.0

0.0.0.0 is a valid address syntax. So it should parse as valid wherever an IP address in traditional dotted-decimal notation is expected. Once parsed and converted to workable numeric form, then its value determines what happens next.



So, because I set 127.0.0.1 as my host, the network packet can not be delivered to the public network, and thus we can not get the network packet from the other network interface controller.



#### Firework in Centos

After I change the host to 0.0.0.0, it still doesn't work. Quickly I wonder whether the Centos keep away the request from my host machine. So I check the firewall in the vm.

```shell
$ sudo systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)
```



But what surprises me is that the firewall in Centos does be turn off. I doubt myself and continue to search solution in Google. Finally I get a amazing command.

```shell
$ iptables -F
```

The command seems like to shut down the firewall thoroughly, but I cannot get more detail in Google. However, the web server problem is solved by the command.



Now, we can see the Faas UI in host machine.

![faas_Ui](faas_Ui.png)



### Deploy the first python serverless function

First, get the Faas-Cli.

```shell
$ curl -sSL cli.openfaas.com | sudo sh
which: no faas-cli in (/sbin:/bin:/usr/sbin:/usr/bin)
x86_64
Downloading package https://github.com/openfaas/faas-cli/releases/download/0.8.14/faas-cli as /tmp/faas-cli
Download complete.
```

Check whether Faas-Cli has been installed.

```shell
$ faas-cli version
  ___                   _____           ____
 / _ \ _ __   ___ _ __ |  ___|_ _  __ _/ ___|
| | | | '_ \ / _ \ '_ \| |_ / _` |/ _` \___ \
| |_| | |_) |  __/ | | |  _| (_| | (_| |___) |
 \___/| .__/ \___|_| |_|_|  \__,_|\__,_|____/
      |_|

CLI:
 commit:  25cada08609e00bed526790a6bdd19e49ca9aa63
 version: 0.8.14
```



### Write first python function

Create a new folder

```shell
$ mkdir -p ~/functions && \
  cd ~/functions
```

Then use Faas-Cli to scaffold a new Python function

```shell
$ faas-cli new --lang python hello-python
```

It will create three fills in the folder

```
+ hello-python
  - handler.py
  - requirements.txt
- hello-python.yml
```

So edit the *handler.py* as the tutorial

```python
def handle(req):
    print("Hello! You said: " + req)
```

 Then edit *hello-python.yml*:

```yaml
version: 1.0
provider:
  name: openfaas
  gateway: https://localhost:8080
functions:
  hello-python:
    lang: python
    handler: ./hello-python
    image: hello-python:latest
```



###Build the python function

let's build the function.

```shell
$ faas-cli build -f ./hello-python.yml
...
Successfully built a0287e7a1b13
Successfully tagged hello-python:latest
Image: hello-python:latest built.
```

If you want to push the function's image to a registry or the Docker Hub. Run command.

```shell
$ faas-cli push -f ./hello-python.yml

Unable to push one or more of your functions to Docker Hub:
- hello-python

You must provide a username or registry prefix to the Function's image such as user1/function1
```



It seem to fail and I found that in order to push a Docker image to the registered public Docker hub repository, we need to first login with our registered credentials. Besides, the name of the images should be like user1/function1.

After login in Docker by `docker login` and rename the image, the push succeed.

```shell
$ faas-cli push -f ./hello-python.yml
[0] > Pushing hello-python [chaos937/hello-python:latest].
The push refers to repository [docker.io/chaos937/hello-python]
...
[0] < Pushing hello-python [chaos937/hello-python:latest] done.
[0] worker done.
```



Now we can check in [Docker Hub](https://hub.docker.com/) and found that the image has been pushed to the repository.

![dockerhub](dockerhub.png)



### Deploy the python function

After building, it's time to deploy the function.

```shell
$ faas-cli deploy -f ./hello-python.yml
Deploying: hello-python.
WARNING! Communication is not secure, please consider using HTTPS. Letsencrypt.org offers free SSL/TLS certificates.

unauthorized access, run "faas-cli login" to setup authentication for this server

Function 'hello-python' failed to deploy with status code: 401
```

Similar failure hit me. The output tell me that I have no authority to access. Remember OpenFaaS uses basic-auth protection. Hence, we first need to authenticate with the API gateway http://192.168.1.19:8080 using the basic-auth credentials.

> Username and password are shown in the output of command `./deploy_stack.sh`



Login in faas-cli

```shell
$ echo -n 9c2b624c6f76b0e55374d302c66bbf34f9c5d213b6ab2af1e554d1ba1d2b90a6 | faas-cli login --username=admin --password-stdin --gateway http://192.168.1.19:8080  
Calling the OpenFaaS server to validate the credentials...
WARNING! Communication is not secure, please consider using HTTPS. Letsencrypt.org offers free SSL/TLS certificates.
credentials saved for admin http://192.168.1.19:8080
```

Again deploy the function.

```shell
$ faas-cli deploy -f hello-python.yml --gateway http://192.168.1.19:8080  
Deploying: hello-python.
WARNING! Communication is not secure, please consider using HTTPS. Letsencrypt.org offers free SSL/TLS certificates.

Deployed. 202 Accepted.
URL: http://192.168.1.19:8080/function/hello-python
```



Now we can see *hello-python* in Faas-Cli UI.

![hello-python](hello-python.png)

Use `curl` to call it in terminal and we can get the response:

```shell
$ curl http://192.168.1.19:8080/function/hello-python -d  "it's tao here"
Hello!, you said: it's tao here
```



In addition, we can also use faas-cli to list and invoke functions.

##### List

```shell
$ faas-cli list
Function                      	Invocations    	Replicas
hello-python                  	6              	1    
```

Here, we can easily know how many times the functions call and their replicas.

##### Invoke

```shell
$ echo "hello" | faas-cli invoke hello-python
Hello!, you said: hello
```

We can also use fans-cli to invoke the function just like the command above.



### Write Python fuction with 3rd party dependencies

Let's look back at the file `hello-python/requirements.txt`. You can use `pip` by providing the file along with your function handler.

Here we add the request in `requirements.txt`:

```
request
```

And then update the Python code

```python
import requests
import json

def handle(req):
    result = {"found": False}
    json_req = json.loads(req)
    r = requests.get(json_req["url"])
    if json_req["term"] in r.text:
        result = {"found": True}

    print json.dumps(result)
```



Rebuild and re-deploy.

```shell
$ faas-cli build -f ./hello-python.yml && \
  faas-cli deploy -f ./hello-python.yml
```

Test the new function:

```shell
$ curl http://192.168.1.19:8080/function/hello-python --data-binary '{
quote>  "url": "https://www.baicu.com",
quote>  "term": "baidu"
quote> }'
{"found": true}
```

It works!



### Data Analysis

Faas-Cli can do more about the functions.

[Prometheus](https://github.com/prometheus) is an open-source systems monitoring and alerting toolkit originally built at SoundCloud, which means you can checkout the various metrics on how your functions are being used as well as how long they're taking to run.

We can view the Prometheus UI at http://192.168.1.19:9090.

![metrics](metrics.png)



### In Conclusion

This is my first time to take a journay in OpenFaas. During the attempt, I find it interesting to deploy a service in OpenFaas due to its lightweight and flexible service. I will continue to explore more in the field.


