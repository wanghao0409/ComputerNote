#  工作原理


## 一.从零开始
### 工作区

```shell
git init 

# 创建**新的空仓库**（从零开始）
# 作用： 创建一个.git 
```
 

### 暂存区

```shell
git add . # 提交所有文件到暂存区
git add * # 不包含.开头的隐藏文件
```

### 本地仓库

将文件保存在本地仓库

```shell
git commit -m '提交信息'
```





### 推送至远程仓库

```shell
# 连接远程仓库

git push 
```

### 分支管理
```shell
git branch  # 查看本地有哪些分支

git branch -r # 查看远程分支

git checkout dev # 切换至某个分支
```


1

## 二. 从远处仓库开始

```shell
git clone #复制现有仓库（从远程下载）

eg:
git clone git@github.com:wanghao0409/future.git

git clone git@github.com:wanghao0409/ComputerNote.git

git clone git@github.com:wanghao0409/ObsidianNote.git

git clone git@github.com:wanghao0409/Language.git

```





# SSH 认证
```shell
ssh-keygen -t ed25519 -C "your_email@example.com"

# -t 压缩算法类型
# -C 注释

# Eg: 
ssh-keygen -t ed25519 -C "398708416@qq.com"

ssh-keygen -t rsa 

cat ~/.ssh/id_rsa.pub
```





`

# 冲突解决



丢弃本地仓库所有修改，同步远程最新


不需要删除仓库重新 clone，可以通过 Git 命令直接放弃所有本地修改，将本地仓库重置到远程分支的最新状态。这样做更快捷，也不会影响 Git 的配置和已有的远程关联。

### 操作步骤（假设要同步 `origin/main` 分支）

1. **获取远程最新信息**  
   ```bash
   git fetch --all --prune
   ```
   - `--all`：获取所有远程仓库（通常只有 `origin`）的更新。
   - `--prune`：删除远程已不存在的分支的本地跟踪引用。

2. **彻底重置当前分支**  
   ```bash
   git reset --hard origin/main
   ```
   - 将当前分支（例如 `main`）强制指向 `origin/main` 的提交。
   - 会丢弃所有未提交的修改和已提交但未推送的本地提交。

3. **清理未跟踪的文件和目录**  
   ```bash
   git clean -fd
   ```
   - `-f`：强制删除（必须）。
   - `-d`：同时删除未跟踪的目录。
   - 如果你还想删除 `.gitignore` 中忽略的文件（谨慎），可以加 `-x`。

### 完整命令示例（一条条执行）
```bash
# 进入仓库目录
cd your-repo

# 获取远程最新状态
git fetch --all --prune

# 重置到远程 main 分支（确保当前在 main 分支，或切换过去）
git checkout main
git reset --hard origin/main

# 删除所有未跟踪的文件和目录
git clean -fd
```

### 注意事项
- **操作不可逆**：上述命令会永久删除本地所有未提交的更改、未推送的提交以及未跟踪的文件。执行前请确认不再需要这些内容。
- **如果本地有其他分支**：如果你在其他分支上有未推送的提交，而你想保留它们，不要在主分支执行 `reset --hard` 后再切换；如果你也想放弃其他分支的修改，可以对每个分支重复类似操作，或使用 `git fetch --all` 后 `git reset --hard origin/<branch>`。
- **更安全的方式**：如果你只是想暂时丢弃修改，可以先 `git stash` 保存，确认无误后再 `git stash drop`。但既然目标是“完全放弃”，直接重置更简洁。

### 为什么不用重新 clone？
重新 clone 也能达到目的，但需要重新下载整个仓库，耗费时间和网络流量。而使用 Git 命令仅在本地进行指针移动和文件清理，几乎瞬间完成。只有在本地仓库严重损坏（如对象数据库损坏）时，才需要 clone 修复。

### 总结
你不需要删除仓库重新 clone。只需执行 `git fetch`、`git reset --hard` 和 `git clean` 组合命令，就能让本地仓库与远程完全同步。



