# learn-lerna

[![lerna](https://img.shields.io/badge/maintained%20with-lerna-cc00ff.svg)](https://lernajs.io/)

学习 lerna 的使用。最好的方式，就是亲手操作一遍

learn 差点死掉，之间我切换 [turborepo](https://turbo.build/) 玩了一阵, lerna 被 nx 收购，起死回生了，并且有个较大提升，同时 pnpm 支持也还好。

- [learn-lerna](#learn-lerna)
  - [项目初始化](#项目初始化)
    - [环境](#环境)
    - [示例项目](#示例项目)
    - [lerna与 pnpm 的 monorepo配置](#lerna与-pnpm-的-monorepo配置)
    - [lerna scripts 配置](#lerna-scripts-配置)
    - [问题](#问题)
    - [nx 缓存](#nx-缓存)
    - [关于发布](#关于发布)
  - [以下是 learn@3.x 的学习](#以下是-learn3x-的学习)
  - [Workspaces](#workspaces)
  - [其他](#其他)


## 项目初始化

### 环境

- node@16.16.0
- pnpm@7.27.0
- lerna@6.4.1
- nx@15.6.3

### 示例项目

- [lerna-monorepo-example](https://github.com/cloudyan/lerna-monorepo-example)
  - 分支
    - main(使用 npm)
    - pnpm
- [lowcode-monorepo](https://github.com/cloudyan/lowcode-monorepo)

### lerna与 pnpm 的 monorepo配置

```bash
npx lerna init
```

- [将 pnpm 与 lerna 结合使用](https://lerna.js.org/docs/recipes/using-pnpm-with-lerna)
  - 默认情况下，lerna 使用 package.json 中的 workspaces 属性来搜索包。
  - 如果使用 pnpm，lerna.json 中已设置 `npmClient: 'pnpm',`，这种情况下，lerna 将使用 pnpm-workspace.yaml 来搜索包。

``` js
// 1. lerna.json 添加以下配置
"npmClient": "pnpm",

// 2. 将以下配置
// package.json
{
  "workspaces": ["packages/*"]
}
// lerna.json
{
  "packages": ["packages/*"]
}

// 3. 移动到 pnpm-workspace.yaml
// pnpm-workspace.yaml
packages:
  - "packages/*"
```

最终 lerna.json

```json
{
  "$schema": "node_modules/lerna/schemas/lerna-schema.json",
  "useWorkspaces": true,
  "npmClient": "pnpm",
  "version": "0.0.0",
  "command": {
    "version": {
      "allowBranch": "release",
      "conventionalCommits": true
    }
  }
}
```

### lerna scripts 配置

执行 script task，可参考文档 [Run Tasks](https://lerna.js.org/docs/features/run-tasks)以及[示例库](https://github.com/lerna/getting-started-example)

```bash
# 多个项目
npx lerna run build --scope=header,footer
npx lerna run build --ignore=header,footer
# 支持 glob
npx lerna run --scope package-1 --scope "*-2" lint
npx lerna run --scope="package-{1,2,5}" test
npx lerna run --ignore "package-@(1|2)" --ignore package-3 lint

# umi4 为对应的 package name
"demo": "lerna run dev --scope=umi4",
"dev": "lerna run dev --ignore=umi4",

# 直接运行库和应用
"dev": "lerna run dev",
```

构建依赖是自动识别的，无需 nx.json 就能完成自动构建依赖的管理

> NOTE: 不要这样 `npm run dev && npm run demo`
> 上面的命令开发运行时，包的构建命令不会退出，所以就没有 umi4 构建的启动

如果要将运行脚本的进程数增加到 5（默认情况下为 3），请传递以下内容：

```bash
npx lerna run build --concurrency=5
```

此时可以通过 `--concurrency=10` 参数增加并行任务数。

解释: 此处参考 [--parallel](https://lerna.js.org/docs/lerna6-obsolete-options#--parallel) 中的解释，lerna 使用任务图来确认哪些任务可以并行运行并自动运行。

> 如果你想限制任务的并发，你仍然可以使用[并发全局选项 --concurrency](https://github.com/lerna/lerna/blob/6cb8ab2d4af7ce25c812e8fb05cd04650105705f/core/global-options/README.md#--concurrency)来完成这个。
> `--concurrency` Lerna 并行化任务时使用多少线程（默认为**逻辑 CPU 核心数**）。
> 此处参数指定为限制并行的数量

- 1. 并不是一定依赖 `@nrwl/nx-cloud` 需要使用云端配置
- 2. 下方的 nx.json 同样支持并行任务

> 目前并行任务多，有点卡，还需要测试验证。
> 怀疑根因就是并行任务太多，超过逻辑 CPU 核心数时，性能降低。
> 当计算机没有开启超线程时，逻辑CPU的个数就是计算机的核数。而当超线程开启后，逻辑CPU的个数是核数的两倍。

如何查看核心数

以 `MacBook Pro (13-inch, 2020, Two Thunderbolt 3 ports)` 为例

参考: https://cloud.tencent.com/developer/ask/sof/244076

- `1.4 GHz 四核Intel Core i5`，这里是物理核心 4 核
- 逻辑核心：查看活动监视器 => 双击 CPU 负载面板 => CPU历史记录 这里展示了所有的内核及数量
- mac 可以使用命令查看
  - `system_profiler SPHardwareDataType`
  - `sysctl -n hw.logicalcpu`
  - `sysctl -n hw.physicalcpu`
  - `sysctl -n hw.ncpu`
  - `getconf _NPROCESSORS_ONLN` 在Mac OS X和Linux中都可以工作
  - `sysctl -a | sort | grep cpu` 提供有关 cpu 的所有信息(推荐)

### 问题

那么在 lowcode-monorepo 中不依赖 nx.json 仅调整并行数可以吗？

经验证不可以，lerna 构建依赖是自动识别的。这导致了 dev 依赖执行存在问题。推测原因如下

特定项目的依赖的构建未完成，导致该特定项目无法开始构建

解决方案有两个

1. 可以拆分为两个命令，比如 dev,lowcode-demo
2. nx.json 中不要添加 dev 的依赖配置管理

还有问题：关于 nx.json 中的 `targetDefaults` dependsOn，配置与不配置的区别是什么？

详细参考：[target-defaults](https://lerna.js.org/docs/api-reference/configuration#target-defaults), 这里理解的还不是很深刻。

关于执行顺序，参考 [add-caching](https://github.com/lerna/lerna/tree/main/packages/lerna/src/commands/add-caching#readme)

### nx 缓存

默认情况下，lerna（通过 nx）使用本地计算缓存。（缓存存储一周），要清除缓存运行 `nx reset`

```bash
npx lerna add-caching
```

如要共享缓存，可参考 https://lerna.js.org/docs/features/share-your-cache

nx.json 最终配置，更多配置项参考 [Nx.json文件](https://lerna.js.org/docs/api-reference/configuration)

```json
{
  "extends": "nx/presets/npm.json",
  "tasksRunnerOptions": {
    "default": {
      "runner": "nx/tasks-runners/default",
      "options": {
        "cacheableOperations": [
          "dev",
          "build"
        ]
      }
    }
  },
  "targetDefaults": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["{projectRoot}/dist"]
    }
  }
}
```

这里可通过 `npx lerna add-caching` 配置缓存配置，并添加 `"extends": "nx/presets/npm.json",`

> 请注意，旧版本的 Nx 使用 targetDependencies 而不是 targetDefaults。两者仍然有效，但建议使用 targetDefaults。
> 该^符号（又名插入符号）仅表示依赖关系
> `"build": { "dependsOn": ["^build"] }` 意味着特定项目的构建取决于该项目的所有依赖的构建目标的完成

```js
// nx.json
"targetDefaults": {
  "dev": {
    "dependsOn": ["^dev"] // 此处不能加，原因是特定项目的依赖的构建未完成，导致该特定项目无法开始构建(可以拆分为两个命令，比如 dev,lowcode-demo)
  },
  "build": {
    "dependsOn": ["^build"],
    "outputs": ["{projectRoot}/dist"]
  }
}
```

其他的一些配置参考

- https://github.com/lerna/getting-started-example/blob/main/nx.json
- https://lerna.js.org/docs/concepts/dte-guide
  - https://github.com/vsavkin/lerna-dte/blob/main/nx.json
- 关于 lerna(nx) 有必要了解下[缓存的工作原理](https://lerna.js.org/docs/concepts/how-caching-works)
  - 默认情况下存储在缓存中`node_modules/.cache/nx`,如要更改
  - 如果要跳过缓存 `npx lerna run build --skip-nx-cache`
- 关于 [分布式任务](https://lerna.js.org/docs/concepts/dte-guide)
  - [示例 repo](https://github.com/vsavkin/lerna-dte)
  - https://nx.dev/core-features/distribute-task-execution
  - https://github.com/vsavkin/interstellar
- [ci: github actions](https://nx.dev/recipes/ci/monorepo-ci-github-actions#distributed-ci-with-nx-cloud)

```js
//
"tasksRunnerOptions": {
  "default": {
    "runner": "@nrwl/nx-cloud",
    "options": {
      "accessToken": "MzhiZmU3ODItNmUxYS00ZDg2LTg0NWQtMjRiNGFjNTBmM2I4fHJlYWQtd3JpdGU=",
      "cacheableOperations": [
        "dev",
        "build",
        "test"
      ],
      "parallel": 10,
      "canTrackAnalytics": false,
      "showUsageWarnings": true,

      // 更改缓存位置, 默认 `node_modules/.cache/nx`
      "cacheDirectory": "/tmp/mycache"
    }
  }
},
```

### 关于发布

```bash
lerna version --no-private
lerna publish --no-private
```

lerna 版本控制，支持两种模式

1. Fixed/Locked mode (default) 固定/锁定模式
   1. 如果您想自动将所有包版本绑定在一起，请使用此选项
   2. 版本保存在 lerna.json 的 version 字段中
2. Independent mode 独立模式
   1. 启用该模式，将 lerna.json 中的 version 字段配置为 `independent`


## 以下是 learn@3.x 的学习

---------------------------------------------------------

参考：

- [monorepo 新浪潮](https://juejin.im/entry/586f00bc128fe100580a6f78)
  - [用lerna-changelog 来梳理 changelog](https://github.com/lerna/lerna-changelog)
  - [使用 monorepo 结构，管理多个 repo(示例)](https://github.com/galaxybing/lerna-repos-init.git)
- [lerna管理前端packages的最佳实践](https://juejin.im/post/5a989fb451882555731b88c2)
- [lerna 中文文档](https://github.com/chinanf-boy/lerna-zh)
  - [常见问题](https://github.com/chinanf-boy/lerna-zh/blob/master/FAQ.zh.md)
- [https://lernajs.io/](https://lernajs.io/)
  - [lerna-wizard lerna的命令行向导](https://github.com/szarouski/lerna-wizard)
  - [使用lerna管理大型前端项目](https://www.jianshu.com/p/2f9c05b119c9)
- [lerna-yarn-workspaces-example](https://github.com/Quramy/lerna-yarn-workspaces-example)
- 独立模式 [lerna-semantic-release](https://github.com/atlassian/lerna-semantic-release/blob/caribou/package.json)


**管理多个 repo ：**

- 单个 lint、build、test 和 release 流程
- 统一的地方处理 issue
- 不用到处去找自己项目的 repo
- 方便管理版本和 dependencies
- 跨项目的操作和修改变得容易
- 方便生成总的 changelog

useWorkspaces 应该是只针对 yarn 的

## Workspaces

Using [yarn workspace feature](https://yarnpkg.com/en/docs/workspaces), configure the following files:

- /package.json

Append the `workspaces` key.

```json
{
  "private": true,
  "workspaces": [
    "packages/*"
  ]
}
```

- lerna.json

Set `npmClient` `"yarn"` and turn `useWorkspaces` on.

```json
{
  "lerna": "2.2.0",
  "packages": [
    "packages/*"
  ],
  "npmClient": "yarn",
  "useWorkspaces": true,
  "version": "1.0.0"
}
```

Exec `yarn install`(or `lerna bootstrap`). After successful running, all dependency packages are downloaded under the repository root `node_modules` directory.

## 其他

建议使用 `npx`, 如 `npx lerna init`

```bash
# 全局安装工具，除了 Lerna,Builder，还可以像Andre Staltz 一样自己用脚本（通过Bash s）来实现 monorepo
npm install -g lerna

# 创建\初始化.git仓库
git init lerna-repo

cd lerna-repo

或
mkdir lerna-repo && cd $_

# 初始化管理目录，同时之后手动配置子 package 间的调用关系：dependencies
lerna init

# 会为各个 package 执行 npm install 所有的外部依赖；
# 并为内部依赖的 package 建立 symlink，对所有的 package 执行 npm prepublish
# bootstrap 将把repo中的依赖关系链接在一起.
lerna bootstrap

lerna bootstrap --hoist ? 这个是什么

# 更新版本(不用主动去更新，直接执行发布，根据提示选择操作即可)
# 使用与 npm version 相同的语法，更新版本号，如
lerna version patch

# Changes:
# - @xmini/package-1: 0.0.1 => 0.0.2
# - @xmini/package-2: 0.0.1 => 0.0.2

# 可以为指定包添加依赖
lerna add @types/node --scope=@xmini/package-1
lerna add --dev typescript --scope=@xmini/package-1

npm install jest --only=dev

# updated 可以查看哪些包发生了改变
lerna updated

# 发布到 npm
# publish将帮助发布任何更新的包（如果包未更新，会忽略）
lerna publish
```

Set up yarn的workspaces模式

https://juejin.im/post/5ced1609e51d455d850d3a6c

- `lerna init` 初始化项目
- `lerna bootstrap` 安装依赖
  - 默认是npm, 而且每个子package都有自己的node_modules
  - 配置 `yarn+workspaces` 后，只有顶层有一个node_modules
- `lerna list` 列出所有的包
- `lerna create <name> [loc]` 创建一个包 默认放在 `workspaces[0]`所指位置
- `lerna run <script>` 运行所有包里面的有这个script的命令
- `lerna exec` 运行任意命令在每个包
- `lerna clean` 删除所有包的node_modules目录
- `lerna changed` 列出下次发版lerna publish 要更新的包。
- `lerna publish` 会打tag，上传git,上传npm。
  - 需要在packages.json添加 "publishConfig": { "access": "public" },

配置 yarn + workspaces

```bash
# package.json 文件加入
"private": true,
"workspaces": [
  "packages/*"
],

# lerna.json 文件加入
"useWorkspaces": true,
"npmClient": "yarn",
```
