# 一个 master，多个 feature 分支，如何有序合并和提交

git-flow





工作流程

去自己的工作分支
`$ git checkout work`

工作
`....`

提交工作分支的修改
`$ git commit -a`

回到主分支
`$ git checkout master`

获取远程最新的修改，此时不会产生冲突
`$ git pull`

回到工作分支
`$ git checkout work`

用rebase合并主干的修改，如果有冲突在此时解决
`$ git rebase master`

回到主分支
`$ git checkout master`

合并工作分支的修改，此时不会产生冲突。
`$ git merge work`

提交到远程主干
`$ git push`

这样做的好处是，远程主干上的历史永远是线性的。每个人在本地分支解决冲突，不会在主干上产生冲突。



简而言之

master  -- rebase -->  feature

master  <-- merge --  feature

重中之重

不管哪个分支，push 上去前一定要先 pull



画一张更详细的流程图