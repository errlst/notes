git 使用.int 文件作为配置文件，支持三层配置文件，优先级从高到低：

- `.git/config`，当前项目特定配置文件，使用 `--file` 选项修改。

- `~/.gitconfig`，当前用户的配置文件，使用 `--global` 选项修改。

- `/etc/gitconfig`，系统级别配置文件，使用 `--system` 选项修改。
