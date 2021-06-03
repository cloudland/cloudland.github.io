---
comment: false
aside:
  toc: true

title: Git 使用手册
date: 2021-06-03 11:00
tags: 版本工具
---

## Git 开发命令

### 仓储管理

* 本地初始仓库

  git init

* 检出仓库

  git clone [Git地址]

  ```shell
  git clone https://github.com/cloudland/cloudland.github.io.git
  ```

* 添加远程仓库

  git remote add [本地仓储名称] [Git地址]

  ```shell
  git remote add origin https://github.com/cloudland/cloudland.github.io.git
  ```

* 删除远程仓库

  git remote rm [本地仓储名称]

  ```shell
  git remote rm origin
  ```

* 拉取远程仓库

  git pull [remoteName] [localBranchName]

  ```shell
  git pull origin master
  ```

* 推送远程仓库

  git push [remoteName] [localBranchName]

  * -f: 强制推送, 覆盖远程文件(禁用)

  * -u: 关联提交之后, 可直接用`git push`省去[remoteName] [localBranchName]

  ```shell
  git push origin master
  ```

### 分支管理

* 查看分支

  * -r: 查看远程

  git branch -r

* 创建本地分支

  * -b: 创建本地分支并切换到新分支

  git branch [localBranchName]

  ```shell
  git branch -d develop
  ```

* 切换分支

  git checkout [localBranchName]

* 删除分支

  * -d: 只能删除已经参与了合并的分支，对于未有合并的分支是无法删除的。

  * -D: 想强制删除一个分支

   git branch -d [localBranchName]

* 合并分支

  > 将名称为[localBranchName]的分支与当前分支合并

  git merge [localBranchName] 

* 本地分支推到远程

  git push origin [localBranchName]


### 文件管理

* 添加文件 Git Index

  git add [fileName]

* 删除指定文件

  * --cached: 只从暂存区中删除

  * -f: 强行删除

  git rm [文件名/路径]

* 提交

  * -a: 提交全部变更

  git commit

  git commit -am "init" 

### 版本管理

* 创建标记

  git tag

    * -a: 版本号

    * -m: 标识说明

    * -d: 删除本地tag

    ```shell
    git tag -a v1.0.0 -m "新建标记"
    ```

* 删除远程标记

  git push origin --delete [tagName] 

* 推送远程仓储

  git push origin [版本号]
  
    * --tag: 推送全部tags


  ```shell
  git push origin v1.0.0
  ```