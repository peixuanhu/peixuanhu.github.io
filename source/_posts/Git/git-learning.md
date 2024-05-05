---
title: Git常用命令学习
categories:
- [Git]
---

#  Git常用命令学习

推荐一个可视化git学习网站：https://learngitbranching.js.org/

## 常用命令举例

### 本地

1. 作用：（強制）移動 main 指向從 HEAD 往上數的第三個 parent commit。

   ```bash
   git branch -f main HEAD~3
   # git branch -f main HEAD^: 往上移一个
   ```

2. 作用：把b及往父亲方向走直到和a的公共祖宗节点之前的路径上节点接到a的后面

   > 如果b是a的祖宗，则没有效果

   ```bash
   git rebase a b
   
   # 等价于现在HEAD在b，执行以下命令：
   git rebase a
   ```

   怎么记？rebase a也就是换base为a，把当前commit（b）的base换成a，b接到a的后面

3. 从当前commit新建一条分支`bugFix`，并切换到`bugFix`

   ```bash 
   git checkout -b bugFix
   # 等价于：
   git branch bugFix
   git checkout bugFix
   ```

4. 取消git修改：`git reset`或者`git revert`，例：

   ```bash
   git reset HEAD~3
   git revert HEAD
   ```

5. 把commit a, b, c依次接到当前指针所在commit的后面

   ```bash
   git cherry-pick a b c
   ```


### 远端

1. 表示一個 fetch 以及一個 rebase。用于远端其他人提交新commit之后更新到本地，并与本地自己的更新合并（适用于在同一个分支开发）

   ```bash
   git pull --rebase
   ```

   > 本地在dev分支开发，如果远端master更新了，需要把变更合并到本地dev（本地的master一般不会动，如果动了就加个`--rebase`）：
   >
   > ```bash
   > git checkout master
   > git pull
   > git checkout dev
   > git rebase master / git merge master
   > ```
   >
   > 

2. 建立一個新的 totallyNotMain branch 並且它會 track origin/main。

   ```bash
   git checkout -b totallyNotMain origin/main
   # 等价于
   git branch -u origin/main totallyNotMain
   ```

   

3. 先到我的 repo 中的 "main" branch，抓下所有的 commit，然後到叫作 "origin" 的 remote 的 "main" branch，檢查 remote 的 commit 有沒有跟我的 repo 一致，如果沒有，就更新。

   ```bash
   git push <remote> <place>
   # 如：
   git push origin main
   ```

   > 这个命令就不需要local当前指针一定指向main了

4. 當我們的 source 以及 destination 是不同的branch的時候用

   ```bash
   git push origin <source>:<destination>
   ```

   

5. git 會到 remote 上的 foo branch，抓下所有不在 local 上的 commit，然後將它們放到 local 的 o/foo branch。

   ```bash
   git fetch origin foo
   ```

   

6. git pull 表示 fetch 之後再 merge 所 fetch 的 commit，當使用 git fetch 時使用一樣的參數，之後再從 fetch 下來的 commit 所放置的位置做 merge。

   ```bash
   git pull origin foo
   # 等价于
   git fetch origin foo; git merge o/foo
   
   git pull origin bar~1:bugFix
   # 等价于
   git fetch origin bar~1:bugFix; git merge bugFix
   ```

   

## 对比

### reset vs revert

1. `reset`参数指针为当前指针所在branch需要回退到的位置；而`revert`参数指针为需要修改的那个指针
2. `reset`可以往回退很多个commit；而`revert`只能取消修改一个commit（后面跟的参数是指针）
3. `reset`相当于只把指针往回移动若干步，别人无法看到这个变化；而`revert`是把当前commit的前一个commit接到当前commit后面，对其他人也可见。

### rebase vs merge

`rebase`优点：rebase 使得你的 commit tree 看起來更為簡潔，因為任何的 commit 都在一條直線上面。

`rebase`缺点：rebase 修改了 commit tree 的歷史紀錄。