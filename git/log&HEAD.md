`git log` 可以获取提交日志，得到的信息类似：

```shell
commit ad29077b458d24f774215564834caf6f3d628f83 (HEAD -> main, origin/main, origin/HEAD)
Author: jin_zhou <purec2220713462@gmail.com>
Date:   Tue Feb 4 17:12:21 2025 +0800

    02-04

commit 94c84558fe810b33aeaa17c61e193a28eb1a7a77
Author: zhou_jin <xxx>
Date:   Fri Jan 17 17:38:50 2025 +0800

    01-17
```

日志包括 comment、date、user 和提交 id。

## HEAD

HEAD 表示当前活跃分支，当使用 `git switch` 切换分支时，HEAD 就会指向新的分支。有且时候 HEAD 会指向没有分支名字的版本，此时就叫做 detached HEAD。

## 切换分支

切换分支前，需要确保 `git status` 一切都是干净的。
