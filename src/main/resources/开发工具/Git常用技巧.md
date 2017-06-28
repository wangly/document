##一、回滚push，但不丢失已有修改
假设要回滚当前分支，分支名为：branch_name
git branch branch_bk  创建一个备份分支
git log  查看commit记录，找到要回滚到的commit_id（正常为最近的第二个commit）
git reset --hard commit_id  彻底回滚本地代码（会删掉以后的修改，所以需要第一步的备份）
git push origin :branch_name  将远程对应分支删除
git push origin  把当前回滚后（干净）的内容push到远程
git merge branch_bk  将备份合并到当前分支
git reset --mixed commit_id  合并过来的分支是已提交的，因此，需要reset commit和add。
【reset参数：--soft仅回滚HEAD指向（对于回滚到上一次提交而言，相当于撤销commit），--mixed回滚HEAD和index（相当于撤销commit和add），–hard回滚HEAD、index和working copy，会删除所以已有修改】
      至此，查看version control，你会发现，所有的修改都保留在你的working copy。此时，根据你需要，重新add、commit、push即可。
##二、merge参数之squash
git  merge --squash another_branch 
      将another_branch分支的内容合并到当前分支，本地文件内容与不使用该选项的合并结果相同，但是不提交、不移动HEAD，因此需要一条额外的commit命令。其效果相当于将another分支上的多个commit合并成一个，放在当前分支上，原来的commit历史则没有拿过来。
##三、commit参数之amend
 git commit --amend
      修改之前提交的commit message，一并提交。
##四、git缓存
git stash
      作用：缓存本地工作区的内容，以临时切换到其他分支进行开发，待开发完成，回到该分支，只需git stash pop，即可返回缓存在堆栈的内容。
      注意：使用stash需谨慎，千万不要以为stash是存储在当前分支下的缓存，而是所有分支公用的堆栈。如果对多个分支同时stash，虽然不是不可以，但当你再次pop的时候，容易造成混乱。
git stash save 'remind info'
如需多次stash，可以使用: 「save」来保存每次stash的提示信息；（使用该方法，可备注stash的分支，以避免多分支stash混乱问题）
git stash list
查看stash的列表及编号

git stash apply stash@{1}
根据stash号，取回缓存数据。
##五、删除远程某个文件或文件夹
某些文件本需要加入跟踪，如target/、idea/，不幸首次提交不小心一同提交了。此时，需要git 删除一个远程文件夹，但本地保留却要保留。
git rm -r -n --cached  */directory/\*  //列出需要删除的文件to check，并非真正执行
git rm -r --cached  */directory/\* 
git commit -m "commit info"
git push origin
强制更新：git push origin --force
##六、重命名远程分支
本质是删除远程分支，重命名本地分支，提交到远程
git push origin  :oldBranchName
git branch -m oldBranchName newBranchName
git push newBranchName
git branch -D(or -d) branch 的区别：
-d：若branch有未合并到当前分支的内容，会提示the branch  XXX is not fully merged。
-D：强制删除branch分支。
##七、git remote prune origin
remote上的一个分支被其他人删除后，需要更新本地的分支列表。
##八、修改远程仓库地址
git remote set-url origin ssh://git@git.sankuai.com/hotel/hotel-pms.git
场景：st分支合了commit1、commit2、commit3、commit4代码，发现commit2需要下线。
解决步骤：
git checkout st //切st分支
git reset --mixed commit2 //将commit3、4恢复到本地工作空间
git stash save "commit3&commit4"  //将commit3、4缓存
git reset --hard commit1 //将commit2代码彻底回滚
git stash pop //将commit3、4pop到工作空间
//解决冲突，commit
git remote set-url origin ssh://git@git.sankuai.com/hotel/hotel-pms.git //将远程仓库地址改为主仓库
git push origin :st //将主仓库的st分支删除
git push origin
//到此为止，主仓库的st分支commit2已经被删除
git remote set-url origin [个人仓库地址] //记得还原远程仓库地址
设置默认远程：git push --set-upstream origin <branch> 指定上游，git pull/push 默认对上游操作
##九、指定多个远程
添加：git remote add <name> <new-url>
删除：git remote remove <name>
查看：git remote -v