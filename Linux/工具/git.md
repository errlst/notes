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

#### 创建分支

`git branch <branch>`

#### 删除分支

`git branch -d <branch>`

#### 切换分支

`git checkout <branch>`

#### 查看分支

`git branch`，查看本地分支。

`git branch -r`，查看远程分支。

`git branch -a`，查看所有分支。

#### 合并分支

`git merge <branch>`，将目标分支合并到当前分支。

#### 取消合并

`git merge --abort`，取消当前正在进行中的合并，并恢复到之前的状态。

#### 取消 commit

`git reset --soft HEAD^`，撤回上次 commit，且不修改当前内容。

`git reset --hard HEAD^`，撤回上次 commit，且修改当前内容。

#### 切到指定 commit

`git log`，查看提交历史，获取指定 commit 的哈希值。

`git checkout <hash>` 切换到指定哈希的 commit。

#### 创建空白分支

`git checkout --orphan <branch>`，创建没有任何提交记录的空白分支。

如果合并空白分支，此时会报错 "拒绝合并无关分支"，此时加上 `--allow_unrelated_histories` 参数允许合并。

> `git merge <branch> --allow_unrelated_histories`。
