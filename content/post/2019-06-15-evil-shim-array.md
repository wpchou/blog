+++
date = "2019-06-15T00:00:00Z"
tags = ["javascript"]
categories = ["javascript"]
title = "从 shim-array 说到 javascript 的一些问题"

+++

我在两个旧的 javascript 项目都遇到一个很让人摸不着头脑的问题，使用数组的 `find` 无论能不能找到，总是返回 `-1`. 影响很大，不只自己的代码不能用，像 `sequelize` 这样的第三方包也用不了。有一段时间只能放弃，后来花了很长时间终于搞明白了是一个叫 `collections` 的依赖包使用所谓的 `shim-array` 覆盖修改了 `Array.prototype.find`, 使得所有的数组的行为都发生了改变。因为以前从来没有遇到过这种修改全局对象的情况，所以都没有起疑过是这里出了问题。

项目需要导出 excel 表格，于是就用了 `excel-export` 包，`excel-export` 又依赖于 `collections`. 这都是相当老的项目了，至少在 es6 标准出来之前就存在的。两个包都是将近废弃的状态。但是现在仍有每周几万的下载量。事情的缘起在于在 es6 之前内置库的功能比较弱，所以 `collections` 就用直接修改 `prototype` 方法增强内置对象，加了个 `find` 方法，没想到 es6 自个儿给加上了。

虽然理论上可以这么做，但是第一次碰到有人真在代码实践中这么做真是大跌眼镜。这真可以算是 javascript 天生的缺陷了。`修改内置全局对象`这样的事情都有人做。这样的包还有人用。有 `lodash` 这样的包在，还会有人用这种三角猫的包。也许 javascript 标准应该在这上面做些什么。

看这两个包的下载量，就可以看到多蠢的包都会有很多人在用。有直接的，也有间接的。`npm 包质量参差不齐`一直都是个问题，依赖层级又都很深，使用第三方包一定要慎之又慎。

代码三五年不更新，在有些语言像 C 语言来看根本就不是事。但在 javascript 这儿，却让代码用不上一些重要的包。`代码腐化的速度实在是太快了`。
