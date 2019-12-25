---
title:  "批量删除本地已合并的开发分支"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: git shell markdown

---

*git branch -d \`git branch | grep -v -E 'master|testing|release'\`*

## 正文

有时砖头太多，本地开了不少分支。所以想批量删除公用分支之外的所有分支

### 挑出不要的分支

```bash
git branch|grep -v -E 'master|testing|release'
```

>   feat/copy_sku_pictures<br>
>   fix-net-profits-error-too-much<br>
>   hotfix/new_value_pack_error_if_no_real_product

### 批量删除

```bash
git branch -d `git branch|grep -v -E 'master|testing|release'`
```

> Deleted branch feat/copy_sku_pictures (was 1fbcac31d).<br>
> Deleted branch fix-net-profits-error-too-much (was 1b4215fbe).<br>
> Deleted branch hotfix/new_value_pack_error_if_no_real_product (was 6596e776d).

## PS: 本文用到的 markdown 语法

### 输出控制字符

例如本文副标题部分的反引号 \`，以及这段话即将出现的 \\，并不想当成控制字符。在其前面加上反斜杠 \\ 即可

### 多行引用换行

默认 `>` 会输出【一整段】引用。可用 `<br>` 控制自己想要的排版，就像【正文】那样

### 多级 Table of Content

使用 `##`（h2）、`###`（h3）即可

## Reference

[Can you delete multiple branches in one command with Git?](https://stackoverflow.com/questions/3670355/can-you-delete-multiple-branches-in-one-command-with-git)

[Grep all string which do not starts with number(s)](https://unix.stackexchange.com/questions/186821/grep-all-string-which-do-not-starts-with-numbers)

[Escaping special characters in markdown](http://tech.saigonist.com/b/code/escaping-special-characters-markdown.html)

[Markdown 引用](https://www.jianshu.com/p/d87d7d2dcea9)
