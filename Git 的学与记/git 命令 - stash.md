# Git 的学与记：git 命令 - stash <!-- omit in toc -->

> 使用版本说明：
>
> - git: 2.19.0

---

- [使用场景](#使用场景)
- [常用命令速记](#常用命令速记)

---

## 使用场景

在某条 feature 分支上开发，突然遇到一个线上的紧急 bug，需要立马切到 bugfix 分支解决，而此时 feature 分支上的代码功能并没有完成。此时，可以通过 `git stash` 命令存储当前分支上的修改到本地，然后切到 bugfix 分支解决线上 bug；解决完后再切回来继续开发 feature 分支的功能。

```shell
$ git stash
$ git checkout bugfix
$ git commit -am "..."
$ git checkout feature
$ git stash pop
```

## 常用命令速记

- `git stash`: 存储当前分支上的修改，包括删除和修改的，但默认不包括 untracked files，即新增的文件。
- `git stash -u`: 作用同上，但会包括新增的文件。参数 `-u` 是 `--include-untracked` 的简写。
- `git stash push`: 作用同 `git stash`，不过此命令可以带上可用的参数，比如参数 `-m`（`--message` 的简写）。
- `git stash save`: 作用同上。
- `git stash list`: 查看已存储的 stash。
- `git stash apply`: 恢复最近一次存储，不过恢复后此 stash 依旧会被保存。
- `git stash apply stash@{n}`: 恢复最近第 n 次的存储。`git stash apply stash@{0}` 等价于上一条命令。同样恢复后此 stash 不会被删除。
- `git stash show`: 查看最近一次存储内容。
- `git stash show stash@{n}`: 查看最近第 n 次的存储内容。
- `git stash drop`: 删除最近一次存储。
- `git stash drop stash@{n}`: 删除最近第 n 次的存储内容。
- `git stash pop`: 等价于 `git stash apply`，后 `git stash drop`。
- `git stash pop stash@{n}`: 等价于 `git stash apply stash@{n}`，后 `git stash drop stash@{n}`。
- `git stash clear`: 清除所有的存储。
- `git stash --help`: 查看当前 stash 可用的命令。

单纯的一个 `git stash` 命令和 `git stash push`，`git stash save` 这两个的区别就是后者它们多了一个参数选项的功能。例如：`git stash push -m "xxx"` 可以手动设置 stash 的信息，而 `git stash -m "xxx"` 将会报错。常见的参数有：

- `-p`: `--patch` 的简写。 `git stash` 命令默认是存储所有的修改内容，而使用参数 `-p` 会出现交互界面，使得可以选择存储的文件。
- `-u`: `--include-untracked` 的简写。`git stash` 默认不会包括 untracked files。而通过此参数可以包括进去。
- `-a`: `--all` 的简写。作用同样会包括 untracked files。还有一种解决办法是先执行 `git add` 命令，后再执行 `git stash`，同样能把新增的文件进行存储。
- `-q`: `--quiet` 的简写。表示执行命令后，不在控制台打印相关日志信息。
- `-m`: `--message` 的简写。作用是手动设置存储消息信息。
- `-k`: `--keep-index` 的简写。那些已经添加到暂存区的文件，默认执行 `git stash` 命令后，会被清空，而通过该参数，那些暂存区的文件就不会被更改了。
