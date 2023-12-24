---
layout:     post
title:      "我的C++开发环境"
subtitle:   " \"Linux\""
date:       2023-12-25 12:00:00
author:     "EricHuang"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - c++
    - vim
---

本文旨在搭建一个纯净版的在linux平台的C++开发环境。安装过程中需要高速访问github，推荐一个梯子[fastlink]([首页 — FASTLINK](https://fl20230821.fastlink.la/user))

# 基础镜像

以大部分服务器常用的centos7为基础，逐步安装所需要的第三方组件

```shell
docker pull centos:7
```

在基础镜像之上安装`git`, `ssh`,`curl`等

````shell
yum install -y
yum install -y ssh vim unzip tar git curl wget cmake  \
        libuuild-devel \
        readline-devel \
        tk-devel       \
        libffi-devel ncurses-libs sqlite-devel bzip2-devel openssl-devel gdbm-devel libdbi-devel libaio-devel zlib-devel \
        xz-devel python-backports-lzma lrzsz git-lfs epel-release centos-release-scl
yum install -y devtoolset-8 && echo "source /opt/rh/devtoolset-8/enable" >> /etc/bashrc
yum install -y epel-release && yum install -y scons
````

# 安装python3

使用源码编译的方式，为了支持后续vim的升级，以so的方式安装，首先从[pythopn官网](https://www.python.org/)拉取源文件，以python3.8.17为例。

```shell
mkdir ~/thirdparty && cd ~/thirdparty
wget https://www.python.org/ftp/python/3.8.17/Python-3.8.17.tgz
tar -zxvf Python-3.8.17.tgz && cd Python-3.8.17
./configure --enable-shared
make -j8 && make install # 默认安装路径在/usr/local
```

由于安装的是so,执行pip3可能会出现找不到lib的情况`/usr/local/bin/python3.8: error while loading shared libraries: libpython3.8.so.1.0: cannot open shared object file: No such file or directory`，需要设置下动态库的查找路径

```shell
echo "export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH" >> ~/.bashrc && source ~/.bashrc
pip3 install --upgrade pip
```

# 安装vim9

```shell
cd ~/thirdparty
wget https://github.com/vim/vim/archive/refs/tags/v9.0.2133.tar.gz
tar -zxvf v9.0.2133.tar.gz && cd vim-9.0.2133
./configure --with-features=huge \
            --enable-multibyte \
            --enable-python3interp=yes \
            --with-python3-config-dir=/usr/local/lib \
            --enable-gui=gtk2 \
            --enable-cscope \
            --prefix=/usr/local \
            --enable-fail-if-missing
make -j8 && make install
echo "export PATH=/usr/local/bin:$PATH" >> ~/.bashrc && source ~/.bashrc
```

# 安装CMAKE

可以直接从github下载以及编译好的bin

```shell
cd ~/thirdparty
wget https://github.com/Kitware/CMake/releases/download/v3.27.9/cmake-3.27.9-linux-x86_64.tar.gz
tar -zxvf cmake-3.27.9-linux-x86_64.tar.gz
echo "export PATH=~/thirdparty/cmake-3.27.9-linux-x86_64/bin:$PATH" >> ~/.bashrc && source ~/.bashrc
```

# 安装LLVM

安装llvm主要是为了安装clangd以支持vim下的C++代码补全

```shell
cd ~/thirdparty
wget https://github.com/llvm/llvm-project/archive/refs/tags/llvmorg-17.0.6.tar.gz
tar -zxvf llvmorg-17.0.6.tar.gz && cd llvm-project-llvmorg-17.0.6
mkdir build &&  cd build
cmake ../llvm  -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra;compiler-rt"
make -j8 && make install
```

# 安装vimplus

使vim用起来像IDE一样好用

```shell
cd ~/thirdparty
git clone https://github.com/chxuan/vimplus.git && cd vimplus
./install.sh # 安装时根据弹窗提示，选择python3
```

编译YCMpython3 ./install.py --clang-completer  --system-libclang

```shell
cd ~/.vim/plugged/YouCompleteMe/
python3 ./install.py --clang-completer  --system-libclangvim
```

在`~/.vimrc`中加入   

```shell
let g:ycm_server_python_interpreter = '/usr/local/bin/python3'
let g:ycm_global_ycm_extra_conf = '~/.vim/plugged/YouCompleteMe/third_party/ycmd/.ycm_extra_conf.py'
let g:ycm_confirm_extra_conf = '~/.vim/plugged/YouCompleteMe/third_party/ycmd/.ycm_extra_conf.py 
```



   let g:ycm_confirm_extra_conf = '~/.vim/plugged/YouCompleteMe/third_party/ycmd/.ycm_extra_conf.py 

配置此ycm_extra_conf文件可以设置自动补全时的查询路径

# 配置命令行

```
alias ll='ls -l'
# 配置提示栏颜色
c_1="\[\e[0m\]"
c0="\[\e[30m\]"
c1="\[\e[31m\]"
c2="\[\e[32m\]"
c3="\[\e[33m\]"
c4="\[\e[34m\]"
c5="\[\e[35m\]"
c6="\[\e[36m\]"
c7="\[\e[37m\]"
export PS1="$c5[\u@$c5\h:$c5\w:$c6\t$c5]\n\$$c_1"
TZ='Asia/Shanghai';
export TZ
alias ls="ls --color=auto"
```

