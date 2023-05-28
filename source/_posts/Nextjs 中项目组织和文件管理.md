---
title:  Next.js 中项目组织和文件管理
date: 2023-05-28
---

前端工程中如何组织和管理文件是一个非常不好处理的问题，那么我们来看看 nextjs  如何组织和管理文件。

## 路由组织

在 `app` 目录中，嵌套的文件夹层次结构定义了路由结构。

每个文件夹代表一个路由段，映射到 UR L路径中的相应段。

然而，即使路由结构是通过文件夹定义的，路由也不能被公开访问，除非将 `page.js` 或 `route.js` 文件添加到路由段中。

![](https://zzh-cat.oss-cn-beijing.aliyuncs.com/assets/2023-05-28-01.avif)

而且，即使路由是公共可访问的，也只有 `page.js` 或 `route.js` 返回的内容才会被发送到客户端。

![](https://zzh-cat.oss-cn-beijing.aliyuncs.com/assets/2023-05-28-02.avif)

这意味着项目文件可以安全地放在 `app` 目录的路由段中，而不会意外地导致可路由。

![](https://zzh-cat.oss-cn-beijing.aliyuncs.com/assets/2023-05-28-03.avif)

## 项目组织特性

Next.js 提供了多个功能来帮助您组织项目。

### 私有文件夹

可以通过在文件夹前加上下划线: `_folderName` 来创建私有文件夹。

这表明该文件夹是一个私有的实现细节，不应该被路由系统考虑，将文件夹及其所有子文件夹从路由中选择出来。

![](https://zzh-cat.oss-cn-beijing.aliyuncs.com/assets/2023-05-28-04.avif)

因为默认情况下 `app` 目录下的文件可以安全地并置，所以不需要私有文件夹来进行并置。然而，它们可以用于:

- 将UI逻辑与路由逻辑分离。
- 在项目和 Next.js 生态系统中持续组织内部文件。
- 在代码编辑器中对文件进行排序和分组。
- 避免潜在的命名冲突与未来的 Next.js 文件约定。

### 路由组

在 `app` 目录中，嵌套文件夹通常被映射到 UR路径。但是，您可以使用路由组打破这种行为。

路由组可以通过在括号 `(folderName)` 中包装一个文件夹来创建。

这表明该文件夹是用于组织目的的，不应该包含在路由的 URL 路径中。

![](https://zzh-cat.oss-cn-beijing.aliyuncs.com/assets/2023-05-28-05.avif)

路由组在以下情况下很有用:

- [将路线组织成组](https://nextjs.org/docs/app/building-your-application/routing/route-groups#organize-routes-without-affecting-the-url-path)，例如按站点部分、目的或团队。
- 在同一路由段级别启用嵌套布局:
    - [在同一段中创建多个嵌套布局，包括多个根布局](https://nextjs.org/docs/app/building-your-application/routing/route-groups#creating-multiple-root-layouts)
    - [在公共段的路由子集中添加一个布局](https://nextjs.org/docs/app/building-your-application/routing/route-groups#opting-specific-segments-into-a-layout)

### `src` 目录

Next.js 支持在 `src` 目录中存储应用程序代码(包括 `app`)。这将应用程序代码从项目配置文件中分离出来，这些文件大多位于项目的根目录中。

![](https://zzh-cat.oss-cn-beijing.aliyuncs.com/assets/2023-05-28-06.avif)

### 模块路径别名

Next.js 支持模块路径别名，这使得在深度嵌套的项目文件中更容易读取和维护导入。

```js
// before
import { Button } from '../../../components/button';
 
// after
import { Button } from '@/components/button';
```

## 项目组织策略

在 Next.js 项目中组织自己的文件和文件夹时，没有“正确”或“错误”的方式。

以下部分列出了对常用策略的非常高级的概述。最简单的收获是选择一个适合你和你的团队的策略，并在整个项目中保持一致。

> 注意:在下面的例子中，我们使用组件和库文件夹作为通用占位符，它们的命名没有特殊的框架意义，你的项目可能会使用其他文件夹，如ui, utils, hooks, styles等。

### 将项目文件保存在 `app` 之外

该策略将所有应用程序代码保存在项目根目录下的共享文件夹中，并保留 `app` 目录纯粹用于路由目的。

![](https://zzh-cat.oss-cn-beijing.aliyuncs.com/assets/2023-05-28-07.avif)

### 将所有项目文件保存在 `app` 中的文件夹中

该策略将所有应用程序代码保存在 `app` 目录根目录下的共享文件夹中。

![](https://zzh-cat.oss-cn-beijing.aliyuncs.com/assets/2023-05-28-08.avif)

### 按特性或路由拆分项目文件

该策略将全局共享的应用代码保存在应用根目录中，并将更具体的应用代码拆分到使用它们的路由段中。

![](https://zzh-cat.oss-cn-beijing.aliyuncs.com/assets/2023-05-28-09.avif)


来源: [Project Organization and File Colocation](https://nextjs.org/docs/app/building-your-application/routing/colocation#keep-project-files-outside-of-app)
