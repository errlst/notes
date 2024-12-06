[toc]



# 配置文件

nvim的相关配置文件路径在`~/.config/nvim/`目录下，该目录需要手动创建。

`~/.config/nvim/init.lua`是nvim中lua相关的入口文件。



# 插件管理

使用[packer.nvim](https://github.com/wbthomason/packer.nvim)作为插件管理工具。

## 安装packer

```shell
> git clone --depth 1 https://github.com/wbthomason/packer.nvim\
~/.local/share/nvim/site/pack/packer/start/packer.nvim
```

## 常用命令

`:PackerCompile`，将packer配置文件（通常为`~/.config/nvim/lua/plugins.lua`）编译为效率更高的形式（每次修改配置后都需要执行）。

`:PackerInstall`，安装缺失的插件。

`:PackerClean`，移除未使用的插件。

`:PackerSync`，更新插件。

`:PackerUpdate`，clean，然后update和install。

## 配置文件

```lua
return require('packer').startup(function(use)
  -- 通过use添加插件
end)
```

其中`use`是接受一个参数的函数，规范插件的字符串或表，如果是表，其主要结构如下：

```lua
{
    'myusername/example',        -- 插件url
    -- 可选参数
    disable = boolean,           -- 是否禁用插件
    as = string,                 -- 插件别名
    after = string or list,      -- 该插件之前需要加载的插件
    requires = string or list,   -- 该插件的依赖项
    master = string,             -- 指定分支
    opt = boolean,               -- 将插件标记为可选（懒加载）
    lock = boolean,              -- 跳过更新插件（依然会被clean）
    config = string or function, -- 插件加载完成后的回调
    setup = string or function,  -- 插件加载前的回调（等待其它条件满足）
    -- 以下参数默认opt=true
    cmd = string or list,        -- 加载插件的指令
    event = string or list,      -- 加载插件的事件
}
```

## 常用插件

#### 中文帮助文档

仓库：https://github.com/yianwillis/vimcdoc。

#### github主题

仓库：https://github.com/projekt0n/github-nvim-theme。

#### coc代码补全

仓库：https://github.com/neoclide/coc.nvim。

使用`release`分支，已构建。

##### 插件管理

`:CocList extensions`管理已安装的拓展。

###### 安装插件

方法一：配置`vim.g.coc_global_extensions`表后，启动coc时自动安装（不会自动卸载）。

方法二：`:CocInstall xxx`手动安装插件。

###### 更新插件

`:CocUpdate`。

###### 搜索插件

方法一：安装coc-marketplace插件后，通过命令`:CocList marketplace`预览。

方法二：https://www.npmjs.com/搜索coc。

##### 配置文件

命令`:CocConfig`可以快速打开coc的用户配置文件。





# 相关仓库

https://github.com/neovim/neovim，nvim。

https://github.com/rockerBOO/awesome-neovim，整理了流行的nvim插件。

https://github.com/glepnir/nvim-lua-guide-zh，nvim中使用lua的指南。

