---
title: Python 多版本管理之 pyenv
date: 2017-10-23 14:22:32
categories: Python
tags: 
 - Python
permalink: python-versions-pyenv
---
## 序

{% asset_img python-pyenv-logo.png [python-pyenv-logo] %}

经常遇到这样的情况：

*   系统自带的 Python 是 2.6，自己需要 Python 2.7 中的某些特性；
*   系统自带的 Python 是 2.x，自己需要 Python 3.x；

此时需要在系统中安装多个 Python，但又不能影响系统自带的 Python，即需要实现 Python 的多版本共存。[pyenv](https://github.com/yyuu/pyenv) 就是这样一个 Python 版本管理器。

## 安装 Pyenv

在终端执行如下命令以安装 pyenv 及其插件：

```
$ curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
```
安装完成后，根据提示将如下语句加入到 `~/.bashrc` 中:

```
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"   # 这句可以不加
```
然后重启终端即可。

## 安装 Python

```
$ pyenv install --list
```
该命令会列出可以用 pyenv 安装的 Python 版本。列表很长，仅列举其中几个:

```
2.7.8   # Python 2 最新版本
3.6.3   # Python 3 最新版本
anaconda2-4.1.0  # 支持 Python 2.6 和 2.7
anaconda3-4.1.0  # 支持 Python 3.3 和 3.4
```
其中 2.7.8 和 3.6.3 这种只有版本号的是 Python 官方版本，其他的形如 `anaconda2-4.1.0` 这种既有名称又有版本后的属于 “衍生版” 或发行版。

### 安装 Python 的依赖包

在编译 Python 过程中会依赖一些其他库文件，因而需要首先安装这些库文件，已知的一些需要预先安装的库如下。

**在 CentOS/RHEL/Fedora 下:**

```
sudo yum install readline readline-devel readline-static
sudo yum install openssl openssl-devel openssl-static
sudo yum install sqlite-devel
sudo yum install bzip2-devel bzip2-libs
```

**在 Ubuntu下：**

```
sudo apt-get update
sudo apt-get install make build-essential libssl-dev zlib1g-dev
sudo apt-get install libbz2-dev libreadline-dev libsqlite3-dev wget curl
sudo apt-get install llvm libncurses5-dev libncursesw5-dev
```
### 安装指定版本

用户可以使用 `pyenv install` 安装指定版本的 python。

```
pyenv install 3.6.3
## ... 安装信息
```
安装过程中，若出现编译错误，通常是由于依赖包未满足，需要在安装依赖包后重新执行该命令。

### 更新数据库

在安装 Python 或者其他带有可执行文件的模块之后，需要对数据库进行更新：

```
$ pyenv rehash
```

### 查看当前已安装的 python 版本

```
$ pyenv versions
* system (set by /home/seisman/.pyenv/version)
anaconda3-4.1.0
```

其中的星号表示当前正在使用的是系统自带的 python。

### 设置全局的 python 版本

```
$ pyenv global 3.6.3
$ pyenv versions
  system
* 3.6.3 (set by PYENV_VERSION environment variable)
```
当前全局的 python 版本已经变成了 3.6.3。不过日常使用推荐使用 `pyenv local` 或 `pyenv shell`临时改变 python 版本。

### 确认 python 版本

```
$ python
Python 3.6.3 (default, Oct 23 2017, 12:15:23)
[GCC 4.4.7 20120313 (Red Hat 4.4.7-17)] on linux
Type "help", "copyright", "credits" or "license" for more information
>>>
```

## 使用 python

*   输入 `python` 即可使用新版本的 python；
*   系统自带的脚本会以 `/usr/bin/python` 的方式直接调用老版本的 python，因而不会对系统脚本产生影响；
*   使用 `pip` 安装第三方模块时会自动按照到当前的python版本下，不会和系统模块发生冲突。
*   使用 `pip` 安装模块后，可能需要执行 `pyenv rehash` 更新数据库；

## pyenv 其他功能

1.  `pyenv uninstall` 卸载某个版本
2.  `pyenv update` 更新 pyenv 及其插件

## 参考

1.  [https://github.com/yyuu/pyenv](https://github.com/yyuu/pyenv)



