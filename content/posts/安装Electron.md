---
title: "安装Electron"
date: 2020-03-20T14:17:22+08:00
draft: false
tags:
  - 今天搜了啥
  - 开发环境
---


Electron 安装常见错误和解决方法。 

1. 自定义镜像 ELECTRON_MIRROR=https://npm.taobao.org/mirrors/electron/ npm --registry=https://registry.npm.taobao.org install electron@5.0.13
2. --unsafe-perm=true

<!--more-->
## 直接安装

npm install Electron 


错误信息:

```
npm ERR! code E404
npm ERR! 404 Not Found - GET https://registry.npmjs.org/Electron - Not found
npm ERR! 404 
npm ERR! 404  'Electron@latest' is not in the npm registry.
npm ERR! 404 Your package name is not valid, because 
npm ERR! 404  1. name can no longer contain capital letters
npm ERR! 404 
npm ERR! 404 Note that you can also install from a
npm ERR! 404 tarball, folder, http url, or git url.

npm ERR! A complete log of this run can be found in:
npm ERR!     xxxx-debug.log

```

## 添加镜像地址 (Custom Mirrors and Caches)

ELECTRON_MIRROR=https://npm.taobao.org/mirrors/electron/ npm --registry=https://registry.npm.taobao.org install electron@5.0.13

错误信息:

```

Error: tunneling socket could not be established, cause=Parse Error
xxxx/node_modules/electron/install.js:49
  throw err
  ^

Error: tunneling socket could not be established, cause=Parse Error
    at ClientRequest.onError (xxxx/node_modules/tunnel-agent/index.js:177:17)
    at Object.onceWrapper (events.js:286:20)
    at ClientRequest.emit (events.js:198:13)
    at Socket.socketOnData (_http_client.js:457:9)
    at Socket.emit (events.js:198:13)
    at addChunk (_stream_readable.js:288:12)
    at readableAddChunk (_stream_readable.js:269:11)
    at Socket.Readable.push (_stream_readable.js:224:10)
    at TCP.onStreamRead (internal/stream_base_commons.js:94:17)
npm WARN ajv-keywords@3.2.0 requires a peer of ajv@^6.0.0 but none is installed. You must install peer dependencies yourself.

npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! electron@5.0.13 postinstall: `node install.js`
npm ERR! Exit status 1
npm ERR! 
npm ERR! Failed at the electron@5.0.13 postinstall script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     xxxx/.npm/_logs/2020-03-20T06_00_26_686Z-debug.log

```

## unsafe-perm flag

ELECTRON_MIRROR=https://npm.taobao.org/mirrors/electron/ npm --registry=https://registry.npm.taobao.org install electron@5.0.13 --unsafe-perm=true --allow-root


安装成功!


## 参考

* https://github.com/electron/electron/blob/master/docs/tutorial/installation.md


