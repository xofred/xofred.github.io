---
title:  "git 以源（目标）分支为准批量解决冲突"
tags: git
---

*tl; dr `--strategy-option`*

---

有时很明确知道以哪条分支为准，合并冲突时可以这样

`git merge --strategy-option theirs`

其中`theirs`采用源分支，`ours` 采用目标分支

除了 `merge` ，像 `rebase`、`pull` 这些有可能产生冲突的命令，都可以用 `--strategy-option` 批量解决冲突

### Reference: [Resolve Git merge conflicts in favor of their changes during a pull](https://stackoverflow.com/questions/10697463/resolve-git-merge-conflicts-in-favor-of-their-changes-during-a-pull)
