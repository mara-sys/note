### git-stash
[原文链接](https://zhuanlan.zhihu.com/p/344100614)
#### 1. stash 当前修改
&emsp;&emsp;`git stash`会把所有未提交的修改（包括暂存的和非暂存的）都保存起来，用于后续恢复当前工作目录。
&emsp;&emsp;比如下面的中间状态，通过`git stash`命令推送一个新的储藏，当前的工作目录就干净了。
```shell
$ git status
On branch master
Changes to be committed:

new file:   style.css

Changes not staged for commit:

modified:   index.html

$ git stash
Saved working directory and index state WIP on master: 5002d47 our new homepage
HEAD is now at 5002d47 our new homepage

$ git status
On branch master
nothing to commit, working tree clean
```
&emsp;&emsp;需要说明一点，`stash`是本地的，不会通过`git push`命令上传到`git server`上。
实际应用中推荐给每个`stash`加一个`message`，用于记录版本，使用`git stash save`取代`git stash`命令。示例如下：
```shell
$ git stash save "test-cmd-stash"
Saved working directory and index state On autoswitch: test-cmd-stash
HEAD 现在位于 296e8d4 remove unnecessary postion reset in onResume function
$ git stash list
stash@{0}: On autoswitch: test-cmd-stash
```
#### 2. 重新应用缓存的 stash
&emsp;&emsp;可以通过`git stash pop`命令恢复之前缓存的工作目录，输出如下：
```shell
$ git status
On branch master
nothing to commit, working tree clean
$ git stash pop
On branch master
Changes to be committed:

    new file:   style.css

Changes not staged for commit:

    modified:   index.html

Dropped refs/stash@{0} (32b3aa1d185dfe6d57b3c3cc3b32cbf3e380cc6a)
```
&emsp;&emsp;这个指令将缓存堆栈中的第一个 stash 删除，并将对应修改应用到当前的工作目录下。你也可以使用`git stash apply`命令，将缓存堆栈中的stash多次应用到工作目录中，但并不删除 stash 拷贝。命令输出如下：
```shell
$ git stash apply
On branch master
Changes to be committed:

    new file:   style.css

Changes not staged for commit:

    modified:   index.html
```

#### 3. 查看现有 stash
&emsp;&emsp;可以使用`git stash list`命令，一个典型的输出如下：
```shell
$ git stash list
stash@{0}: WIP on master: 049d078 added the index file
stash@{1}: WIP on master: c264051 Revert "added file_size"
stash@{2}: WIP on master: 21d80a5 added number to log
```
&emsp;&emsp;在使用`git stash apply`命令时可以通过名字指定使用哪个`stash`，默认使用最近的`stash`（即`stash@{0}`）。

#### 4. 移除 stash
&emsp;&emsp;可以使用`git stash drop`命令，后面可以跟着`stash`名字。下面是一个示例：
```shell
$ git stash list
stash@{0}: WIP on master: 049d078 added the index file
stash@{1}: WIP on master: c264051 Revert "added file_size"
stash@{2}: WIP on master: 21d80a5 added number to log
$ git stash drop stash@{0}
Dropped stash@{0} (364e91f3f268f0900bc3ee613f9f733e82aaed43)
```
&emsp;&emsp;或者使用`git stash clear`命令，删除所有缓存的`stash`。

#### 5. 查看指定 stash 的 diff
&emsp;&emsp;可以使用`git stash show`命令，后面可以跟着`stash`名字。示例如下：
```shell
$ git stash show
 index.html | 1 +
 style.css | 3 +++
 2 files changed, 4 insertions(+)
```
&emsp;&emsp;在该命令后面添加`-p`或`--patch`可以查看特定stash的全部diff，如下：
```shell
$ git stash show -p
diff --git a/style.css b/style.css
new file mode 100644
index 0000000..d92368b
--- /dev/null
+++ b/style.css
@@ -0,0 +1,3 @@
+* {
+  text-decoration: blink;
+}
diff --git a/index.html b/index.html
index 9daeafb..ebdcbd2 100644
--- a/index.html
+++ b/index.html
@@ -1 +1,2 @@
+<link rel="stylesheet" href="style.css"/>
```

#### 6. 从 stash 创建分支
&emsp;&emsp;如果你储藏了一些工作，暂时不去理会，然后继续在你储藏工作的分支上工作，你在重新应用工作时可能会碰到一些问题。如果尝试应用的变更是针对一个你那之后修改过的文件，你会碰到一个归并冲突并且必须去化解它。如果你想用更方便的方法来重新检验你储藏的变更，你可以运行 git stash branch，这会创建一个新的分支，检出你储藏工作时的所处的提交，重新应用你的工作，如果成功，将会丢弃储藏。
```shell
$ git stash branch testchanges
Switched to a new branch "testchanges"
# On branch testchanges
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#      modified:   index.html
#
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#
#      modified:   lib/simplegit.rb
#
Dropped refs/stash@{0} (f0dfc4d5dc332d1cee34a634182e168c4efc3359)
```
&emsp;&emsp;这是一个很棒的捷径来恢复储藏的工作然后在新的分支上继续当时的工作。

#### 7. 暂存未跟踪或忽略的文件
&emsp;&emsp;默认情况下，git stash会缓存下列文件：
* 添加到暂存区的修改（staged changes）
* Git 跟踪的但并未添加到暂存区的修改（unstaged changes）

&emsp;&emsp;但不会缓存以下文件：
* 在工作目录中新的文件（untracked files）
* 被忽略的文件（ignored files）

`git stash`命令提供了参数用于缓存上面两种类型的文件。使用`-u`或者`--include-untracked`可以`stash untracked`文件。使用`-a`或者`--all`命令可以`stash`当前目录下的所有修改。












