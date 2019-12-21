---
title: 操作系统包管理
date: 2019-12-21 18:25:54
tags: '操作系统'
categories: '使用说明'
permalink: package-management
---

# 软件包管理

## Linux

### Debian

#### 简介

Debian 软件包管理叫做 Advanced Packaging Tool(APT), 是一套管理软件包和相关依赖的机制, 可以实现应用程序的安装, 移除和更新等. APT 有很多的实现, 如 dpkg, apt-get 等.

软件源的格式通常是4部分:

`[archive type] [repository URL] [distribution] [component]`

* archive type 有两种, deb 表示二进制软件包, deb-src 表示源代码软件包.
* repository URL 表示仓库地址, 国内镜像站通常是 https://mirrors.xxx.xxx/debian  (如果是 ubuntu, 就替换 debian)
* distribution 表示发行版本的代号, 如 Debian 9是 stretch, Ubuntu 17.04是 zesty (来自完整的版本号 Zesty Zapus)

component 通常有 main, contrib 和 non-free 三类, 可以有多个, 空格分隔 (Ubuntu 中是 main, restricted, universe 和 multiverse)

Ubuntu PPA 源:

PPA (Personal Package Archives)介绍: 由 launchpad.net (Ubuntu 母公司 Canonical 架设) 提供的个人软件包集合 (非 Ubuntu 官方维护) , 允许用户建立自己的软件仓库, 也用于发布一些测试版本的软件, 因此可靠性上存在一定缺失.
使用 add-apt-repository 添加并更新列表后, 就可以使用 apt 安装了.
参考: [Ubuntu PPA 软件源的介绍与使用](https://link.jianshu.com/?t=http://blog.csdn.net/baidu_22502417/article/details/46683549)

#### 使用

apt  (apt-get/apt-cache/apt-config 的精简结合)

| apt 命令         | 等价的命令           | 命令的功能                          |
| :--------------- | :------------------- | :---------------------------------- |
| apt install      | apt-get install      | 安装软件包                          |
| apt remove       | apt-get remove       | 移除软件包                          |
| apt purge        | apt-get purge        | 移除软件包及配置文件                |
| apt update       | apt-get update       | 刷新存储库索引                      |
| apt upgrade      | apt-get upgrade      | 升级所有可升级的软件包              |
| apt autoremove   | apt-get autoremove   | 自动删除不需要的包                  |
| apt full-upgrade | apt-get dist-upgrade | 在升级软件包时自动处理依赖关系      |
| apt search       | apt-cache search     | 搜索应用程序                        |
| apt show         | apt-cache show       | 显示装细节                          |
| apt list         |                      | 列出包含条件的包 (已安装, 可升级等) |
| apt edit-sources |                      | 编辑源列表                          |

APT 包管理系统会将下载的 Deb 包缓存在 /var/cache/apt/archives

apt-get 在线安装/移除/升级 (需要使用 sudo 提升权限)

apt-get 是命令行形式的软件包管理工具. 常用命令:

| 命令                                   | 含义                            |
| :------------------------------------- | :------------------------------ |
| apt-get update                         | 更新本地包数据库列表            |
| apt-get upgrade                        | 升级包 (已安装的, 不会删除)     |
| apt-get dist-update                    | 升级包 (根据依赖会添加或删除包) |
| apt-get install packagename [p2 p3...] | 安装软件包                      |
| apt-get install -y 包名                | 无需提示, 默认参数直接安装      |
| apt-get remove 包名                    | 移除已安装的包                  |
| apt-get autoremove                     | 自动移除已知不需要的包/依赖     |
| apt-get autoclean                      |                                 |

apt-cache 软件包相关信息查询

| 命令                      | 含义                         |
| :------------------------ | :--------------------------- |
| apt-cache search 搜索内容 | 搜索软件包 (不需要完整名字)  |
| apt-cache show 包名       | 查看软件包的 (本地缓存) 信息 |

apt-config 用于读取 APT 配置文件

dpkg 本地.deb 包安装和管理

| 命令                  | 含义                                      |
| :-------------------- | :---------------------------------------- |
| dpkg -s 包名          | 显示软件包的安装状态 (-s, --status)       |
| sudo dpkg -i 包文件名 | 安装软件包, 包以.deb 结尾 (-i, --install) |
| dpkg --info 包名      | 查看软件包的信息                          |

aptitude: 同时包含文本模式界面和图形界面

其他:

* Gnome 中 software 软件中心
* Synaptic(新立得) 图形化软件
* GDebi 图形化安装本地.deb 包 `$ sudo apt install gdebi`

#### Ubuntu 镜像使用帮助

Ubuntu 的软件源配置文件是 `/etc/apt/sources.list`. 将系统自带的该文件做个备份, 将该文件替换为下面内容, 即可使用 TUNA 的软件源镜像.

```sh
# 默认注释了源码镜像以提高 apt update 速度, 如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse

# 预发布软件源, 不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```

### Red Hat Package Manager

#### 简介

RPM (RedHat Package Manager), RPM通过以一个数据库记录的方式来将你所需的软件安装到你的Linux系统上的. 在你所安装的软件前先通过编译完成, 打包成RPM格式的文件, 数据库记录的方式搜索对应需要具备的依赖关系的软件, 那么当你在安装该软件的时候, RPM会查看你系统环境和依赖性关系来判定你是否能安装此软件. 若能满足, 则允许安装. 否则将不给予安装. 并且在安装的时候将该软件的信息写入RPM的数据库中, 以便日后查询, 检验和升级. 这个软件安装方法是由Red Hat公司开发出来的, 由于非常的简单实用, 很多的distributions都使用这个机制来安装和管理软件. 例如: CentOS, SUSE等.

```sh
# RPM包的命名格式
bash-4.2.3-3.centos5.x86_64.rpm
# 表示bash-4.2.3, 第三次发行, 支持CentOS5系统, 支持硬件平台x86_64位系统
```

#### 打包工具的分包机制

假设一个程序有20个功能: 常用功能有8个, 特殊功能A: 3个, 特殊功能B: 6个, 二次开发相关功能: 3个. 那如果用户只需要常用功能, 可是必须要全部安装, 那么就会很占用空间, 而且其他功能根本不会使用, 这时就会分包机制了.

分包机制:

核心包 (主包) + 子包 (分包) 组成

核心包: 命令与源程序一致, 例如: `bash-4.2.3-3.centos7.x86_64.rpm`

子包: (安装子包前必须安装核心包), 例如: `bash-a-4.2.3-3.centos7.x86_64.rpm`

#### 使用

```sh
rpm [option] Package_file
# 安装
-i: install安装操作

-v: 安装时显示详细信息

-vv: 安装时显示更详细信息

-h: hash码, 在安装过程中使用#号来显示安装进度

–-test: 仅作测试, 不做安装操作

-–nodeps: 忽略依赖关系, 强制安装如果某包依赖于其他包, 要么解决所有的依赖关系, 要么忽略依赖关系, 强制安装. 但是如果强制安装完成后, 软件未必能正常使用.

–-replacepkgs: 重新安装程序包
备注: 如果原有配置文件作了修改, 很有可能不执行替换文件, 而是将新生成的配置文件重命名后缀为 .rpmnew

# 卸载
-e: erase 删除

–nodeps 忽略依赖关系

# 升级
-Uvh: 升级或安装 (如果有老版本就升级, 如果没有就安装)

-Fvh: 直接升级 (如果有老版本就安装新版本)
```

升级的时候也可能会出现版本冲突等问题, 所以如果想强制升级可以使用 --force

注意: 不应该对内核执行升级操作, 而是安装 (因为Linux系统允许多内核并存)

```sh
# 查询某包是否安装
rpm -q package_name

# 查询所有已经安装的包
rpm -qa #a表示all

# 查询包的表述信息
rpm -qi package_name

# 查询某包生成了哪些文件
rpm -ql package_name

# 查询某包生成了哪些配置文件
rpm -qc package_name

# 查询某包生成了哪些帮助文件
rpm -qd package_name

# 查询程序包的相关脚本
rpm -q –scripts package_name

# 查询某文件是由哪个包安装生成的
rpm -qf /path/to/some_file

# 查询某包所提供的capabilities
rpm -q provides PACKAGE_NAME

# 查询某包所依赖的capabilities
rpm -q --requires PACKAGE_NAME

# 对尚未安装的包执行查询
rpm [option] /path/to/package_file

-q  : 查看软件包是否安装

-qpi: 包的信息

-qpl: 安装以后会生成什么文件

-qpc: 安装以后会生成什么配置文件

-qpd: 安装以后会生成什么帮助文件

# 查询指定的CAPABILITY由哪个包所提供
rpm  -q --whatprovides CAPABILITY

# 查询指定的CAPABILITY被哪个包所依赖
rpm  -q --whatrequires CAPABILITY

# 查询某包制作时随版本变化的changelog信息
rpm -q --changelog PACKAGE_NAME

# 预览包内文件
rpm2cpio 包文件|cpio –itv   #需要制定包的路径

# 释放包内文件
rpm2cpio 包文件|cpio –id “ *.conf” #需要制定包的路径

# 校验(用于检查包装后文件属性是否发生变化)
rpm -V Package_name

S file Size differs # 大小

M Mode differs (includes permissions and file type) # 权限, 文件类型改变

5 digest (formerly MD5 sum) differs # md5校验码发生改变

D Device major/minor number mismatch # 如果是设备文件, 则主设备号和次设备号发生改变

L readLink(2) path mismatch # 路径发生改变

U User ownership differs # 属主发生改变

G Group ownership differs # 属组发生改变

T mTime differs # 修改时间发生变化

P caPabilities differ # 能力发生变化 (可以理解为功能)

# rpm 的数据库目录:  /var/lib/rpm

rpm –-initdb # 初始化如果事先没有库, 会新建一个；如果有, 则不覆盖

rpm –-rebuilddb # 重建直接重建, 覆盖原有的数据库
```

脚本有四类

* preinstall: 安装前脚本
* postinstall: 安装后脚本
* preuninstall: 卸载前脚本
* postuninstall: 卸载后脚本

#### CentOS 镜像使用帮助

首先备份 CentOS-Base.repo `sudo mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak`

之后启用 TUNA 软件仓库， 将以下内容写入 `/etc/yum.repos.d/CentOS-Base.repo`

```
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/updates/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/centosplus/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

更新软件包缓存 `sudo yum makecache`

## macOS

### Homebrew

#### 简介

macOS 缺失的软件包的管理器

使用 Homebrew 安装 Apple 没有预装但 [你需要的东西](https://formulae.brew.sh/formula/).

```sh
$ brew install wget
```

Homebrew 会将软件包安装到独立目录, 并将其文件软链接至 /usr/local .

```sh
$ cd /usr/local
$ find Cellar
Cellar/wget/1.16.1
Cellar/wget/1.16.1/bin/wget
Cellar/wget/1.16.1/share/man/man1/wget.1

$ ls -l bin
bin/wget -> ../Cellar/wget/1.16.1/bin/wget
```

Homebrew 不会将文件安装到它本身目录之外, 所以您可将 Homebrew 安装到任意位置.

轻松创建你自己的 Homebrew 包.

```sh
$ brew create https://foo.com/bar-1.0.tgz
Created /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/bar.rb
```

完全基于 Git 和 ruby, 所以自由修改的同时你仍可以轻松撤销你的变更或与上游更新合并.

```sh
# 使用 $EDITOR 编辑!
$ brew edit wget
```

Homebrew 的配方都是简单的 Ruby 脚本:

```ruby
class Wget < Formula
  homepage "https://www.gnu.org/software/wget/"
  url "https://ftp.gnu.org/gnu/wget/wget-1.15.tar.gz"
  sha256 "52126be8cf1bddd7536886e74c053ad7d0ed2aa89b4b630f76785bac21695fcd"

  def install
    system "./configure", "--prefix=#{prefix}"
    system "make", "install"
  end
end
```

Homebrew 使 macOS 更完整. 使用 gem 来安装 RubyGems, 用 brew 来安装那些依赖包.

"To install, drag this icon..." no more. brew cask installs macOS apps, fonts and plugins and other non-open source software.

```sh
$ brew cask install firefox
```

Making a cask is as simple as creating a formula.

```sh
$ brew cask create foo
Editing /usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask/Casks/foo.rb
```

#### 安装

```sh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

将以上命令粘贴至终端. 脚本会在执行前暂停, 并说明将它将做什么. 高级安装选项在 [这里](https://docs.brew.sh/Installation) (required for Linux and Windows Subsystem for Linux).

[更多文档](https://docs.brew.sh/)

#### 使用

查找安装卸载软件

* brew --version 或者 brew -v 显示 brew 版本信息
* brew install <formula> 安装指定软件
* brew unistall <formula> 卸载指定软件
* brew list  显示所有的已安装的软件
* brew search text 搜索本地远程仓库的软件, 已安装会显示绿色的勾
* brew search /text/ 使用正则表达式搜软件

升级软件

* brew update 自动升级 homebrew (从 github 下载最新版本)
* brew outdated 检测已经过时的软件
* brew upgrade  升级所有已过时的软件, 即列出的以过时软件
* brew upgrade <formula> 升级指定的软件
* brew pin <formula> 禁止指定软件升级
* brew unpin <formula> 解锁禁止升级
* brew upgrade --all 升级所有的软件包, 包括未清理干净的旧版本的包

清理

* brew cleanup -n 列出需要清理的内容
* brew cleanup <formula> 清理指定的软件过时包
* brew cleanup 清理所有的过时软件
* brew unistall <formula> 卸载指定软件
* brew unistall <fromula> --force 彻底卸载指定软件, 包括旧版本

Brew 更新软件很简单, 但是 brew cask 就没这么简单了.

```sh
# 安装 brew-cask-upgrade
brew tap buo/cask-upgrade
```

* brew cu 更新所有过时应用
* brew cu [CASK] 更新指定应用

#### Homebrew 镜像使用帮助

替换现有上游:

```sh
git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git

git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git

brew update
```

复原:

```sh
git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew.git

git -C "$(brew --repo homebrew/core)" remote set-url origin https://github.com/Homebrew/homebrew-core.git

git -C "$(brew --repo homebrew/cask)" remote set-url origin https://github.com/Homebrew/homebrew-cask.git

brew update
```

### Windows

#### 简介

用 Scoop 来安装和管理我们的软件:

* 集搜索, 下载, 安装, 更新软件于一体:极大的降低了安装维护一个软件的成本, 我们甚至不必在软件本身的复杂菜单中寻找那个更新按钮来更新软件自己
* 将软件干干净净的安装到电脑的用户文件夹下: 这样既不会污染路径也不会请求不必要的权限 (UAC)
* 在卸载软件的时候, 能够尽量清空软件在电脑上存储的任何数据和痕迹

特别的, Scoop 最适合安装那种干净, 小巧, 开源的软件.

#### 安装

安装 Scoop 很简单, 不过你需要先确定一些基础环境是否符合安装要求:

* Windows 版本不低于 Windows 7
* Windows 中的 PowerShell 版本不低于 PowerShell 3
* 你能 正常, 快速 的访问 GitHub 并下载上面的资源
* 你的 Windows 用户名为英文 (Windows 用户环境变量中路径值不支持中文字符)

在 PowerShell 中输入下面内容, 保证允许本地脚本的执行:

```sh
set-executionpolicy remotesigned -scope currentuser
```

然后执行下面的命令安装 Scoop:

```sh
iex (new-object net.webclient).downloadstring('https://get.scoop.sh')
```

静待脚本执行完成就可以了,

#### 使用

最常用的几个基础动作有这些:

| 命令      | 动作         |
| :-------- | :----------- |
| search    | 搜索软件名   |
| install   | 安装软件     |
| update    | 更新软件     |
| status    | 查看软件状态 |
| uninstall | 卸载软件     |
| info      | 查看软件详情 |
| home      | 打开软件主页 |

Scoop 和 Homebrew 对软件包安装位置有着相同的处理哲学:下载, 安装在用户文件夹下. 具体来讲:

* Scoop 在你的用户根目录 (一般是 C:\Users\用户名) 下创建了一个名为 scoop 的文件夹, 并默认将软件下载安装到这个文件夹下
* Scoop 将软件安装到一个相对隔离的环境下 (Each program you install is isolated and independent), 从而保证环境的统一和路径不被污染

scoop 文件夹下的 apps 存放有安装的所有应用. 值得一提的是: scoop 是通过 shim 来软链接一些应用, 这样的设计让应用之间不会互相干扰, 十分方便.

### 软件包管理哲学

在写这篇文章之前我也看了我派上面对包管理工具介绍的文章, 我觉得这些文章其实都没太讲清为什么我们需要用包管理这个看上去复杂难用的命令行工具去下载, 管理我们的软件. 毕竟现在的软件管理哲学是我去 App Store 下一个不就行了嘛.

需要明确的是:包管理的设计初衷是为了方便开发者管理和搭建开发环境. 用包管理工具能够快速的安装开发工具, 开发依赖, 从而免去复杂的路径, 环境变量等信息的配置. 而我们作为普通用户, 实际上用包管理工具的过程, 就是在借鉴这种软件管理哲学.

* 一行代码省去了搜索, 筛选, 下载等繁琐步骤
* 安装方便, 更新方便, 卸载也方便
* 同时也最大程度杜绝了流氓捆绑软件的安装 (因为 Scoop 本身和 Scoop 安装过程参考的配置文件都是开源的, 要安装什么一目了然)
* 这些都是传统的搜索 - 筛选 - 下载的软件管理过程带来的复杂过程和安全隐患的极佳解决方法.
