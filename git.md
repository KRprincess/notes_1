## git
### 1.本地子分支与本地和远程master的同步
<br> (1) 本地master:  git pull
<br> (2) 本地子分支：  git rebase master
<br> (3) 本地master:  git merge 子分支名
<br> (4) 本地master:  git push

### 2. git切换分支出现的问题
举例:
当" git checkout master"<br>
出现"Deletion of directory 'src/tracking/test' failed. Should I try again?"<br>
出现以上问题一时该目录或文件处于busy状态，最好不要中断（abort）或ctral+c，不然子分支的修改也会丢失（但也不要紧张可以恢复版本）。<br>
一般关闭对文件和目录的操作就能解决。<br>
也有解释删除index.lock文件解决的，没试过，不知道有什么风险吗。


### 3. `.gitignore`的实际作用
对于**未入库/untracked**的文件进行忽略屏蔽，已经`commit`的文件则不起作用！！！

### 4. 不小心`add` 并且`commit`了一个大文件，怎么办？
导致的问题：
 - 增加clone,fetch,pull,push的负担，并且不利于版本控制
 - 无法push,因为一般git限制提交的文件不超过100M，虽然可以修改限制，但会一直成为负担问题

有效的解决方案：<br>
- 版本回退至add大文件的commit前
- 回退后最好将大文件（可使用通配符屏蔽一类）放到`.gitignore`里面屏蔽掉，以防后面无意又track了
- 重新开始，不要提交大文件即可

大致步骤：
-  首先将当前项目备份下，不然回退后修改就没有了
-  `git log`或`git reflog`查看要回退的位置，copy对应的`commitID`或者`HEAD@{XX}`
-  `git reset --hard commitID(或HEAD@{xx})`进行回退
-   创建或修改已有的`.gitignore`文件，将忽略的文件添加进去，例如`*.mp4`忽略mp4的video
-   将第一步备份的修改更新下项目
-   重新进行`add`和`commit`步骤

无效或不推荐的解决方案：<br>
- 无效：`.gitignore`<br>
  因为`.gitignore`只能屏蔽掉**un-tracked**的文件
- 不推荐：`git filter-branch`<br>
  有些人介绍可以使用`git filter-branch`修改所有有关指定文件的`commit`,该方法确实能够过略掉对指定文件的添加，但是，要使操作生效，必须对版本的reflog做过期操作，强制让之前的记录全部国企。<br>
  参考：<br>
       (1) https://blog.csdn.net/xmh594603296/article/details/83987271<br>
       (2)https://blog.csdn.net/xmh594603296/article/details/83987271<br>

### 5. git stash, git restore, git tag的重要作用


### 6.git rebase压缩commit


### 7. 撤销修改和误删除
(1)撤销部分文件的修改
git checkout -- path/to/file1 path/to/file2

或者(git 2.23+)

git restore --worktree path/to/file1 path/to/file2

(2)撤销工作区下所有文件的修改（不包括新增文件）
git checkout -- .

或者(git 2.23+)

git restore --worktree .

(3)将一个或多个文件回滚到指定版本
git checkout dce33f4 -- path/to/file1 path/to/file2

(4)将一个或多个文件回滚到指定版本的前2个版本
git checkout dce33f4 ~2 -- path/to/file1 path/to/file2

(5)将一个或多个文件回滚到指定分支版本
git checkout develop -- path/to/file1 path/to/file2

(6)丢弃工作区中所有不受版本控制的文件或目录
git clean -fdx

作者：炉石不传说
链接：https://www.jianshu.com/p/38921d19ba0a
