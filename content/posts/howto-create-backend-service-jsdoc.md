+++
title = "构建完整web服务 - jsdoc"
date = 2020-01-25T02:53:21+08:00
draft = false
tags = []
categories = []
+++

通过jsdoc实现node的日志生成。

<!--more-->

js是弱类型，就一定需要用文档，日志，单元测试来约束，弱类型写起来一时爽，但后期很容易崩，写好日志是必不可少的一环。

`yarn add jsdoc docdash`

docdash是一个jsdoc的template, 能够生成更加友好的style。

然后配上jsdoc.conf

```js
{
  "tags": {
    "allowUnknownTags": true
  },
  "source": {
    "include": ["./src"],
    "includePattern": ".js$",
    "excludePattern": "(node_modules/|docs)"
  },
  "plugins": ["plugins/markdown"],
  "opts": {
    "recurse": true,
    "verbose": true,
    "encoding": "utf8",
    "template": "node_modules/docdash",
    "destination": "./jsdoc/",
    "readme": "README.md"
  },
  "templates": {
    "cleverLinks": false,
    "monospaceLinks": false
  }
}
```

最后在package.json中加上,

```js
"scripts": {
  ...
  "jsdoc": "jsdoc -c jsdoc.json"
}
```

基本就ok了。

然后来看怎么写注释:

```js
/** 
 * 这是一个Class的注释
 */
class App {
  /**
   * 这是一个方法的注释
   */
   open() {
     ...
   }
}
```

然后关键词参考官网就行了, 移步 [Use JSDoc: Index](https://jsdoc.app/index.html)