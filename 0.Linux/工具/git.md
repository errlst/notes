## 搭建 gitea 私有服务器

- 创建 git 用户

```shell
$ sudo adduser git
```

- 下载 gitea 二进制包

```shell
$ sudo wget -O /usr/local/bin/gitea https://dl.gitea.com/gitea/1.22.3/gitea-1.22.3-linux-amd64
$ sudo chmod +x /usr/local/bin/gitea
```

- 创建需要的工作目录

```shell

```

- 配置 systemd 服务文件

```shell
[Unit]
Description=Gitea
After=syslog.target
After=network.target

[Service]
RestartSec=3s
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/gitea/

ExecStart=/usr/local/bin/gitea web --config /etc/gitea/app.ini
Restart=always
Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea

[Install]
WantedBy=multi-user.target
```

## 指令

#### 分支

创建分支

```shell
$ git branch <分支名>
```

删除分支

```shell
$ git branch -d <分支名>
```

切换分支

```shell
$ git checkout <分支名>
```

查看分支

```shell
# 查看本地分支
$ git branch

# 查看远程分支
$ git branch -r

# 查看所有分支
$ git branch -a
```

合并分支

```shell
# 将目标分支合并到当前分支
$ git merge <分支名>
```
