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
git add 
```

### 本地仓库

```shell
git commit 
```


将文件保存在本地仓库


### 推送至远程仓库

```shell
# 连接远程仓库

git push 
```

## 二. 从远处仓库开始

```shell
git clone #复制现有仓库（从远程下载）

eg:
git clone git@github.com:wanghao0409/future.git

git clone git@github.com:wanghao0409/ComputerNote.git

git clone git@github.com:wanghao0409/ObsidianNote.git

```





# SSH 认证
```shell
ssh-keygen -t ed25519 -C "your_email@example.com"

# -t 压缩算法类型
# -C 注释

# E
ssh-keygen -t ed25519 -C "398708416@qq.com"

ssh-keygen -t rsa

cat ~/.ssh/id_rsa.pub
```





`

# 冲突解决








