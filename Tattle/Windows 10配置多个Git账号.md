# Windows 10配置多个Git账号

## 清除全局配置

如果你已经全局配置过git账号，那么一定要先把全局的配置给清除掉

```
git config --global --list  // 看一下是否配置过user.name 和 user.email
git config --global --unset user.name // 清除全局用户名
git config --global --unset user.email // 清除全局邮箱
```

## 删除已经生成过的密钥

如果你已经全局配置过git账号，那么你一定生成过全局ssh密钥。 这个密钥通常在 ~/.ssh 目录下。可以全部删除了

## 生成新的密钥

设置`Gitee密钥`
```
$ ssh-keygen -t rsa -C "muba@qq.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/anzelin/.ssh/id_rsa): id_rsa_gitee 秘钥名称
Enter passphrase (empty for no passphrase): 输入gitee账号密码
Enter same passphrase again: 再次输入gitee账号密码
Your identification has been saved in id_rsa_gitee
Your public key has been saved in id_rsa_gitee.pub
The key fingerprint is:
SHA256:tT5SC8V7dtheyjufrokeufhckLyChSbIy8rjS/jRO4 muba@qq.com
The key's randomart image is:
+---[RSA 3072]----+
| o..    .        |
|... .  . +       |
|..+  .. . *      |
|=*o  ..  + +     |
|+Boo .. S + .    |
|=.= +.o  + o     |
|.O + o... +      |
|X .  .o  . .     |
|oE. ..           |
+----[SHA256]-----+
```

设置gitlab密钥，操作同上。

```
$ ssh-keygen -t rsa -C "muba@163.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/anzelin/.ssh/id_rsa): id_rsa_gitlab
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in id_rsa_gitlab
Your public key has been saved in id_rsa_gitlab.pub
The key fingerprint is:
SHA256:JW/BArcU1CkdesxyT4+KxRrthysiAk8JoSF0WevJ8zMi0 muba@163.com
The key's randomart image is:
+---[RSA 3072]----+
|o .++oB.o        |
|.o.+ +.X o       |
|+ o++ + = +      |
|==. +. . = .     |
|B+ *.   S o      |
|o * oo . .       |
| . oE *          |
|  .  + o         |
|                 |
+----[SHA256]-----+
```

## 添加公钥到远端仓库

在Gitub、Gitee中配置公钥

## 配置密钥管理文件

在.ssh目录下，创建config文件。执行 vim ~/.ssh/config

```
Host gitee.com
HostName gitee.com  踩坑的地方，别设置错了
User muba@qq.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_gitee

Host git.xxx.com
HostName git.xxx.com  踩坑的地方，别设置错了（公司仓库域名或IP）
User muba
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa_gitlab
```

- Host ---------远端仓库的别名，简写 例如： 配置后 git@gitee.com 可以写为 git@gitee
- HostName ---------远端仓库地址或IP（不支持配置端口）
- User ---------仓库上的用户名
- PreferredAuthentications ---------强制使用Publickey验证
- IdentityFile ---------指向仓库私钥的绝对路径
- Port ---------远端仓库端口，一般不需要设置，特殊情况除外

## 测试一下仓库连接

```
# gitee
$ ssh -T git@gitee.com
Hi 沐码! You've successfully authenticated, but GITEE.COM does not provide shell access.

# gitlab
$ ssh -T git@git.xxx.com
Welcome to GitLab, @muba!
```

## 单独指向设置

在最开始已删除全局配置，现在对每个仓库单独设置用户名信息。

```
$ git clone git@git.xxx.com:abc/efd/my-demo.git

$ cd my-demo

# 如果还没有设置用户账号信息，进行提交修改会报错
$ git commit -m 'update readme'
Author identity unknown

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got 'muba@LAPTOP-50L7DUOE.(none)')
```

针对仓库设置用户信息。

```
$ git config --local user.name "muba"

$ git config --local user.email "muba@163.com"
```

设置完可查看该仓库的配置信息
```
$ git config --local --list
```