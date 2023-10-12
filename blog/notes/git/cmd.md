## Git 常用命令

1. 解决终端里 git status 不能显示中文：

```bash
git config --global core.quotepath false
```

2. 快速拉取最小体积的仓库代码（适用于只想 clone 最新版本使用或学习，无需下载  git 协作历史，因为不需要提交代码）：

```bash
git clone --depth=1 $git-url
```

3. git clone 仓库的一个子目录

```bash
git clone --filter=blob:none --sparse $git-url
git sparse-checkout set $your-folder

# 示例
git clone --filter=blob:none --sparse https://github.com/Yonatan-D/Monster.git
git sparse-checkout set GobangPC
```

4. 暂存当前正在进行的工作，拉取最新代码：

```bash
git stash
git pull
git stash pop
```

5. 撤销已提交文件的跟踪状态。遇到一个情况，提交代码后才想起来要添加 .gitignore，这时候再加入忽略是不行的。解决方法是先把本地缓存删除（改变成未track状态），然后再提交：

```bash
git rm -r --cached 要撤销的文件名
git commit -m 'delete 要撤销的文件名'
```

6. 删除 Git 历史记录中的文件（常见情况：node_modules、密码文件、构建好的二进制或镜像大文件）。假如误将大文件提交了，即使后面已经删除，但在 .git 目录中这个文件还是存在的：

```bash
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch 你要删除的文件名' --prune-empty --tag-name-filter cat -- --all
```

在重写提交的过程中，会有以下日志输出：

```bash
Rewrite 6cdbb293d453ced07e6a07e0aa6e580e6a5538f4 (266/266)
# Ref 'refs/heads/master' was rewritten
```

删除的过程会很漫长，实现原理是遍历一遍项目的 git 树，从每一个节点中去搜索你输入的文件名，找到有就删掉。执行完成后，本地 .git 里面还需要清理和回收空间，最快捷的办法是删掉本地项目，重新克隆。

参考自：https://www.hollischuang.com/archives/1708

7. git 查看文件权限

```bash
git ls-tree HEAD 文件名
```

8. git 修改文件权限

```bash
git update-index --chmod=+x 文件名
```

9. 将一个本地项目同时更新到 GitHub 和 Gitee 上

修改 .git/config 配置文件：

```properties
[core]     
    repositoryformatversion = 1    
    filemode = false     
    bare = false     
    logallrefupdates = true     
    symlinks = false     
    ignorecase = true    
[remote "origin"]     
    url = github项目地址    
    url = gitee项目地址     
    fetch = +refs/heads/*:refs/remotes/origin/*    
[branch "master"]     
    remote = origin     
    merge = refs/heads/master 
```

推送时只需要执行 git push，记得先 git pull。

10. git add 时忽略文件或文件夹

```bash
git add --all -- ':!src/*'
```

11. git pull 不用每次输入用户名密码

```bash
git config --global credential.helper store
```

或者，编辑 ~/.gitconfig 配置文件：

```properties
[user]
name = ‘git用户名’
email = ‘git邮箱’

[credential] 
helper = store
```

12. 撤消所有的本地更改，重置为分支的原始版本：

```bash
git fetch --all
git reset --hard origin/master
git pull
```

13. 解决 warning: LF will be replaced by CRLF

问题：

```
warning: LF will be replaced by CRLF
warning: LF will be replaced by CRLF in xxxxx
The file will have its original line endings in your working directory.
```

原因：Git 默认配置替换回车换行成统一的CRLF，我们只需要修改配置禁用该功能即可。

解决办法：

```bash
git config --global core.autocrlf false
```

14. 修改提交信息

```bash
git commit --amend -m "new message"
```

15. 添加文件到最后一次提交

```bash
git add <file_name>
git commit --amend HEAD~1
```

16. 撤消最近一次提交但保留更改

```bash
git reset --soft HEAD~1
```

可以撤消任意数量的提交，例如：`git reset HEAD~3`（返回 HEAD 之前的 3 个提交)

17. 撤消提交和更改（丢弃更改）

```bash
git reset --hard HEAD~1
```

返回特定的提交：`git reset --hard <commit_id>`

18. 撤消提交而不修改现有历史记录（创建新的提交来撤消提交）

```bash
git revert HEAD
```

如果尚未推送提交，并且不想引入糟糕的提交到远程分支，可以使用 git reset。

19. 撤消已经推送到远程分支的合并提交

```bash
git revert -m 1 <commit_id>
```

