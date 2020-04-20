---
title: "基于Gitlab MR做代码审核 -  2演示"
date: 2020-04-15T17:52:08+08:00
draft: false
tags:
  - Code Review
  - 代码评审
  - 项目规范
---

# Gitlab Merge Requests 流程说明二 - 演示

## 迁出代码

```
#仅克隆远程master分支 
git clone https://gitlab.company.com/demo/prdemo.git

#查看所有分支

git branch -a

#显示如下 

* master  (仅迁出master分支)
  remotes/origin/HEAD -> origin/master
  remotes/origin/develop
  remotes/origin/master
  
# 迁出develop分支

git checkout -b develop origin/develop
git branch -a

* develop (develop已经迁出，且当前处于develop分支中)
  master (master分支已经迁出)
  remotes/origin/HEAD -> origin/master
  remotes/origin/develop
  remotes/origin/master
  
```

<!--more-->

## 在受保护的develop分支中操作

旧的工作流，直接在  develop上开发，并提交。在不受保护的分支中是没有问题，如果分支开启了保护，并禁止push，则push代码会被拒绝。



```
vi test1.md

...

git add .
git commit -a -m 'add  test1.md'

#输出
[develop f8270d4] add  test1.md
 1 file changed, 3 insertions(+)
 create mode 100644 test1.md
 

git push

#输出
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 368 bytes | 368.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)

#收到保护，禁止push
remote: GitLab: You are not allowed to push code to protected branches on this project.
To https://gitlab.company.com/demo/prdemo.git
 ! [remote rejected] develop -> develop (pre-receive hook declined)
error: failed to push some refs to 'https://gitlab.company.com/demo/prdemo.git'


```

develop的配置如下：

