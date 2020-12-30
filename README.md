# pwn-college

This repository has the artifacts necessary for deploying a pwn.college instance.

## Setup

```bash

git clone https://github.com/cszuo/pwncollege.git /opt/pwncollege/
cd /opt/pwncollege/

#docker build -t pwncollege_challenge containers/pwncollege_challenge
# or
docker pull xiaoyanfujun/pwncollege_challenge:1.0
docker tag xiaoyanfujun/pwncollege_challenge:1.0 pwncollege_challenge

#docker build -t pwncollege_ctfd CTFd
# or
docker pull xiaoyanfujun/pwncollege_ctfd:1.0
docker tag xiaoyanfujun/pwncollege_ctfd:1.0 pwncollege_ctfd

sudo chmod 666 /var/run/docker.sock
sudo adduser pwncollege
sudo pip3 install docker

cp -r CTFd-pwn-college-plugin CTFd/CTFd/plugins

./init_user_homes.py 500 # MAX_USER_ID
```

## Start

```bash


docker-compose up -d

docker run --detach --name nginx-proxy --publish 80:80 --publish 443:443 --volume /etc/nginx/certs --volume `pwd`/conf/nginx/vhost.d:/etc/nginx/vhost.d --volume /usr/share/nginx/html --volume /var/run/docker.sock:/tmp/docker.sock:ro jwilder/nginx-proxy

docker run --detach --name nginx-proxy-letsencrypt --volumes-from nginx-proxy --volume /var/run/docker.sock:/var/run/docker.sock:ro --env "DEFAULT_EMAIL=example@example.com" jrcs/letsencrypt-nginx-proxy-companion

docker network connect pwncollege_default nginx-proxy

docker restart nginx-proxy

```

It may be something other than pwncollege_default, depending on instance.

For container access (/etc/ssh/sshd_config):
```
Match User pwncollege
      AuthorizedKeysCommand /opt/pwncollege/auth.py pwncollege_db pwncollege
      AuthorizedKeysCommandUser root
      X11Forwarding no
      AllowTcpForwarding no
```

Persistent mounts (in /opt/pwn-college/persistent):
```bash
sudo su ubuntu
cp -r data/0 data/$i
mkdir data-nosuid/$i
sudo mount --bind data/$i data-nosuid/$i
sudo mount -o remount,nosuid data-nosuid/$i
```

## Global Resources

```sh
sysctl -w kernel.pty.max=1048576
```
