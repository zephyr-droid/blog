# GitHub Setup Guide

这篇文章记录了CachyOS配置GitHub仓库的基本流程。

## 安装GitHub CLI

执行：

```shell
sudo pacman -S github-cli
```

## 生成新SSH密钥

执行以下命令，将`your_email@example.com`替换为GitHub邮箱：

```shell
ssh-keygen -t ed25519 -C "your_email@example.com"
```

按回车键接受默认值

## 将SSH密钥添加到ssh代理

1. 启动ssh代理：
   
   ```shell
   eval (ssh-agent -c)
   ```

2. 将SSH私钥添加到ssh代理：
   
   ```shell
   ssh-add ~/.ssh/id_ed25519
   ```

## 身份验证

执行：

```shell
gh auth login
```

按照提示操作

## 创建存储库

执行：

```shell
gh repo create
```

## 初始化配置

执行以下命令，初始化用户信息，否则无法提交代码：

```shell
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
git config --global init.defaultBranch main
```

## 提交更改

1. 创建自述文件：
   
   ```shell
   echo "info about this project" >> README.md
   ```

2. 暂存并提交文件：
   
   ```shell
   git add README.md && git commit -m "feat: add readme"
   ```

3. 推送更改：
   
   ```shell
   git push --set-upstream origin main
   ```

## 参考

- [仓库快速入门](https://docs.github.com/zh/repositories/creating-and-managing-repositories/quickstart-for-repositories)
