# learn-lerna

[![lerna](https://img.shields.io/badge/maintained%20with-lerna-cc00ff.svg)](https://lernajs.io/)

学习 lerna 的使用。最好的方式，就是亲手操作一遍

learn 差点死掉，之间我切换 [turborepo](https://turbo.build/) 玩了一阵, lerna 被 nx 收购，起死回生了，并且有个较大提升，同时 pnpm 支持也还好。

## 项目初始化

monorepo

```bash
npx lerna init
```

- [使用 pnpm](https://lerna.js.org/docs/recipes/using-pnpm-with-lerna)

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

执行 script task，可参考文档 [Run Tasks](https://lerna.js.org/docs/features/run-tasks)以及[示例库](https://github.com/lerna/getting-started-example)

```bash
# umi4 为对应的 package name
"demo": "lerna run dev --scope=umi4",
"dev": "lerna run dev --ignore=umi4",

# 直接运行库和应用
"dev": "lerna run dev",
```

构建依赖是自动识别的，无需 nx.json 就能完成自动构建依赖的管理

> NOTE: 不要这样 `npm run dev && npm run demo`
> 上面的命令开发运行时，包的构建命令不会退出，所以就没有 umi4 构建的启动

lerna 默认并行构建 5 个进程，构建(build) 时没问题，但开发时就不行了，因为每个进程都不退出，后续的进程无法启动，开发态没有项目完整性，导致问题

此时可以通过 `--concurrency=10` 参数增加并行任务数（依赖 nx-cloud?）

- 1. 并不是一定依赖 `@nrwl/nx-cloud` 需要使用云端配置
- 2. 下方的 nx.json 同样支持并行任务

> 目前并行任务多，有点卡，还需要继续测试验证。

```js
// nx.json
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
  "targetDependencies": {
    "build": [
      {
        "target": "build",
        "projects": "dependencies"
      }
    ]
  }
}
```

示例参考

- [lerna-monorepo-example](https://github.com/cloudyan/lerna-monorepo-example)
  - 分支
    - main(使用 npm)
    - pnpm
- [lowcode-monorepo](https://github.com/cloudyan/lowcode-monorepo)

这里参考文档时，通过 `npx lerna add-caching` 操作添加的 nx.json 会 targetDefaults 配置，该配置会影响 concurrency，并且新增需要配置 `"extends": "nx/presets/npm.json",`

```js
// nx.json
"targetDefaults": {
  "dev": {
    "dependsOn": [
      "^dev"
    ]
  },
  "build": {
    "dependsOn": ["^build"],
    "outputs": ["{projectRoot}/dist"]
  }
}
```

其他的一些配置参考

- https://github.com/lerna/getting-started-example/blob/main/nx.json
- https://github.com/vsavkin/lerna-dte/blob/main/nx.json

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
      "showUsageWarnings": true
    }
  }
},
```


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
