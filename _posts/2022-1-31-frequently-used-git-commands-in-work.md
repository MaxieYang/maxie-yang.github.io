---
layout: post
title: 工作中常用的Git命令
---
  
## 修改备注
*修改最新提交的commit备注*  
`git commit --amend`  
*pick 不变，r为变更备注*  
`git rebase -i <指定commit的父commit>`  

### 合并commit
*将改为s的commit合并到最前一个pick，也可调整p的commit的位置*  
`git rebase -i <指定commit的父commit>`  


## 比较差异

*比较暂存区（add后）和head文件的差异*  
`git diff --cached`  

*比较工作区和暂存区的文件差异*  
`git diff`  

*比较指定文件工作区和暂存区的文件差异*  
`git diff -- [<filename>...]`  

*查看指定文件在commit_id1和commit_id2的差异*  
`git diff <commit_id1> <commit_id2> -- [<filename>...]`  


## 变更操作
### 变更工作区内容用git checkout  变更暂存区内容用git reset  

*将暂存区的内容变回HEAD文件（用git diff --cached验证暂存区和HEAD的区别）*  
`git reset HEAD`  

*将工作区的指定文件变回暂存区内容（用git diff -- [<filename>] 验证工作区和暂存区的区别）*   
`git checkout -- [<filename>...]`  

*将暂存区的指定文件取消变更*  
`git reset HEAD -- [<filename>]`  
  
*将分支、暂存区、工作区的文件内容都指向到指定commit上*   
`git reset --hard <commit_id>`  

*将指定文件在暂存区中删除*  
`git rm filename`  

*将工作区内容暂时存放起来*  
`git stash`

*将存放到stash的内容取出并删除或只是取出*
`git stash pop/apply`  
