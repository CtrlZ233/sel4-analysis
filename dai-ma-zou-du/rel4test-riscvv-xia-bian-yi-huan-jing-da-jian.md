---
description: Rel4环境配置和源码部署帮助文档
---

# reL4test RISCV-V下编译环境搭建

## 背景工具

### Terminal

在本项目中优于PowerShell的终端软件，在Microsoft商店中搜索下载即可。

### Ubuntu

本项目采用Ubuntu 22.04.3 LTS，在Microsoft商店中搜索下载即可用于WSL。

### WSL

* 优势：可以直接访问主机的软硬件资源，不容易出现网络问题
* 使用方法：默认开启，安装Ubuntu后在Terminal中打开Ubuntu即可
* 补充：
  * WSL迁移到非系统盘（WSL默认安装在C盘，可能导致空间不足问题）：[教程](https://blog.csdn.net/m0\_37605642/article/details/127812965)
  * 修改root密码（在后续使用中可能会遇到root默认密码未知的问题）：[教程](https://baijiahao.baidu.com/s?id=1751797503085666267\&wfr=spider\&for=pc)

### 网络

为终端配置网络（每次新开终端都需要配置）

配置步骤：

1. Windows下在cmd输入ipconfig
2. 查看WLAN下IPV4地址，如10.62.58.178
3. 在Ubuntu下输入（需要将127.0.0.1替换为上述IPV4地址）：

```sh
export http_proxy=http://127.0.0.1:7890
export https_proxy=$http_proxy
```

测试方法：在Ubuntu下输入如下代码可以获取对应http内容

```sh
curl -I www.google.com
```

### VSCode编辑器

* 安装方法：在Windows下安装VSCode并配置WSL组件
* 环境配置
  1. 在Ubuntu下安装Rust(安装选项选择default)
  2. 在VSCode安装rust-analyzer组件
* Rust安装Shell指令

```sh
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

## SeL4运行环境配置

* 按照 [Host Dependencies | seL4 docs](https://docs.sel4.systems/projects/buildsystem/host-dependencies.html#python-dependencies) 安装相应依赖（内容如下）
* 依赖内容
  * Repo
  * Base Dependencies下所有内容
  * Python Dependencies下所有内容
* Tips：如果在安装过程中缺失前置依赖，则安装对应依赖即可

## Qemu模拟器

#### 克隆仓库

```
git clone https://github.com/rel4team/qemu.git
```

#### 创建qemu-build文件夹

```
mkdir qemu-build
```

#### 配置

```sh
./configure --target-list=riscv64-softmmu
```

#### 安装（TODO）

```
// Some code
```

## 交叉编译工具

* 基本安装方法

按照 [Host Dependencies | seL4 docs](https://docs.sel4.systems/projects/buildsystem/host-dependencies.html#python-dependencies) 安装“Cross-compiling for RISC-V targets”部分

潜在问题：在make linux时出现Building GCC requires GMP 4.2+, MPFR 3.1.0+ and MPC 0.8.0+问题（暂时无法解决，安装上述以来后仍然报错）

* 备选方法（适用于上述方法无法跑通）

在Ubuntu下载解压已经完成编译的工具后，在/bin目录下输入

```sh
./riscv64-unknown-linux-gnu-gcc -v
```

将/bin目录加入环境变量（写入\~/.bashrc）

```sh
export PATH="文件位置/bin:$PATH"
```

输入如下内容同步环境变量

```sh
source ~/.bashrc
```

## Rel4运行环境配置

### 基本配置

#### **克隆仓库**

```sh
mkdir rel4test && cd rel4test
repo init -u https://github.com/rel4team/sel4test-manifest.git
repo sync
```

#### **配置环境**

```sh
cd ./rel4_kernel
make env
```

### 构建build.py参数命令解释

* \-b：编译c侧接口
* \-u：开启用户态中断功能
* \-c：开启cpu多核支持，可以输入数字表示cpu核数
* \-i：表示install，安装部分头文件内容，生成可执行二进制文件

### C侧代码运行

#### **构建编译：**

进入rel4test/rel4\_kernel输入命令

```sh
./build.py -c 4 -u
```

#### **仿真运行：**

进入rel4test/rel4\_kernel/build输入如下命令

```sh
./simulate -b qemu-build路径/qemu-system-riscv64 -M virt --cpu-num 4
```

其中：

* \-b：指定qemu路径，后续qemu路径需根据以往的qemu配置更改
* \-M：指定虚拟机器类型
* \--cpu-num：指定cpu核心数量

### Rust侧代码运行

#### **克隆仓库**

在rel4test/projects下递归克隆仓库

```sh
git clone --recurse-submodules https://github.com/rel4team/rust-root-task-demo.git
```

#### **头文件安装**

编译部分C侧头文件供Rust侧内核调用，进入rel4test/rel4\_kernel输入命令

```sh
./build.py -c 4 -u -i
```

#### **获取安装路径**

进入rel4test/kernel/install目录输入pwd（获得路径例如）

```
/home/xxx/workspace/rel4test/kernel/install
```

#### **配置环境变量**

将路径写入\~/.bashrc

```sh
export SEL4_INSTALL_DIR=/home/xxx/workspace/rel4test/kernel/install
export SEL4_PREFIX=/home/xxx/workspace/rel4test/kernel/install
```

#### **同步环境变量**

输入如下命令同步

```sh
source ~/.bashrc
```

#### **验证**

输入命令后可以确认已将路径加入环境变量

```sh
env
```

#### **编译**

在rel4test/rel4\_kernel下输入

```
./build.py -c 4 -u -r
```

#### **运行**

进入rel4test/rel4\_kernel/build输入如下命令

```sh
./simulate -b qemu-build路径/qemu-system-riscv64 -M virt --cpu-num 4
```

### 代码修改后重新编译问题

* 修改用户态代码：重新编译运行即可
* 修改内核态：删除build目录下kernel，重新编译运行即可

## 其他问题（不断更新中）

* Permission Denied：不断尝试或使用chmod指令获取更多权限
