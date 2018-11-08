# Git 的学与记：git 配置 <!-- omit in toc -->

> 使用版本说明：
>
> - git: 2.19.0

---

- [安装 git](#安装-git)
- [配置 ssh key](#配置-ssh-key)
- [ssh-keygen 使用](#ssh-keygen-使用)
- [配置多个 ssh key](#配置多个-ssh-key)
- [配置 git config](#配置-git-config)

---

## 安装 git

贴个链接，简单粗暴，本人也没啥料可讲的，依个人喜好安装即可 :facepunch:

https://www.atlassian.com/git/tutorials/install-git

## 配置 ssh key

一般来说，拉取服务器 git 仓库到本地，或提交本地仓库到 git 服务器，都需要用到 ssh key 来进行验证。典型的就是 GitHub、GitLab。在配置前，可以先检查一下是否已经配置过了，方式如下：

密钥默认保存在 `~/.ssh` 文件夹下面，查看该文件夹下面是否有类似于 `id_dsa`，`id_dsa.pub` 文件的存在。如果存在，表示之前已经生成过一次 ssh key 了。`.pub` 表示公钥，另一个文件表示私钥。

```shell
$ cd ~/.ssh
$ ls
authorized_keys2  id_dsa       known_hosts
config            id_dsa.pub
```

紧接着上步，如果没有发现存在 ssh key，那么通过 ssh-keygen 去生成一对密钥。

```shell
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (~/.ssh/id_rsa):
Created directory '~/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in ~/.ssh/id_rsa.
Your public key has been saved in ~/.ssh/id_rsa.pub.
```

在终端输入 `ssh-keygen` 命令后，首先会提示你输入保存密钥的文件位置。这里，可以直接回车来使用默认的文件位置以及默认的密钥文件名，即 `~/.ssh/id_rsa`。也可以输入具体的文件路径，并修改密钥文件名称。 提示如下：

```shell
Enter file in which to save the key (~/.ssh/id_rsa):
```

然后会提示你输入密码口令，用于使用密钥时作为验证的手段。不过，同样可以两次回车跳过表示不需要口令。否则的话，每次使用密钥的时候，都可能会提示你输入口令。

最后，通过上一步生成了 ssh key，需要使用它，我们需要把公钥 `id_rsa.pub` 文件内容复制到发送给 git 服务器，例如 GitHub。

```shell
pbcopy < ~/.ssh/id_rsa.pub
```

这里同样简单粗暴附上 GitHub 和 GitLab 的使用方式：

https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/

https://docs.gitlab.com/ee/ssh/

## ssh-keygen 使用

如果你看过上面的两个链接里面的教程，会发现使用 ssh-keygen 的命令是带着参数的。

```shell
ssh-keygen -t rsa -C "your.email@example.com" -b 4096
```

这里使用了三个参数 -t，-C 和 -b。

- `-t`: 指定生成密钥文件的方式，值有 dsa | ecdsa | ed25519 | rsa。可以发现，不用这个参数的话，默认是 rsa。并且，需要知道的是，这个值和生成的密钥文件名称是相关的，使用 rsa，默认生成的密钥文件就是 `id_rsa`，如果是选择的是 dsa，那么默认生成密钥文件就是 `id_dsa` 了。
- `-C`: 生成密钥时的注释，会自动添加到公钥文件末尾 `id_rsa.pub` 里面。不设置也有一个默认值 `系统用户名.local`，设置的话一般设置为你的邮箱。
- `-b`: 指定密钥文件多少位数，默认是 2048，可以通过 `ssh-keygen -l -f .ssh/id_rsa` 查看信息。

> :information_source: 可以输入命令 `ssh-keygen --help` 来查看可用的参数信息。例如修改密码口令 ssh-keygen -P old_passphrase -n new_passphrase。

## 配置多个 ssh key

假设有这样的使用场景：我有一个 GitHub 账号，一个公司 GitLab 的账号。那么，怎么分别建立对应的 ssh key 呢？

这样的场景很常见，往往我们需要一个自己的 git 账号来提交自个的代码，如 GitHub；需要一个公司的 git 账号来提交代码到公司的 git 服务器上。这样的话，我们就需要建立两个 ssh key 了。

首先，我们通过前面讲到的 ssh-keygen 来生成两个密钥：

```shell
$ ssh-keygen -f .ssh/id_rsa_github
....

$ ssh-keygen -f .ssh/id_rsa_gitlab
```

然后，通过 ssh-add 命令添加生成的密钥文件：

```shell
$ ssh-add ~/.ssh/id_rsa_github
$ ssh-add ~/.ssh/id_rsa_gitlab
```

以防万一，可以使用以下命令删除之前添加过的 ssh key：

```shell
$ ssh-add -D
```

其次，我们需要配置 `.ssh/config` 这个文件，主要是配置每个 ssh key 的 host 域名以及对应的私钥文件的位置。例如：

```shell
# github
Host github.com
  HostName github.com
  User yourname
  IdentityFile ~/.ssh/id_rsa_github

# gitlab
Host git.yourcompany.com
  HostName git.yourcompany.com
  Port 2222
  User yourname
  IdentityFile ~/.ssh/id_rsa_gitlab
```

其中，需要配置起始的 `Host` 这个参数，输入对应 git 服务器的地址，例如 GitHub 的是 `github.com`，又或者是你公司 GitLab 地址 `git.yourcompany.com`。

> :question: 实在不太清楚 Host 和 HostName 这两者的区别。一般我都是填写同样内容。若有人明白的话，请告知 :pray:

然后，`User` 这个参数可以不配置，一般配置为你的账号名称；`Port` 这个参数根据实际情况填写，比如你公司 GitLab 的地址有着对应的端口号。

接着，最重要的是配置 `IdentityFile` 这个参数，对应的是你私钥文件的位置。

最后，可以通过以下命令来测试是否成功：

```shell
$ ssh-keygen -T git@github.com
$ ssh-keygen -T gitlab@git.yourcompany.com
```

## 配置 git config

git 有三种类型/位置的配置文件，分别是 `system`、`global` 和 `local`，优先级从左到右由低到高：

- system: 系统类型的配置文件。应该是在安装 git 之后就会存在的配置文件，文件位置据说是在 `/etc/gitconifg`。
- global: 全局类型的配置文件。全局配置时方便用到，文件位置在 `~/.gitconfig`。
- local: 本地仓库的配置文件。为某一个仓库配置特殊配置时用到，文件位置在仓库的 `.git/config`。

可以通过以下命令查看配置内容：

```shell
# 包含所有的配置，会展示多项相同的配置，最后一项会覆盖前面相同的配置
$ git config --list
# 查看系统类型的配置
$ git config --list --system
# 查看全局类型的配置
$ git config --list --global
# 查看本地配置
$ git config --list --local
```

可以通过以下命令查看某一项配置的值（先从优先级高的配置文件检索，找到匹配的配置项后终止检索）：

```shell
$ git config --get user.name
$ git config --get user.email
```

可以通过以下命令设置配置项：

```shell
git config user.name "yourname"
git config --global user.name "yourname"
```

最后，介绍一项有趣的配置：git 别名（alias），通过别名配置，可以把长长的 git 命令，设置成你喜欢的短短的顺口的 git 命令。比如个人很喜欢的一项别名配置：

```shell
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

通过上面的别名配置，我就可以很方便的通过 `git lg` 来查看分支提交记录了，爽歪歪~

> :information_source: 通过 `git config --help` 查看更多的配置命令（不过个人实在看的懵逼）

> :question: 我们一般习惯通过 git config 全局配置 user.name 和 user.email，但如果在配置了多个 ssh key 的情况下，应该怎么来方便区分？比如 Github 提交时显示 GitHub 账号的邮箱和名称；而 GitLab 提交时显示个人公司的邮箱和名称？
