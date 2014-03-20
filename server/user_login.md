服务器登入指导
=====

首次登入
----
ssh 首次登入后，你可能需要修改密码，强度过弱的密码会被拒绝，请使用足够强度的密码，密码修改后需要重新登录。


不使用密码登录
----

每次输入密码的话麻烦，ssh 可以使用公私钥验证的方式登录，免去了输入密码的麻烦，而且更安全。强烈推荐设置一下。

原理：将本地的 ~/.ssh/id_rsa.pub 内容添加到服务器对应用户的 ~/.ssh/authorized_keys 下，即可。

如果提示权限错误，使用下列命令修复

```
chmod 700 ~/.ssh
chmod 640 ~/.ssh/authorized_keys
```


使用 zsh
----

长时间在命令行下工作，没理由不换一个更好用的工具。

```
# 安装 oh my zsh
curl -L http://install.ohmyz.sh | sh

# 切换 login shell
chsh -s $(which zsh)
```

另见：https://beta.wikiversity.org/wiki/Topic:shell/Zsh