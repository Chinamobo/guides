Chinamobo 服务器管理手册
======

**目录**

* [服务器初始配置](#init)

[服务器初始配置](id:init)
----

### [更新软件包](id:yum-update)

```
yum upgrade
yum install git
reboot
```

### [数据盘挂载](id:mount)

* 阿里云额外数据盘挂载教程 http://help.aliyun.com/view/11108189_13435365.html

### 安装 Ruby

```
curl -L https://get.rvm.io | bash -s stable

```

*TODO*

### 初始权限配置

注意：
> 用户权限必须用用户组分配，严禁给单一用户定义权限

创建用户组

```
groupadd site-admin
groupadd developer
```

使用 `visudo` 命令，修改组权限。在文件结部加入以下内容：

```
# 以下是我们的权限定义

# 站点高级管理员
%site-admin   ALL=(ALL)   ALL

# 站点维护


# 开发人员
%developer    ALL = /sbin/service, /sbin/chkconfig

```

编辑完成后务必检查格式：

```
visudo -c
```

### 分配用户

```
# 设置变量
MUser=用户名
MUserPassword=密码
# 创建用户
useradd --comment "姓名" --no-user-group --groups 所属组[,其他所属组] --create-home $MUser
# 设置密码
echo $MUser:MUserPassword | sudo chpasswd
# 强制登录时修改
passwd -e $MUser
```

分配用户后使用邮件通知指定人员，邮件模版：

> 你在 *服务器名* 的账号已准备好。
> 
> 服务器公网 IP 是 *123.123.123.123*
> 
> 用户名是你的用户名全拼，初始密码是 **mobo你的用户名全拼**，首次登录后你需要修改密码，强度过弱的密码会被拒绝，请使用足够强度的密码，密码修改后需要重新登录
> 
> 关于登入的相关问题见：https://github.com/Chinamobo/guides/blob/master/server/user_login.md
> 有其他问题联系请管理员。


### 工具软件安装

#### zsh
yum 上的版本太低了，需要从源码编译安装

```
sudo yum install ncurses-devel
curl -L http://sourceforge.net/projects/zsh/files/zsh/5.0.5/zsh-5.0.5.tar.bz2/download | tar xj
cd zsh-5.0.5
./configure && sudo make install
# 编译好的 zsh 执行文件在 Src/zsh
```
