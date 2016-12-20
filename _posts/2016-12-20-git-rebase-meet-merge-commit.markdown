---
layout: post
title: "当 git rebase 遇到 merge commit"
date: 2016-12-20 21:39:21
categories: git
---

今天想要合并一些commits。理所当然要使用 git rebase -i，然后有冲突（因为这个项目就我一人开发，所以冲突都直接采取更新的版本，解决的很快）。然而却出现了问题，rebase完成之后，有几个文件的内容却并不是最新的版本。仔细一看，发现在 rebase -i 的list中少了一个 commit，这个 commit 是一个 merge 操作产生的。难道 rebase 对 merge commit 有些特别？ 一搜还真是，具体原因可以参看[这里的置顶的回答](http://stackoverflow.com/questions/15915430/what-exactly-does-gits-rebase-preserve-merges-do-and-why)，回答的很详细，反而倒是 git 的文档说的过于简洁。一句话说，就是默认的 rebase 会忽略 merge commit，只有使用了 --preserve-merges 参数才会重演 merge commit。


接着加上这个参数之后，结果在 rebase 到了这个 merge commit 的时候仍然出错了

`error: Commit 5f43262298b846a219baa6fd7ec2a39363557112 is a merge but no -m option was given.
fatal: cherry-pick failed
`

这次是因为 merge commit 存在两个 parent。rebase 操作进行的时候是依次重演每个 commit（依次添加 diff 的内容），而到了 merge commit 时，它有两个 parent，那么就有两个 diff，rebase 就不知道该选择哪个 diff 来进行重演了。

最后我换了一个思路。使用 git cherry-pick 操作，新建一个分支，然后把目标分支的 commit 依次使用 cherry-pick 重演，当然到了那个 merge commit，cherry-pick 也会失败，需要使用 --mainline 参数，指明 cherry-pick 的 parent 才行，这个新的分支，就是目标分支精简 commit 后的样子。
