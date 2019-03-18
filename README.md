# learn-lerna

[![lerna](https://img.shields.io/badge/maintained%20with-lerna-cc00ff.svg)](https://lernajs.io/)

学习 lerna 的使用。最好的方式，就是亲手操作一遍

参考：

- [monorepo 新浪潮](https://juejin.im/entry/586f00bc128fe100580a6f78)
  - [用lerna-changelog 来梳理 changelog](https://github.com/lerna/lerna-changelog)
  - [使用 monorepo 结构，管理多个 repo(示例)](https://github.com/galaxybing/lerna-repos-init.git)
- [lerna管理前端packages的最佳实践](https://juejin.im/post/5a989fb451882555731b88c2)
- [lerna 中文文档](https://github.com/chinanf-boy/lerna-zh)
  - [常见问题](https://github.com/chinanf-boy/lerna-zh/blob/master/FAQ.zh.md)
- [https://lernajs.io/](https://lernajs.io/)
  - [lerna-wizard lerna的命令行向导](https://github.com/szarouski/lerna-wizard)

**管理多个 repo ：**

- 单个 lint、build、test 和 release 流程
- 统一的地方处理 issue
- 不用到处去找自己项目的 repo
- 方便管理版本和 dependencies
- 跨项目的操作和修改变得容易
- 方便生成总的 changelog

## 安装

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

# 更新版本(不用主动去更新，直接执行发布，根据提示选择操作即可)
# 使用与 npm version 相同的语法，更新版本号，如
lerna version patch

# Changes:
# - @xmini/package-1: 0.0.1 => 0.0.2
# - @xmini/package-2: 0.0.1 => 0.0.2

# 发布到 npm
# publish将帮助发布任何更新的包（如果包未更新，会忽略）
lerna publish
```
