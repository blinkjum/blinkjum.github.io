



最常用的repo命令：
repo abandon xxx 删除xxx分支
**repo branch 查看当前分支**
repo checkout 切换分支
repo cherry-pick
**repo diff 显示提交树和工作树之间的更改**
repo diffmanifests 清单差异
repo download 下载并切换到
repo grep 打印匹配的行模型
repo info 获取有关的分支清单、当前分支或未合并的分支信息
**repo init 在当前目录中初始化repo**
**repo list 列出项目及其相关联的目录**
repo overview 显示未合并项目分支的概述
repo prune 删除已经合并的主题
repo rebase 将本地分支重新建立在上游分支上
repo smartsync 将工作树更新为最新的已知良好版本
repo stage 提交的阶段文件
repo start --all xxx 开始一个新的开发分支
**repo status 显示工作树的状态**
**repo sync 将工作树更新为最新版本**
repo upload 上传更改以供代码审查

repo用法的基本形式为:

```
repo <COMMAND> <OPTIONS>
```

可选项在[]中表示,例如许多命令接收一个**项目列表**作为参数,你可以通过一组**名字**或者p本地源目录的**path**来指定项目列表.

```
repo sync [<PROJECT0> <PROJECT1> <PROJECTN>]
repo sync [</PATH/TO/PROJECT0> ... </PATH/TO/PROJECTN>]
```

## help

使用`help`命令可以获得最新的帮助文档,由各子命令来组织.

你也可以查看子命令的帮助

```
repo help <COMMAND>
```

## init

```
$ repo init -u <URL> [<OPTIONS>]
```

在当前目录安装repo, 将创建一个`.repo`目录,内含源代码的git仓库和标准android manifest 文件, 同时还有一个`manifest.xml`文件,指向`.repo/manifests/`目录中的定manifest.
参数:

`-u` URL,指定manifest位置

```
-m` 选择仓库中的manifest文件,缺省为`default.xml
```

`-b` 指定revision, 如一个特定的manifest分支.

> repo的其它命令需要在.repo的父目录或者某一子目录中执行

## sync

```
repo sync [<PROJECT_LIST>]
```

下载新更改,并更新本地环境中的工作文件,如果不带参数,将同步所有的项目的所有文件.

当运行repo sync时,会执行以下操作:

1. 如果之前未进行过同步,那么等同于git clone, 所有远程的分支都被复制到本地.

2. 如果之前同步过,那么等同于:

   ```
   git remote update 
   git rebase origin/<BRANCH>
   ```

> 其中`<branch>`指当前本地目录所有的分支, 如果本地分支没有track远程分支,则不会同步.

1. 如果rebase发生冲突, 需要使用git命令来Fix冲突,如`git rebase --continue`.

sync成功之后, 指定项目中的代码更新到最新.

选项:

`-d` 将指定的项目切换回指定的manifest revision, 当项目处于topic分支,但需要manifest revision时有用

`-s` sync到一个已知的良好的build, 由当前manifest中的`manifest-server`元素指定.

`-f` 当某个project同步失败时继续sync

## upload

```
repo upload [<PROJECT_LIST>]
```

repo会比较指定项目的本地和远程最新更新,提示你**选择**尚未提交的项目

选择完成以后,repo会将项目的所有commit上通过**HTTPS**传到**gerrit**,你需要设置**认证**信息.访问此处[Password Generator](https://android-review.googlesource.com/new-password)可以生成一对新的username/password对.

当gerrit收到对象数据时, 会将每个commit转换为修改,这样review人员可以对每一处评论. 如果想合并commit,可以在upload前执行`git rebase -i`操作.

如果upload时不指定项目列表,它会搜寻所有项目更改.

如果upload之后要更改编辑,可以使用`git rebase -i`和`git commit --amend`来更新本地commit,之后:

- 确认当前branch是要提交的branch
- 对序列中每个要提交的commit,在括号中输入gerrit修改id

```
# Replacing from branch foo 
[ 3021 ] 35f2596c Refactor part of GetUploadableBranches to lookup one specific...
[ 2829 ] ec18b4ba Update proto client to support patch set replacments 
# Insert change numbers in the brackets to add a new patch set.
# To create a new change record, leave the brackets empty.
```

上传完成后,修改将会有附加的Patch集.

## diff

基于git diff, 显示commit和当前工作树的修改.

```
repo diff [<PROJECT_LIST>]
```

## download

```
repo download <TARGET> <CHANGE>
```

下载指定的修改并应用到工作目录中.

如download 23823号修改到本地目录`platform/framworks/base/`

```
$ repo download platform/build 23823
```

repo sync会移除通过download检索过的修改, 你可也以切换远程分支`git checkout m/master`

> 从Gerrit看到修改到用户可以下载,是有一小段延迟的,因为冗余备份的缘故.

## forall

```
repo forall [<PROJECT_LIST>] -c <COMMAND>
```

在指定的项目中逐个执行`<command>`, 可以使用以下环境变量

`REPO_PROJECT` 设置唯一的项目名
`REPO_PATH` 相对client root的相对路径
`REPO_REMOTE` manifest中的远程系统名
`REPO_LREV` manifest中的revision name翻译到本地tracking分支，当需要传递manifest的revision到本地执行的git命令的使用。
`REPO_RREV` 等于manifest中的revision name。

选项：

`-c` 需要执行的命令与参数，由/bin/sh解释
`-p` 在显示执行结果前打印项目headers, 通过bind管道到命令的Stdin , stdout和stderr来实现
`-v` 显示stderr信息

## prune

```
repo prune [<PROJECT_LIST>]
```

修剪/删除已经合并的topic

## start

```
repo start <BRANCH_NAME> [<PROJECT_LIST>]
```

从manifest中指定的revision**创建**一个新**分支**

`<BRANCH_NAME>` 应该是对该分支将要做的修改的简短描述，如果不确定，应该填入分支名字。

`<PROJECT_LIST>` 指定哪些项目将参与此topic分支

> `.` 是**当前工作目录**所有项目的简写

## status

```
repo status [<PROJECT_LIST>]
```

将当前工作树与列表中的每个项目的HEAD所在进行比较，并以行概要的形式打印每个文件的不同。

如果只打印当前项目的diff信息，直接运行`repo status`