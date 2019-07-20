---
layout: post
title: docker-ubuntu
date: 2019-07-20 15:33:38 +0800
category: Note
tag: Docker
---

* content
{: toc}
---

## Customize Image

### get Ubuntu

```bash
docker pull ubuntu
```

### create a container

```bash
docker run -it --name uos ubuntu /bin/bash
```

### update ubuntu

```bash
apt-get update
apt-get install vim curl git python3 python3-pip -y
```

### update ssh config

```bash
apt-get install openssh-server

vim /etc/ssh/sshd_config
# use these configs
PermitRootLogin yes
PubkeyAuthentication yes
AuthorizedKeysFile	.ssh/authorized_keys

/etc/init.d/ssh restart
```

### get ssh key from macOS

```bash
cat ~/.ssh/id_rsa.pub | pbcopy
```

### paste the key

```bash
mkdir ~/.ssh
vim ~/.ssh/authorized_keys
```

### quit container

`ctrl` + `d`

### commit to a new image

```bash
# get CONTAINER ID
docker ps -a

docker commit -m 'add ssh' -a 'weqopy' CONTAINER-ID ubuntu-ssh
```

## use new image

### create new container by the new image

```bash
docker run -d -p 22222:22 -v macOS_dir_path:/root/docker_share --name ussh ubuntu-ssh /usr/sbin/sshd -D


# start new container
ssh -p 22222 root@localhost
```

---
> [mac 下使用 Docker 搭建 ubuntu 环境](https://www.smslit.top/2018/12/20/docker_ubuntu_learn/)
