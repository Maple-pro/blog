---
title: git 开发流程
tags: [GIT]
categories: [记录]
date: 2021-09-14 20:34:21
mathjax: false
description: 记录常用的 git 开发流程
---

远程迭代分支：`origin sprint_0804`

本地迭代分支：`sprint_0804`

本地开发分支：`sprint_0804_dev`

```bash
# 1. 拉取远程迭代分支，建立本地迭代分支
git clone 'https://github.com/Maple-pro/test.git'
git(master): git checkout sprint_0804

# 2. 建立本地开发分支
git(sprint_0804): git branch sprint_0804_dev
git(sprint_0804): git checkout sprint_0804_dev

# 3. 本地开发分支提交
git(sprint_0804_dev): git add .
git(sprint_0804_dev): git commit -m 'feat: xxxxx'

# 4. 本地迭代分支同步
git(sprint_0804_dev): git pull origin sprint_0804:sprint_0804

# 5. 本地开发分支和本地迭代分支合并
git(sprint_0804_dev): git merge sprint_0804
# or
git(sprint_0804_dev): git rebase sprint_0804

# 6. 合并解决冲突
git(sprint_0804_dev): git add .
git(sprint_0804_dev): git commit -m 'Merge xxxxx'
# or
git(sprint_0804_dev): git add .
git(sprint_0804_dev): git rebase --continue

# 7. 提交到远程临时分支
git(sprint_0804_dev): git push origin sprint_0804_dev

# 8. 建立 pull request, 评审后合并分支

# 9. 根据评审意见修改代码再次提交
git(sprint_0804_dev): git add .
git(sprint_0804_dev): git commit -m 'fix: xxx'
git(sprint_0804_dev): git push origin sprint_0804_dev
# or
git(sprint_0804_dev): git reset sprint_0804
git(sprint_0804_dev): git add .
git(sprint_0804_dev): git commit -m 'feat: xxxxx'
git(sprint_0804_dev): git push origin sprint_0804_dev -f

# 10. 本地迭代分支同步
git(sprint_0804_dev): git pull origin sprint_0804:sprint_0804

# 11. 继续开发
git(sprint_0804_dev): git merge sprint_0804
```

