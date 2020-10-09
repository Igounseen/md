#### 1. conda 概述

anaconda 是一个包管理软件

#### 2. conda 安装

​		如果用户从来没有使用过conda config 命令，就不会有配置文件，当用户第一次运行 `conda config`命令时，将会在用户的家目录创建该文件，即一个名为**.condarc**的文本文件

- windows:  `C:\users\username\.condarc`
- linux: `/home/username/.condarc`

`.condarc`配置文件，是一种可选的（optional）运行期配置文件，其默认情况下是不存在的，但当用户第一次运行 `conda config`命令时，才会在用户的家目录创建该文件。我可以通过conda config 命令来配置该文件，也完全可以自己手动编辑也可以。

