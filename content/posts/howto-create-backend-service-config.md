+++
title = "构建完整web服务 - Config"
date = 2020-01-01T22:43:21+08:00
draft = false
tags = []
categories = []
+++

通过config和dotenv实现在不同模式下的配置。

<!--more-->

配置依赖于当前的NODE_ENV, 程序应将默认值设置为development。
我们一般把常规配置项放在config内，在env中放入私密的账号信息以及相关的环境变量。

```
- config
  - default.yaml
  - development.yaml
  - production.yaml
  - test.yaml
- src
- test
- .env
- .env.production
- .env.test
```

config/default.yaml
```
msg: 'default yaml'
```

config/test.yaml
```
msg: 'test yaml'
```

.env
```
DEBUG=app*
DB_USER=root
DB_PASS=000000
```

.env.test
```
DEBUG=app*
DB_USER=test
DB_PASS=123456
```

app.js
```js
require('dotenv').config();
const config = require('config');
const debug = require('debug')('app');

if (!process.env.NODE_ENV) {
  process.env.NODE_ENV = 'development';
}

debug(process.env.NODE_ENV);
debug(config.get('msg'));

debug(process.env.DB_USER);
debug(process.env.DB_PASS);
```

test.js
```js
require('dotenv').config({ path: '.env.test' });
const config = require('config');
const debug = require('debug')('app');

debug(process.env.NODE_ENV);
debug(config.get('msg'));

debug(process.env.DB_USER);
debug(process.env.DB_PASS);
```

config是通过NODE_ENV加载不同的配置，
- 首先加载__dirname/config/default.yaml
- 然后__dirname/config/NODE_ENV.yaml覆盖

dotenv略有不同,
- 如果没有指定path, 则加载.env
- 或者手动指定path, 但不会默认加载.env

最后, .env会涉及隐私, 所以需要在.gitignore中排除.env*。