# Webpack External与npm link不兼容的问题

Webpack External会在运行时引入指定的包，最终的表现形式类似于`const pkg_name = require("pkg_name")`。

这个问题是在使用npm link调试一个使用webpack构建的包时发现的，当我在另一个包里调用使用了Webpack External构建出来的产物时，程序最终弹出了一个无法索引到相关包的错误，我怀疑这是因为Nodejs无法基于两个文件的实际位置索引到同一个node_modules目录从而引发了这个问题，于是就创建了一个Demo测试了一下，发现这个问题确实存在。

# 如何复现

这个Demo算是一个非常简化的问题复现代码，跟随下面的步骤即可复现这个问题：

1. 在package-2目录下执行命令`npm i`以安装依赖。

2. 在package-1目录下执行命令`npm link`将`@test/package-1`添加到`npm link`注册表中。

3. 在package-2目录下执行命令`npm link @test/package-1`将package-1使用符号链接链接到`node_modules`目录下。

4. 使用node命令运行package-2的index.js文件。

如果一切顺利应该会得到这样一个错误：

```
node:internal/modules/cjs/loader:998
  throw err;
  ^

Error: Cannot find module 'react'
Require stack:
- /root/workspace/external-link/package-1/index.js
- /root/workspace/external-link/package-2/index.js
    at Function.Module._resolveFilename (node:internal/modules/cjs/loader:995:15)
    at Function.Module._load (node:internal/modules/cjs/loader:841:27)
    at Module.require (node:internal/modules/cjs/loader:1067:19)
    at require (node:internal/modules/cjs/helpers:103:18)
    at Object.<anonymous> (/root/workspace/external-link/package-1/index.js:1:15)
    at Module._compile (node:internal/modules/cjs/loader:1165:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1219:10)
    at Module.load (node:internal/modules/cjs/loader:1043:32)
    at Function.Module._load (node:internal/modules/cjs/loader:878:12)
    at Module.require (node:internal/modules/cjs/loader:1067:19) {
  code: 'MODULE_NOT_FOUND',
  requireStack: [
    '/root/workspace/external-link/package-1/index.js',
    '/root/workspace/external-link/package-2/index.js'
  ]
}
```