![https://wx3.sinaimg.cn/mw690/90594e01gy1gdulwk6wwij20os0e80u1.jpg](https://wx3.sinaimg.cn/mw690/90594e01gy1gdulwk6wwij20os0e80u1.jpg)


## 新的工作流

### 创建功能开发分支 并提交更改

```

git checkout -b  feature-v0.0.3-demo 

git checkout -b feature-v0.0.3-demo 
Switched to a new branch 'feature-v0.0.3-demo'

git branch
  develop
* feature-v0.0.3-demo (当前所在分支)
  master


```

```
vi 1.txt

...


git add .
git commit -a -m 'add 1.txt'
[feature-v0.0.3-demo 520620d] add 1.txt
 1 file changed, 1 insertion(+)
 create mode 100644 1.txt
 
git push

#输出错误，当前分支未与远程进行绑定
fatal: The current branch feature-v0.0.3-demo has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin feature-v0.0.3-demo
    
#push并绑定上游origin
git push --set-upstream origin feature-v0.0.3-demo

#push成功
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 4 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 720 bytes | 720.00 KiB/s, done.
Total 6 (delta 0), reused 0 (delta 0)
remote: 
#提示创建merge_requests的访问地址
remote: To create a merge request for feature-v0.0.3-demo, visit:
remote:   https://gitlab.company.com/demo/prdemo/merge_requests/new?merge_request%5Bsource_branch%5D=feature-v0.0.3-demo
remote: 
To https://gitlab.company.com/demo/prdemo.git
 * [new branch]      feature-v0.0.3-demo -> feature-v0.0.3-demo
Branch 'feature-v0.0.3-demo' set up to track remote branch 'feature-v0.0.3-demo' from 'origin'.

```

gitlab上查看新建的分支已经存在

![https://wx3.sinaimg.cn/mw690/90594e01gy1gdumc8unt3j20r80edta3.jpg](https://wx3.sinaimg.cn/mw690/90594e01gy1gdumc8unt3j20r80edta3.jpg)


### 创建Merge Request

除了从提示中直接进入创建merge request页面外，新push分支后，进入gitlab会看到 Create Merge Request的按钮：

![https://wx1.sinaimg.cn/mw1024/90594e01gy1gdumqfbsa4j20qw03njro.jpg](https://wx1.sinaimg.cn/mw1024/90594e01gy1gdumqfbsa4j20qw03njro.jpg)

进入Create 页面：
![https://wx3.sinaimg.cn/mw1024/90594e01gy1gdumqfbp0zj20wj05h0t5.jpg](https://wx3.sinaimg.cn/mw1024/90594e01gy1gdumqfbp0zj20wj05h0t5.jpg)

需要确认要合并到的分支（gitlab默认要合并到master，我们需要更改到develop),点击"change branches":

![https://wx2.sinaimg.cn/mw1024/90594e01gy1gdumqfdnskj20wn093t9t.jpg](https://wx2.sinaimg.cn/mw1024/90594e01gy1gdumqfdnskj20wn093t9t.jpg)

填写Merge Request的信息：

![https://wx4.sinaimg.cn/mw1024/90594e01gy1gdumqfd3v2j20zw0n0di3.jpg](https://wx4.sinaimg.cn/mw1024/90594e01gy1gdumqfd3v2j20zw0n0di3.jpg)

选择默认合并选项：

1. 选中  Delete source branch when merge request is accepted. （合并后删除源分支)
2. 不选中 Squash commits when merge request is accepted. 

提交merge request请求：

![https://wx2.sinaimg.cn/mw1024/90594e01gy1gdumqfcpe8j20md0brdgo.jpg](https://wx2.sinaimg.cn/mw1024/90594e01gy1gdumqfcpe8j20md0brdgo.jpg)

会看到项目出现了一个 未处理的 Merge Requests .


###  评审&管理Merge Request 合并

一般merge requests列表如下：

![https://docs.gitlab.com/ee/user/project/merge_requests/img/project_merge_requests_list_view.png](https://docs.gitlab.com/ee/user/project/merge_requests/img/project_merge_requests_list_view.png)

点击进入Merge Request详情，可以查看该次MR的Commit信息，Changes信息。

![https://wx2.sinaimg.cn/mw1024/90594e01gy1gdunsqhn3pj20s70n8dij.jpg](https://wx2.sinaimg.cn/mw1024/90594e01gy1gdunsqhn3pj20s70n8dij.jpg)

并可以在代码变动中的任何位置提交评论。
![https://wx1.sinaimg.cn/mw1024/90594e01gy1gdunsqemcwj20td0emq57.jpg](https://wx1.sinaimg.cn/mw1024/90594e01gy1gdunsqemcwj20td0emq57.jpg)

发现问题并修复后可点击 Merge 进行合并：
![https://wx3.sinaimg.cn/mw1024/90594e01gy1gdunsqdlz6j20es06dq3k.jpg](https://wx3.sinaimg.cn/mw1024/90594e01gy1gdunsqdlz6j20es06dq3k.jpg)

进入Repository - Branches中查看，可以发现合并的记录，同时远程的源分支 'feature-v0.0.3-demo'已经被删除。

![https://wx2.sinaimg.cn/mw1024/90594e01gy1gdunsqdt0uj20z709d75i.jpg](https://wx2.sinaimg.cn/mw1024/90594e01gy1gdunsqdt0uj20z709d75i.jpg)

### 合并后的内容拉取到本地

```
git branch
  develop
* feature-v0.0.3-demo
  master

git checkout develop
Switched to branch 'develop'


git pull

Updating f8270d4..673a963
Fast-forward
 1.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 1.txt
 
 
git log 
 
commit 673a963d89affcc2760296a10ec80e2a33be0192 (HEAD -> develop, origin/develop)
Merge: 9d9fb48 520620d
Author: 蒋晶 <jiangjing@imohoo.com>
Date:   Wed Apr 15 18:59:42 2020 +0800

    Merge branch 'feature-v0.0.3-demo' into 'develop'
    
    Feature v0.0.3 demo - 标题位置
    
    See merge request demo/prdemo!1
 
```

Merge Reqeust流程完成。

### Merge Requst评审未通过的处理

默认采用不创建新的 ’New merge request'方式。 

1. A 发起Merege Request
2. B 查看MR, 进行Code Review, 发现问题并提交Comment
3. A 收到Comment提醒，并确认问题
4. A 在本地调整代码，并重新push
5. 此时可进行两种操作
	* 5.1 创建新的Merge Request
	* 5.2 不创建新的，所有的流程仍然会在上一MR中合并展示

6. B 继续评审无问题后可Merge代码到目标分支


A在一个分支中继续push代码后，出现创建新MR的按钮	
![https://wx3.sinaimg.cn/mw1024/90594e01gy1ge09uu6mv9j20xk0a13zh.jpg](https://wx3.sinaimg.cn/mw1024/90594e01gy1ge09uu6mv9j20xk0a13zh.jpg)

如果不新建MR，则所有的信息都会在一起展示

![https://wx1.sinaimg.cn/mw1024/90594e01gy1ge09uxpi87j20se0ql778.jpg](https://wx1.sinaimg.cn/mw1024/90594e01gy1ge09uxpi87j20se0ql778.jpg)

对于Code Review的中发现的问题解决后，可以‘Resolve thread'

![https://wx2.sinaimg.cn/mw1024/90594e01gy1ge09uxpe6nj20rd07xaap.jpg](https://wx2.sinaimg.cn/mw1024/90594e01gy1ge09uxpe6nj20rd07xaap.jpg)



### 错误处理

**1. merge request 落后于目的branch**

需要将目的分支pull到本地，并处理完冲突&合并，然后重新提交merge request

**2.冲突处理**

本地合并结解决冲突


