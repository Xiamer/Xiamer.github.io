---
title: git 常用命令总结
date: 2020-11-19 17:43:24
categories: 
- git
tags:
- commit
---


## git cherry-pick (指定的提交（commit）应用于当前分支)

http://www.ruanyifeng.com/blog/2020/04/git-cherry-pick.html

```bash
git cherry-pick [hash] // 单个
git cherry-pick <HashA><HashB> // 多个
git cherry-pick A..B  // A到B 不包含A
git cherry-pick A^..B // A到B
// 代码冲突
--continue  --abort 丢弃 --quit 退出cherry pick，但不回答之前

// 某个commit 更新部分文件
git cherry-pick -n //(--no-commit) 只更新工作区和暂存区，不产生新的提交。
```

## git  rebase (合并commit 、 合并分支或别的分支commit到当前分支)

https://www.jianshu.com/p/4a8f4af4e803

1. 场景：合并多个commit 为 1个commit（push之前）
	a. git rebase -i  [startpoint]  [endpoint] 前开后闭
	b. git rebase -i HEAD~n   最近n条
2. 场景：将某一个分支的部分commit 合并到当前分支
	a. git rebase   [startpoint]   [endpoint]  --onto  [branchName] -> [hash] （分支未变，只是head变到了 hash位置，接下来需加target 分支重置到超前的hash）
	b. git checkout [targetBranch] 
	c. git reset --hard [hash]
  
## git reset (回退到某个commit)

 https://blog.csdn.net/yxlshk/article/details/79944535

1. 场景：<font>回退到某个commit</font>
	a. git reset --hard [hash | HEAD^]  
	b. git push -f // <font>需要加参数 -f 强推</font>， 远程hash后的commit将会消失
2. 常用参数 https://www.jianshu.com/p/c2ec5f06cf1a
  a. git reset --sort [hash] 回退到hash 版本，hash版本后修改放在暂存区
  b. git reset --mixed [hash] 回退到hash版本，版本之后的所有改变都放到工作区
  c. git reset --hard [hash] 回退到hash 版本，hash版本后内容清空！！

## git revert （撤销某个commit）
https://blog.csdn.net/yxlshk/article/details/79944535
场景：<font>撤销某个commit</font>，但是保留了后面的版本 （会生成一个新的commit）
a. git revert -n [hash]
b. git commit -m "revert xxx"

## git log

git log --pretty=oneline
