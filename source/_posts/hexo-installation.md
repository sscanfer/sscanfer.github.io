---
title: 基于 hexo 的个人博客搭建
date: 2022-06-11 11:11:11
categories: 工具环境配置
tags: hexo
---

# 基于 hexo 的个人博客搭建

>参考 [手把手教你从0开始搭建自己的个人博客 |无坑版视频教程| hexo](https://www.bilibili.com/video/BV1Yb411a7ty)

## 环境准备

### 1. 安装 nodejs

> 下载地址: <https://nodejs.org>

安装完成包含两个组件: nodejs, npm. 使用下面的命令查看是否安装成功:

``` shell
node -v # 查看 node 版本

npm -v  # 查看 npm 版本
```

### 2. 更换国内镜像源 cnpm

`npm` 国内镜像源慢, 可以使用下面的命令安装 `cnpm` 解决:

``` shell
npm install -g cnpm --registry=https://registry.npm.taobao.org # -g 全局安装

cnpm -v                                                        # 查看 cnpm 版本
```

### 3. 使用 cnpm 安装 hexo

使用下面的命令安装 hexo 并验证是否安装成功:

```shell
cnpm install -g hexo-cli # 安装 hexo

hexo -v                  # 查询 hexo 版本
```

到这里前期环境准备完毕.

---

## hexo 配置

### 1. 新建存放博客的目录, 并切换进入

```shell
mkdir blog # 比如当前在 D:

cd blog    # 进入目录
```

### 2. hexo 初始化

```shell
hexo init  # 初始化
```

### 3. 启动博客预览

```shell
hexo s     # hexo server 启动博客预览, 默认可通过 http://localhost:4000 访问, ctrl+c 即可停止
```

### 4. 新建博客文章

```shell
hexo n "my_first_blog" # hexo new 默认生成 my_first_blog.md, 位于 D:/blog/source/_posts/ 路径下
```

### 5. 编辑后需要先清理

```shell
hexo clean # 编辑 my_first_blog.md 并保存后, 先清理本地
```

### 6. 生成静态文件

```shell
hexo g # hexo generate 生成静态文件后, 重新 hexo s 即可启动预览
```

完成 hexo 本地配置和使用之后, 下一步部署到远端服务器上, 比如 github.io, gitee.io

---

## 部署个人博客到远端

### 1. cnpm 安装 hexo-deployer-git

```shell
cnpm install --save hexo-deployer-git # 安装过程 WARNING 可忽略
```

### 2. 建立远端仓库

- **github page** 则新建仓库名为 `sscanfer.github.io`, 即可直接通过 <https://sscanfer.github.io/> 访问

- ***gitee page*** 则新建仓库名为 `sscanfer`, 即可直接访问 <https://sscanfer.gitee.io/> 访问

> 注意 `sscanfer` 为 github username 或 gitee 个人空间地址 (可在 "设置-基本设置-个人资料-个人空间地址" 修改)

### 3. 本地配置 _config.yml

以 github page 为例:

```yml
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repo: https://github.com/sscanfer/sscanfer.github.io.git
  branch: master
```

### 4. 部署到远端

```shell
hexo d # hexo deploy 部署到远端
```

至此基于 hexo 的个人博客部署完毕. 下面提供多设备提交更新博客的解决方案.

---

## 多设备提交和更新博客的解决方案

> 参考文章:
>
> <https://www.jianshu.com/p/0b1fccce74e0/>
>
> <https://blog.csdn.net/ShmilyCoder/article/details/79916973>
>
> 主要思路: 利用 git 分支实现.
>
> hexo 生成的静态博客文件默认放在 `master` 分支上. 新建 `hexo` 分支, 然后将 hexo 的源文件 (部署环境文件) 都放在 `hexo` 分支上, 换新设备时, 直接 `git clone hexo` 分支即可

### 1. 对 sscanfer.github.io 仓库新建 hexo 分支, 并克隆到本地

- 新建分支: 在 **Github** 的 `sscanfer.github.io` 仓库上新建一个 `hexo` 分支, 并切换到该分支
- 设置为默认分支: 在该仓库->Settings->Branches->Default branch中将默认分支设为 `hexo`, save 保存
- 克隆到本地: 将该仓库克隆到本地，进入该 `sscanfer.github.io` 文件目录。

### 2. 将本地博客的部署文件拷贝进 `sscanfer.github.io` 目录

- 将本地博客的部署文件**全部**拷贝进 `sscanfer.github.io` 目录
- 将 `themes` 目录内的 `.git` 目录删除 (如果有), 因为一个 git 仓库不能包含另一个 git 仓库
- 提交至新建的 `hexo` 分支

> 删除 `themes` 目录下的 `.git` 目录就不能 `git pull` 更新主题的解决办法:
>>单独 `git clone` 主题最新版本, 然后拷贝到当前主题目录替换即可

### 3. 新设备的同步方法

- 克隆 `hexo` 分支到本地
- 切换到 `sscanfer.github.io` 目录, 执行 `npm install`. (由于仓库有一个 `.gitignore` 文件, 里面默认是忽略掉 `node_modules` 文件夹的, 也就是说仓库的 `hexo` 分支并没有存储该目录 (也不需要), 所以需要install下)
