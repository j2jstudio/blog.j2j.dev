---
title: "基于Gitlab MR做代码审核 -  1基础配置"
date: 2020-04-15T16:52:08+08:00
draft: false
tags:
  - Code Review
  - 代码评审
  - 项目规范
---

## 分支的说明

gitflow工作流，规范但是过于复杂。

![https://docs.gitlab.com/ee/topics/img/gitlab_flow_gitdashflow.png](https://docs.gitlab.com/ee/topics/img/gitlab_flow_gitdashflow.png)


我们采用简易的gitflow工作流 

* master - 用于准备发布部署
* develop - 用于当前的主线开发
* feature - 用于特定功能的开发
* tag标签 - 用于版本发布 v0.9.10


<!--more-->

## 分支的权限管理


默认情况下，受保护的分支有如下特性:

* 未创建时，仅允许具有项目管理权限的用户(Maintainer)创建.
* 默认阻止向其 push 提交，除非提交的用户具有允许push的权限  .
* It prevents anyone from force pushing to the branch.
* It prevents anyone from deleting the branch.

### 项目Maintainer设置分支保护

1. Navigate to your project’s Settings ➔ Repository
2. Scroll to find the Protected branches section.
3. From the Branch dropdown menu, select the branch you want to protect and click Protect. In the screenshot below, we chose the develop branch.

设置 develop 分支受保护，仅允许Maintainer 自动合并、push。
![https://docs.gitlab.com/ee/user/project/img/protected_branches_page_v12_3.png](https://docs.gitlab.com/ee/user/project/img/protected_branches_page_v12_3.png)

4. 操作完成后 保护分支将在下方的列表中显示.

![https://docs.gitlab.com/ee/user/project/img/protected_branches_page_v12_3.png](https://docs.gitlab.com/ee/user/project/img/protected_branches_page_v12_3.png)

### Allowed to merge and Allowed to push 设置


Using the “Allowed to push” and “Allowed to merge” settings, you can control the actions that different roles can perform with the protected branch. For example, you could set “Allowed to push” to “No one”, and “Allowed to merge” to “Developers + Maintainers”, to require everyone to submit a merge request for changes going into the protected branch. This is compatible with workflows like the GitLab workflow.

设置任何人都需要提交合并请求到保护分支。

* “Allowed to push” 改为 “No one”禁止任何人push
* “Allowed to merge” to “Developers + Maintainers”，允许开发者+管理者合并

![https://docs.gitlab.com/ee/user/project/img/protected_branches_devs_can_push_v12_3.png](https://docs.gitlab.com/ee/user/project/img/protected_branches_devs_can_push_v12_3.png)


 




## 评审&合并流程 Merge/pull requests with GitLab flow

![https://docs.gitlab.com/ee/topics/img/gitlab_flow_mr_inline_comments.png](https://docs.gitlab.com/ee/topics/img/gitlab_flow_mr_inline_comments.png)


    Merge or pull requests are created in a Git management application. They ask an assigned person to merge two branches. Tools such as GitHub and Bitbucket choose the name “pull request” since the first manual action is to pull the feature branch. Tools such as GitLab and others choose the name “merge request” since the final action is to merge the feature branch. In this article, we’ll refer to them as merge requests.


## 参考

* [https://docs.gitlab.com/ee/topics/gitlab_flow.html](https://docs.gitlab.com/ee/topics/gitlab_flow.html)
* [https://docs.gitlab.com/ee/user/project/protected_branches.html](https://docs.gitlab.com/ee/user/project/protected_branches.html)
* [https://docs.gitlab.com/ee/user/permissions.html](https://docs.gitlab.com/ee/user/permissions.html)