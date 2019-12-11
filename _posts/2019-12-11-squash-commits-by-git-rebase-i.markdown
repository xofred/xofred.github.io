---
title:  "用 git rebase -i 合并多条提交记录"
tags: git

---

*git rebase -i master*

正如[这篇文章](https://blog.carbonfive.com/2017/08/28/always-squash-and-rebase-your-git-commits/)说的，多条提交合成一条后，合并到主分支后的历史看起来会更简洁、清晰。万一有问题需要排查或回滚，也方便许多。这也是[Rails 建议的提交流程](https://edgeguides.rubyonrails.org/contributing_to_ruby_on_rails.html)

用到的几个命令

`pick` 一般是作为第一条记录

`squash` 将这次提交合并到之前的提交

`reword` 想修改提交内容，但保留提交

例如这条工作日志，其实是由多条提交记录合并而成，并记录了每次提交的内容

```bash
commit ee5d2cb0984cbffe3883e819890b083c815ebe86 (HEAD -> feat/sku_toggle_is_value_pack, origin/feat/sku_toggle_is_value_pack)
Author: Justin Chen <justin.c@shopperplus.com>
Date:   Mon Dec 9 12:14:33 2019 +0800

    feat(sku): Value Pack 切换

    fix(sku_toggle_is_value_pack): 可能多个 master product

      [修复]可能多个 master product。只留切换后的 sku 作为 master product

      [功能]首次切换时，image 也初始化上传

    fix(sku_toggle_is_value_pack): 隐藏流程有误

      [修复]隐藏流程有误

      [功能]首次切换新增变种关系(变种默认取原来的)

    fix(sku_toggle_is_value_pack): 变种记录重复

      由于 VariantValidator 限制了一条变种记录不能对应多个 sku

      需要新增变种及关系(而且名称不能重复，暂定做法是名称后面加上【-1】)

    fix(sku_toggle_is_value_pack): sku 名称太长

      由于 sku_name、sku_name_fr 长度不能超过 100，100 之后的字符【自动舍弃】
```

但在主分支日志看来，关于这项工作的改动都在一条提交记录，历史比较清晰

```bash
*   6e394cb5f (HEAD -> master, origin/master, origin/HEAD) Merge branch 'feat/sku_toggle_is_value_pack' into 'master'
|\
| * ee5d2cb09 (origin/feat/sku_toggle_is_value_pack, feat/sku_toggle_is_value_pack) feat(sku): Value Pack 切换
*
```
