---
title: '快速开始本地开发'
description: '对 FastGPT 进行开发调试'
icon: 'developer_guide'
draft: false
toc: true
weight: 705
---

本文档介绍了如何设置开发环境以构建和测试 [FastGPT](https://fastgpt.in)。


## 前置依赖项

您需要在计算机上安装和配置以下依赖项才能构建 [FastGPT](https://fastgpt.in)：

- [Git](http://git-scm.com/)
- [Docker](https://www.docker.com/)（构建镜像）
- [Node.js v18.x (不推荐最新的，可能有兼容问题)](http://nodejs.org)
- [pnpm](https://pnpm.io/) 版本 8.x.x 

## 开始本地开发

{{% alert context="success" %}}
1. 用户默认的时区为 `Asia/Shanghai`,非 linux 环境时候，获取系统时间会异常，本地开发时候，可以将用户的时区调整成 UTC（+0）。
2. 建议先服务器装好**数据库**，再进行本地开发。
{{% /alert %}}

### 1. Fork 存储库

您需要 Fork [存储库](https://github.com/labring/FastGPT)。

### 2. 克隆存储库

克隆您在 GitHub 上 Fork 的存储库：

```
git clone git@github.com:<github_username>/FastGPT.git
```

**目录简要说明**

1. `projects` 目录下为 FastGPT 应用代码。其中 `app` 为 FastGPT 核心应用。（后续可能会引入其他应用）
2. NextJS 框架前后端放在一起，API 服务位于 `src/pages/api` 目录内。
3. `packages` 目录为共用代码，通过 workspace 被注入到 `projects` 中，已配置 monorepo 自动注入，无需额外打包。

### 3. 安装数据库

第一次开发，需要先部署数据库，建议本地开发可以随便找一台 2C2G 的轻量小数据库实践。数据库部署教程：[Docker 快速部署](/docs/development/docker/)。部署完了，可以本地访问其数据库。

Mongo 数据库需要注意，需要注意在连接地址中增加 `directConnection=true` 参数，才能连接上副本集的数据库。

### 4. 初始配置

以下文件均在 `projects/app` 路径下。

**1. 环境变量**

复制`.env.template`文件，在同级目录下生成一个`.env.local` 文件，修改`.env.local` 里内容才是有效的变量。变量说明见 .env.template

**2. config 配置文件**

复制 `data/config.json` 文件，生成一个 `data/config.local.json` 配置文件，具体配置参数说明，可参考 [config 配置说明](/docs/development/configuration)

**注意：json 配置文件不能包含注释，介绍中为了方便看才加入的注释**

这个文件大部分时候不需要修改。只需要关注 `systemEnv` 里的参数：

- `vectorMaxProcess`: 向量生成最大进程，根据数据库和 key 的并发数来决定，通常单个 120 号，2c4g 服务器设置 10~15。
- `qaMaxProcess`: QA 生成最大进程
- `pgHNSWEfSearch`: PostgreSQL vector 索引参数，越大搜索精度越高但是速度越慢，具体可看 pgvector 官方说明。

### 5. 运行

```bash
# 给自动化脚本代码执行权限(非 linux 系统, 可以手动执行里面的 postinstall.sh 文件内容)
chmod -R +x ./scripts/
# 代码根目录下执行，会安装根 package、projects 和 packages 内所有依赖
pnpm i
# 切换到应用目录
cd projects/app 
# 开发模式运行
pnpm dev
```

### 6. 部署打包

```bash
# 根目录下执行
docker build -t dockername/fastgpt:tag --build-arg name=app .
# 使用代理
docker build -t dockername/fastgpt:tag --build-arg name=app --build-arg proxy=taobao .
```

## 提交代码至开源仓库

1. 确保你的代码是 Fork [FastGPT](https://github.com/labring/FastGPT) 仓库
2. 尽可能少量的提交代码，每次提交仅解决一个问题。
3. 向 FastGPT 的 main 分支提交一个 PR，提交请求后，FastGPT 团队/社区的其他人将与您一起审查它。

如果遇到问题，比如合并冲突或不知道如何打开拉取请求，请查看 GitHub 的[拉取请求教程](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests)，了解如何解决合并冲突和其他问题。一旦您的 PR 被合并，您将自豪地被列为[贡献者表](https://github.com/labring/FastGPT/graphs/contributors)中的一员。



## QA

### 本地数据库无法连接

1. 如果你是连接远程的数据库，先检查对应的端口是否开放。
2. 如果是本地运行的数据库，可尝试`host`改成`localhost`或`127.0.0.1`

### sh ./scripts/postinstall.sh 没权限

FastGPT 在`pnpm i`后会执行`postinstall`脚本，用于自动生成`ChakraUI`的`Type`。如果没有权限，可以先执行`chmod -R +x ./scripts/`，再执行`pnpm i`。

### 长时间运行后崩溃

似乎是由于 tiktoken 库的开发环境问题，生产环境中未遇到，暂时可忽略。

### TypeError: Cannot read properties of null (reading 'useMemo' )

删除所有的`node_modules`，用 Node18 重新 install 试试，可能最新的 Node 有问题。 本地开发流程：

1. 根目录: `pnpm i`
2. 复制 `config.json` -> `config.local.json`
3. 复制 `.env.template` -> `.env.local`
4. `cd projects/app`
5. `pnpm dev`


## 加入社区

遇到困难了吗？有任何问题吗? 加入微信群与开发者和用户保持沟通。

<img width="400px" src="https://oss.laf.run/htr4n1-images/fastgpt-qr-code.jpg" class="medium-zoom-image" />

## 代码结构说明

### nextjs

FastGPT 使用了 nextjs 的 page route 作为框架。为了区分好前后端代码，在目录分配上会分成 global, service, web 3个自目录，分别对应着 `前后端共用`、`后端专用`、`前端专用`的代码。

### monorepo
FastGPT 采用 pnpm workspace 方式构建 monorepo 项目，主要分为两个部分：

- projects/app - FastGPT 主项目  
- packages/ - 子模块  
  - global - 共用代码，通常是放一些前后端都能执行的函数、类型声明、常量。  
  - service - 服务端代码  
  - web - 前端代码  
  - plugin - 工作流自定义插件的代码  

### 领域驱动模式（DDD）

FastGPT 在代码模块划分时，按DDD的思想进行划分，主要分为以下几个领域：

core - 核心功能（知识库，工作流，应用，对话）
support - 支撑功能（用户体系，计费，鉴权等）
common - 基础功能（日志管理，文件读写等）

{{% details title="代码结构说明" closed="true" %}}
```
.
├── .github                      // github 相关配置
├── .husky                       // 格式化配置
├── docSite                      // 文档
├── files                        // 一些外部文件，例如 docker-compose, helm
├── packages                     // 子包
│   ├── global                   // 前后端通用子包
│   ├── plugins                  // 工作流插件（需要自定义包时候使用到）
│   ├── service                  // 后端子包
│   └── web                      // 前端子包
├── projects
│   └── app                      // FastGPT 主项目
├── python                       // 存放一些模型代码，和 FastGPT 本身无关
└── scripts                      // 一些自动化脚本
    ├── icon                     // icon预览脚本，可以在顶层 pnpm initIcon(把svg写入到代码中), pnpm previewIcon（预览icon）
    └── postinstall.sh           // chakraUI自定义theme初始化 ts 类型
├── package.json                 // 顶层monorepo
├── pnpm-lock.yaml
├── pnpm-workspace.yaml          // monorepo 声明
├── Dockerfile
├── LICENSE
├── README.md
├── README_en.md
├── README_ja.md
├── dev.md
```
{{% /details %}}
