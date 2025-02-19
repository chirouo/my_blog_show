---

title: Git学习总结

description: 马上要做毕设等大项目了，想总结一下Git的常用命令，方便以后所有项目均用Git管理，提高编码效率；同时多设备同步也比较方便。

date: 2025-02-19

# slug: git

image: https://raw.githubusercontent.com/chirouo/my_blog_img/main/xdc1.jpg

categories:
  - 工具
tags:
  - Git

---

## 目录

- [宏观git](#宏观git)
- [git reset 命令](#git reset 命令)
- [git checkout 命令](#git checkout 命令)
- [git diff 命令](#git diff 命令)
- [git stash 命令](#git stash 命令)
- [git log 命令](#git log 命令)
- [git cherry-pick 命令](#git cherry-pick 命令)
- [git rebase 和 git merge 的区别](# git rebase 和 git merge 的区别)
- [git 多分支开发工作流](#git 多分支开发工作流)
  - [基础操作](#基础操作)
  - [分支管理](#分支管理)
  - [工作流程](#工作流程)
  - [规范标准](#规范标准)

## 宏观git

drop和show可以+栈索引

![1](https://raw.githubusercontent.com/chirouo/my_blog_img/main/image-20250219115120890.png)

## git reset 命令

```bash
# 基本语法
git reset [--soft | --mixed | --hard] [commit_id]
reset默认不加任何参数的时候是--mixed行为
# 三种模式
--soft   # 只重置HEAD指针
--mixed  # 重置HEAD指针和暂存区
--hard   # 重置HEAD指针、暂存区和工作区
```

- --soft模式

  ```bash
  git reset --soft HEAD~n
  # 效果：
  # 1. 只移动HEAD指针
  # 2. 撤销最近n次的提交，但保留所有修改在暂存区中，可以重新提交。
  # 3. 常用于修改最近的commit信息
  ```

- --mixed模式（默认）

  ```bash
  git reset [HEAD~n | HEAD fileA]
  # 或
  git reset --mixed [HEAD~n | HEAD fileA]
  # 效果：
  # 1. 移动HEAD指针
  # 2. 重置暂存区到前n次的版本（不会丢失任何更改，只是将文件状态从"已暂存"变回"已修改"，如果指定了文件fileA，就只对这一个文件进行这样的操作）
  # 3. 保留工作区的修改
  # 4. 常用于合并多个提交
  ```

- --hard模式

  ```bash
  git reset --hard HEAD~n
  # 效果：
  # 1. 移动HEAD指针
  # 2. 重置暂存区和工作区
  # 3. 危险操作：会丢失未提交的修改（直接回到前n次的版本，工作区和暂存区都没有任何缓存）
  # 4. 常用于完全放弃最近的提交
  ```

## git checkout 命令

checkout + 分支，可以切分枝， +文件，可以把工作区的改动撤销

```bash
# 切换分支
git checkout branch_name

# 创建并切换分支
git checkout -b new_branch

# 从工作区检出（删除）特定文件
git checkout -- file_name
```

## git diff 命令

- 使用示例

  ```bash
  # 比较暂存区和工作区的差异	（工作区有，暂存区没有，显示+）
  git diff [-- path/file_name]
  # 比较版本库和暂存区中的差异	（暂存区有，版本库中没有，显示+）
  git diff --cached [-- path/file_name]
  # 比较版本库和工作区的差异	（工作区有，版本库中没有，显示+）
  git diff HEAD [-- path/file_name]
  # 比较不同提交之间的差异	（to_commit有，from_commit中没有，显示+）
  git diff <from_commit> <to_commit> [-- path/file_name]
  # 比较不同分支之间的差异	（branchA有，branchB中没有，显示+）
  git diff main develop
  ```

- 最佳实践

  ```bash
  # 检查即将提交的内容（即检查版本库和暂存区之间的差异）
  git diff --cached
  
  # 查看完整的修改
  git status        # 先查看文件状态
  git diff          # 查看未暂存的改动（即检查暂存区和工作区之间的差异）
  git diff --cached # 查看已暂存的改动
  ```

- 对运行完git diff命令后类似@@ -xxx,x +xxx,x @@的解释

  ![image-20250219151522377](https://raw.githubusercontent.com/chirouo/my_blog_img/main/image-20250219151522377.png)

## git stash 命令

- 入栈

  ```bash
  git stash
  git stash save "comment"
  git stash push -m（新版本）
  ```

- 出栈

  ```bash
  # 应用并删除最近的暂存
  git stash pop
  # 应用最近的暂存（但是不删除）
  git stash apply
  ```

- 删除

  ```bash
  # 清除栈中所有暂存
  git stash clear
  # 删除指定stash_id的暂存
  git stash drop stash@{n}
  ```

- 查

  ```bash
  # 列出所有暂存
  git stash list
  # 展示指定stash_id的暂存
  git stash show stash@{n}
  ```

  

## git log 命令

## git 多分支开发工作流git cherry pick 命令

![image-20250219145455621](https://raw.githubusercontent.com/chirouo/my_blog_img/main/image-20250219145455621.png)

```bash
# 第一种
git cherry-pick 4_commit_id
# 第二种
git cherry-pick 4_commit_id 8_commit_id		merge 4和8 变成 4’和8’后放到当前分支上
# 第三种
git cherry-pick 4_commit_id..8_commit_id	merge (4, 8]变成(4,8]’后放到当前分支上
# 第四种
git cherry-pick 4_commit_id^..8_commit_id	merge [4, 8]变成[4,8]’后放到当前分支上
```



## git rebase 和 git merge 的区别

- 在main上 git merge develop：

  ![image-20250219123757293](https://raw.githubusercontent.com/chirouo/my_blog_img/main/image-20250219123757293.png)

- 在 main 上 git rebase develop：

  会把develop的`4`和`6`节点嫁接到main和develop的公共parent`3`之后

  再把main在parent`3`之后的commit节点7 和 5 变成变成**新的commit** **节点**`7'` `5'`放到develop之后

  ![image-20250219123253615](https://raw.githubusercontent.com/chirouo/my_blog_img/main/image-20250219123253615.png)

  git rebase会打乱时间顺序，但是合并之后很清爽，自己舍取优缺点

- 应用场景

  ```
  	一个新的需求，就按这个需求开一个新分支，当主分支有新增内容时（其他小伙伴合入代码了），且新增的内容对自己的新代码有影响，那么就rebase下主分支，将主分支的内容同步过来，并解决下自己的代码，比如冲突啥的，适配主分支的新代码。自己的需求开发完毕后，提MR，merge合入到主分支
  	尽量不要在公共分支rebase，公共分支例如main，用merge
  ```

## git 多分支开发工作流

### 基础操作

- 远程仓库操作

  ```bash
  # 查看远程仓库
  git remote -v
  
  # 查看所有分支
  git branch -a
  
  # 更新远程分支信息
  git fetch origin
  
  # fetch与pull的区别
  git fetch  # 只下载，不合并
  git pull   # 下载并合并
  ```

### 分支管理

- 主要分支结构

  ```bash
  main/master    # 主分支，稳定版本
  develop        # 开发分支
  feature/*      # 功能分支
  hotfix/*       # 修复分支
  release/*      # 发布分支
  ```

- 分支命名示例

  ```bash
  feature/login     # 新功能
  hotfix/bug-fix    # 紧急修复
  release/v1.0.0    # 发布版本
  docs/api          # 文档更新
  ```

### 工作流程

- 功能开发流程

  ```bash
  # 创建功能分支
  git checkout develop
  git checkout -b feature/new-feature
  
  # 开发并提交
  git add .
  git commit -m "feat: new feature"
  git push -u origin feature/new-feature
  
  # 完成合并
  git checkout develop
  git merge feature/new-feature
  ```

- 发布流程

  ```bash
  # 创建发布分支
  git checkout -b release/v1.0.0 develop
  
  # 完成发布
  git checkout main
  git merge release/v1.0.0
  git tag v1.0.0
  ```

### 规范标准

- 提交信息规范

  ```bash
  feat: 新功能
  fix: 修复bug
  docs: 文档更新
  style: 格式调整
  refactor: 重构
  test: 测试相关
  chore: 构建/工具
  ```

- 最佳实践

  ```bash
  1. 定期同步上游更新
  2. 功能完成提交PR
  3. 保持提交信息清晰
  4. 及时清理已合并分支
  5. 重要节点打Tag标记
  ```

  