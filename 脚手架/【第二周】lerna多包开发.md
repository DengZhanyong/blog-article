### 多包管理

开发一个好的脚手架，需要让它具有更好的灵活性和扩展性。最重要的方式就是拆分，将一个脚手架分模块，分功能的进行拆分成多个 `npm` 包，他们之间通过相互引用的方式来完成整个脚手架的功能。

单包管理的痛点：每次更新都必须要更新版本，难维护。

多包管理方式的最大的好处就是**易扩展与维护**，无论是增加新功能，还是优化修复以前的功能，基本上不需要更新脚手架的主包版本。



### lerna是什么

> Lerna 是一个管理工具，用于管理包含多个软件包（package）的 JavaScript 项目
>
> Lerna 是一种工具，针对 使用 git 和 npm 管理多软件包代码仓库的工作流程进行优化

将大型代码仓库分割成多个独立版本化的 软件包（package）对于代码共享来说非常有用。但是，如果某些更改跨越了多个代码仓库的话将变得很 *麻烦* 并且难以跟踪，并且， 跨越多个代码仓库的测试将迅速变得非常复杂。

为了解决这些（以及许多其它）问题，某些项目会将 代码仓库分割成多个软件包（package），并将每个软件包存放到独立的代码仓库中。但是，例如 Babel、 React、Angular、Ember、Meteor、Jest 等项目以及许多其他项目则是在 一个代码仓库中包含了多个软件包（package）并进行开发。

**有哪些好处：**

- 可以让我们在一个项目中开发多个包，只需要一个仓库
- 每个包都可以同时独立的被发布
- 发布前会自动帮我们将代码提交到远程
- 每次发布只会发布有变动的包
- 自动帮我们升级包的版本号，可灵活选择版本号的升级方式



### lerna的基础用法

#### 初始化项目

```shell
# 创建一个文件夹
mkdir steamed-cli

# 初始化npm
npm init --yes

# 初始化lerna配置
lerna init

# 创建packages
lerna create utils  

# 包名建议加上组织  @steamed/utils
lerna create @steamed/utils
```

**初始化后的目录结构**

```shell
F:
│ lerna.json
│ package.json
│
└─packages
```

- `lerna.json` ：`lerna` 的配置文件 
- `packages`：所有的包都会存放在这个目录下

_package.json_

```json
{
  "packages": [
    "packages/*"
  ],
  "version": "1.0.0"
}
```

**什么是组织？**

例如 `@vue/cli`  中的 `vue` 就是一个组织，带有组织的包名的形式为：`@组织名/名称`

**组织的好处**

- 避免包名重复：例如我想建一个工具包取名为 `utils` ，很显然这个包在 `npm` 上早已存在
- 分类作用：将组织名取名为脚手架的名称，这个组织下所有的包都是属于这个脚手架的内容，方便管理

**如何创建组织**

1. 需要注册 `npm` 账号（[https://www.npmjs.com](https://www.npmjs.com/)）

2. 登录后选择 `+Add Organization`

   ![](https://resource.dengzhanyong.com/images/1639559078567.jpg)

3. 填写组织名称，选择创建一个免费的组织

   ![](https://resource.dengzhanyong.com/images/1639559231412.jpg)

**如何将包上传到该组织下**

在创建包的时候，包名需要是 `@组织名/名称` 的形式，在 `publish` 后，会自动上传到对应的组织下，如果组织不存在，则会上传失败。

#### 安装依赖

```shell
# 对所有的packages安装依赖
lerna add echarts

# 对指定的目录安装依赖
lerna add echarts packages/core/

# 安装/重装依赖
lerna bootstrap

# 创建软连接
lerna link
```



#### 其他常用命令

```shell
# 在所有的packages下执行shell命令
lerna exec -- rm -rf node_modules

# 删除指定包下面的node_modules
lerna exec --scope @imooc-cli-dev/utils -- rm -rf node_modules

# 执行 npm  script 命令
lerna run start
lerna run --scope @imooc-cli-dev/utils start

# 查看有哪些package有变更
lerna changed

# 查看变更
lerna diff 

# 提交
lerna publish
```


