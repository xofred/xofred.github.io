---
title:  "解决 git pull 本地引用更新失败"
toc: true
toc_label: "目录"
toc_icon: "cog"
tags: git

---

*`git remote prune origin`*

### 问题

```bash
git checkout master
```

> Switched to branch 'master'<br>
> Your branch is behind 'origin/master' by 35 commits, and can be fast-forwarded.<br>
>   (use "git pull" to update your local branch)<br>

```bash
git pull
```

> error: cannot lock ref 'refs/remotes/origin/fix/po_sent_email_histories_pdf_price_wrong': 'refs/remotes/origin/fix' exists; cannot create 'refs/remotes/origin/fix/po_sent_email_histories_pdf_price_wrong'<br>
> From code.shopperplus.com:rain/console<br>
>  ! [new branch]          fix/po_sent_email_histories_pdf_price_wrong -> origin/fix/po_sent_email_histories_pdf_price_wrong  (unable to update local ref)<br>

### 解决

```bash
git gc --prune=now
```

> Enumerating objects: 217374, done.<br>
> Counting objects: 100% (217374/217374), done.<br>
> Delta compression using up to 8 threads<br>
> Compressing objects: 100% (51634/51634), done.<br>
> Writing objects: 100% (217374/217374), done.<br>
> Total 217374 (delta 168535), reused 212702 (delta 164244)<br>
> Checking connectivity: 217374, done.<br>

```bash
git remote prune origin
```

> Pruning origin<br>
> URL: git@code.shopperplus.com:rain/console.git<br>
>
>  * [pruned] origin/170109<br>
>  * [pruned] origin/AR4<br>
>  * [pruned] origin/Add-link-to-Tracking-Number<br>
>
> ...

```bash
git pull
```

> From code.shopperplus.com:rain/console<br>
>
>  * [new branch]          fix/po_sent_email_histories_pdf_price_wrong -> origin/fix/po_sent_email_histories_pdf_price_wrong<br>
>    Updating 2778783a7..f30a80f48<br>
>    Fast-forward<br>
>     Gemfile                                                                |   1 -<br>
>     Gemfile.lock                                                           |  26 ------<br>
>     app/controllers/application_controller.rb                              |   1 -<br>
>
> ...

```bash
git status
```

> On branch master<br>
> Your branch is up to date with 'origin/master'.<br>
>
> nothing to commit, working tree clean<br>

### 原因
> error: cannot lock ref 'refs/remotes/origin/fix/po_sent_email_histories_pdf_price_wrong': 'refs/remotes/origin/fix' exists; cannot create 'refs/remotes/origin/fix/po_sent_email_histories_pdf_price_wrong'

`refs/remotes/origin/fix` 已经存在，导致 `refs/remotes/origin/fix/po_sent_email_histories_pdf_price_wrong` 无法创建。更好的做法，本地清掉 `refs/remotes/origin/fix` 即可。不过上面的命令全清也不错，公司的代码分支是有点乱😅

### Reference

[git pull fails “unable to resolve reference” “unable to update local ref”](https://stackoverflow.com/questions/2998832/git-pull-fails-unable-to-resolve-reference-unable-to-update-local-ref)
