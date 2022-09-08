## Git上传大文件

突破GitHub的限制，使用 [git-lfs(Git Large File Storage)](https://git-lfs.github.com/) 支持单个文件超过100M

 <img src=".asserts/image-20220321115832783.png" alt="image-20220321115832783" style="zoom:67%;" />

LFS 并不能像”变魔术一样”处理所有的大型数据：它需要记录并保存每一个变化。然而，这就把负担转移给了远程服务器 - 允许本地仓库保持相对的精简。

为了实现这个可能，LFS 耍了一个小把戏：它在本地仓库中并不保留所有的文件版本，而是仅根据需要提供检出版本中必需的文件。

但这引发了一个有意思的问题：如果这些庞大的文件本身没有出现在你的本地仓库中….改用什么来代替呢? LFS 保存轻量级指针中有真实的文件数据。当你用一个这样的指针去迁出一个修订版时，LFS 会很轻易地找到源文件（不在他上面可能就在服务器上，特殊缓存）然后你下载就行了。

因此，你最终只会得到你真正想要的文件 - 而不是一些你可能永远都不需要冗余数据。

```shell
# 1、安装git-lfs
brew install git-lfs

# 2、没有特别说明的情况下，LFS 不会处理大文件问题，因此，我们必须明确告诉 LFS 该处理哪些文件。将 FrameworkFold/XXXFramework/xxx的文件设置成大文件标示。
git lfs track "FrameworkFold/XXXFramework/xxx"

# 3、常规的push操作
git add .
git commit -m "add large file"
git push
```



## 冷门命令

*  git reset HEAD <fileName>   // 撤销add

* **git reset --soft HEAD^**	// 撤销commit

* git push --set-upstream origin 分支名 // 不提交代码，推送本地分支到远程仓库

* 从分支1拉出分支2，改动后想直接Push到分支2：

  git push origin branch2:branch1

* 批量删除分支： 

  比如删除名字带“cherry-pick”的所有分支: git branch | grep 'cherry-pick' | xargs git branch -D

  命令解析：

  * | 管道命令，用于将一串命令串联起来。前面命令的输出可以作为后面命令的输入。
  * git branch 用于列出本地所有分支
  * grep 搜索过滤命令。使用正则表达式搜索文本，并把匹配的行打印出来。
  * Xargs 参数传递命令。用于将标准输入作为命令的参数传给下一个命令。

* Git合并多个commit 参考https://segmentfault.com/a/1190000007748862

  * Git log 查看历史

  * git rebase，想要合并 1-3 条，有两个方法: 

    * git rebase -i HEAD~3		// 从HEAD版本开始往过去数3个版本
    * git rebase -i 3a4226b      // 指名要合并的版本之前的版本号, 请注意 `3a4226b` 这个版本是不参与合并的，可以把它当做一个坐标

  * 执行了 `rebase` 命令之后，会弹出一个窗口，头几行如下：

    ```shell
    pick 3ca6ec3   '注释**********'
    pick 1b40566   '注释*********'
    pick 53f244a   '注释**********'
    ```

  * 将 `pick` 改为 `squash` 或者 `s`，之后保存并关闭文本编辑窗口即可。改完之后文本内容如下：

    ```shell
    pick 3ca6ec3   '注释**********'
    s 1b40566   '注释*********'
    s 53f244a   '注释**********'
    ```

  * 然后保存退出，Git 会压缩提交历史，如果有冲突，需要修改，修改的时候要注意，保留最新的历史，不然我们的修改就丢弃了。修改以后要记得敲下面的命令：

    ```shell
    git add .  
    git rebase --continue  
    ```

  * 如果没有冲突，或者冲突已经解决，则会出现如下的编辑窗口：

    ```shell
    # This is a combination of 4 commits.  
    #The first commit’s message is:  
    注释......
    # The 2nd commit’s message is:  
    注释......
    # The 3rd commit’s message is:  
    注释......
    # Please enter the commit message for your changes. Lines starting # with ‘#’ will be ignored, and an empty message aborts the commit.
    ```

    





