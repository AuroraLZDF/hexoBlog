---
title: Docker安装gitlab
date: 2020-04-01 08:44:51
categories: [Docker]
tags: [Docker]
---

# 在Docker中运行 gitlab 镜像
```bash
sudo docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```
上面的命令将会下载最新的 `gitlab-ce` 镜像，并创建一个容器。同时开放了 `80、443、22` 端口。数据存储在 `/srv/gitlab/` 中。**容器将会跟随系统一起启动**。 

## Docker容器的重启策略如下：
- `no`，默认策略，在容器退出时不重启容器
- `on-failure`，在容器非正常退出时（退出状态非0），才会重启容器
- `on-failure:3`，在容器非正常退出时重启容器，最多重启3次
- `always`，在容器退出时总是重启容器
- `unless-stopped`，在容器退出时总是重启容器，但是不考虑在Docker守护进程启动时就已经停止了的容器
- `unless-stopped`，在容器退出时总是重启容器，但是不考虑在Docker守护进程启动时就已经停止了的容器

## 数据存放在哪里
`Gitlab` 容器使用主机绑定的 `volumes` 来存储数据：

|本地路径|容器路径|用途|
| --- | --- | --- |
|`/srv/gitlab/data`|`/var/opt/gitlab`|存储应用数据|
|`/srv/gitlab/logs`|`/var/log/gitlab`|存储日志|
|`/srv/gitlab/config`|`/etc/gitlab`|存储 `gitlab` 配置文件|

## 配置 Gtilab
```bash
# 进入容器
$ sudo docker exec -it gitlab /bin/bash
# 或者直接编辑配置文件
$ sudo docker exec -it gitlab editor /etc/gitlab/gitlab.rb
# 然后重启 gitlab
$ sudo docker restart gitlab
```
## 或者也可以在启动 Gitlab镜像时加入配置信息
可以通过环境变量 `GITLAB_OMNIBUS_CONFIG` 来添加配置信息：
```bash
sudo docker run --detach \
  --hostname gitlab.example.com \
  --env GITLAB_OMNIBUS_CONFIG="external_url 'http://gitlab.example.com/'; gitlab_rails['lfs_enabled'] = true;" \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

# 通过 `docker-compose` 安装 Gitlab
```bash
version: '2'

services:
    gitlab:
      image: 'gitlab/gitlab-ce:latest'
      restart: unless-stopped
      hostname: 'gitlab.example.com'
      environment:
        TZ: 'Asia/Shanghai'
        GITLAB_OMNIBUS_CONFIG: |
          external_url 'http://gitlab.example.com'
          unicorn['port'] = 80
          gitlab_rails['time_zone'] = 'Asia/Shanghai'
          # Add any other gitlab.rb configuration here, each on its own line
      ports:
        - '80:80'
        - '443:443'
        - '22:22'
      volumes:
        - '/home/xxxx/docker/gitlab/config:/etc/gitlab'
        - '/home/xxxx/docker/gitlab/logs:/var/log/gitlab'
        - '/home/xxxx/docker/gitlab/data:/var/opt/gitlab'
```

# 测试
- 安装完成之后，浏览器打开你自己定义的地址 `http://gitlab.example.com`,输入登陆密码：
![gitlab-login.png](/images/docker/gitlab-login.png)

- 添加一个测试项目：
![gitlab-new-project.png](/images/docker/gitlab-new-project.png)

到此，`Gitlab` 安装完成！