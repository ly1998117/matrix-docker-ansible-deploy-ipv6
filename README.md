# [Ipv6 Support] Matrix server setup using Ansible and Docker

## Purpose
> This [Ansible](https://www.ansible.com/) playbook is meant to help you run your own [Matrix](http://matrix.org/) homeserver, along with the [various services](#supported-services) related to that.
> 
> That is, it lets you join the Matrix network using your own `@<username>:<your-domain>` identifier, all hosted on your own server (see [prerequisites](docs/prerequisites.md)).
> 
> We run all services in [Docker](https://www.docker.com/) containers (see [the container images we use](docs/container-images.md)), which lets us have a predictable and up-to-date setup, across multiple supported distros (see [prerequisites](docs/prerequisites.md)) and [architectures](docs/alternative-architectures.md) (x86/amd64 being recommended).
> 
> [Installation](docs/README.md) (upgrades) and some maintenance tasks are automated using [Ansible](https://www.ansible.com/) (see [our Ansible guide](docs/ansible.md)).

The Matrix server, originally set up [using Ansible and Docker](https://github.com/spantaleev/matrix-docker-ansible-deploy), operates efficiently on servers with public IPv4 addresses. However, in my experience, **Matrix federation fails on IPv6-only servers** (without public IPv4). This issue likely stems from the server's foundation on Docker, as the official Docker platform lacks built-in IPv6 NAT support, to the best of my knowledge. Therefore, we can potentially resolve this issue by configuring the IPv6 network settings of Docker.

## Requirements
+ Security: The container's network needs to be private, and its behavior should align with Docker IPv4. Only specific ports can be accessed.
+ The host machine is in a small range of subnets (with a prefix greater than /80), such as xxx::xx/128, and there are no extra IPv6 addresses available for allocation to containers.

## Resolution
The official Docker does not have built-in IPv6 NAT. If you want to use IPv6 NAT, you need to install an external IPv6 startup. see [docker-ipv6nat](https://github.com/robbertkl/docker-ipv6nat).

Using the following command, start IPv6 NAT in the background (configured to start automatically on boot using --restart always).

```
docker run -d --name matrix-ipv6nat --privileged --network host --restart always -v /var/run/docker.sock:/var/run/docker.sock:ro -v /lib/modules:/lib/modules:ro robbertkl/ipv6nat
```

Customize network support for IPv6, create a bridge network matrix that supports IPv6. The --subnet parameter is a subnet of fe80::/10.
```
docker network create matrix --subnet="fd00:1::1/80" --gateway="fd00:1::1"
```
Check if it is effective by using the command `docker network inspect my-net-ipv6`. If it is effective, then the value of EnableIPv6 should be true.

## Installation
I have incorporated an IPv6 setup process into the original project, using the [docker-ipv6nat](https://github.com/robbertkl/docker-ipv6nat) container.

+ Added the docker-ipv6nat container to the path `/roles/custom/matrix-ipv6nat` and set IPv6 NAT to be enabled by default.
+ Added IPv6 listening configuration `listen [::]:<port>` to several `matrix-*.conf` files located in the path `/roles/custom/matrix-nginx-proxy/templates/nginx/conf.d`.

For setup instructions, please refer to the original project's Installation Guide: [Matrix server setup using Ansible and Docker](https://github.com/spantaleev/matrix-docker-ansible-deploy).

## Result

+ All containers after `ansible-playbook` deployment,  We can see that `ipv6nat` started successfully.

  ![ps -a](/Users/liuyang/DATA/Downloads/matrix-ipv6/images/ps -a.png)

+ ipv6 test inside the docker container `matrix-nginx-proxy`

  ![ping6 in docker container](/Users/liuyang/DATA/Downloads/matrix-ipv6/images/ping6 in docker container.png)

## Related Project

+ [docker-ipv6nat](https://github.com/robbertkl/docker-ipv6nat). 

+ [Matrix (An open network for secure, decentralized communication) server setup using Ansible and Docker](https://github.com/spantaleev/matrix-docker-ansible-deploy)