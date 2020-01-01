+++
title = "构建完整web服务 - Logging"
date = 2020-01-01T20:11:42+08:00
draft = false 
tags = []
categories = []
+++

我们在nodejs中采用winston记录日志, 然后通过ELK进行查询和分析。

<!--more-->

winston的基本用法参考[winston logger](https://github.com/nonocast/issues/issues/64)

Logger.js

- 打印文件名和行号
- 时间采用unix timestamp
- 支持console, 文本输出
- 支持rotate-file, json输出
- 支持查询

```js
const config = require('config');
const moment = require('moment');
const path = require('path');
const util = require('util');
const stackTrace = require('stack-trace');
const strip = require('strip-color');
const { createLogger, format, transports } = require('winston');
require('winston-daily-rotate-file');

class Logger {
  static get instance() {
    if (!this._instance) {
      this._instance = new Logger();
    }
    return this._instance;
  }

  constructor() {
    if (Logger._instance) {
      return Logger._instance;
    }

    this.init();
  }

  child(options) {
    let child = this.logger.child(options);

    let that = this;
    let log = child.log;

    let debug = child.debug;
    child.debug = function () {
      debug.call(this, ...arguments, that.getCallerInfo(2));
    }

    let info = child.info;
    child.info = function () {
      info.call(this, ...arguments, that.getCallerInfo(2));
    }

    let warn = child.warn;
    child.warn = function () {
      warn.call(this, ...arguments, that.getCallerInfo(2));
    }

    let error = child.error;
    child.error = function () {
      error.call(this, ...arguments, that.getCallerInfo(2));
    }

    return child;
  }

  getLabel(callingModule) {
    const parts = callingModule.filename.split(path.sep);
    return path.join(parts[parts.length - 2], parts.pop());
  }

  init() {
    this.logger = createLogger({
      level: 'debug',
      format: format.combine(
        format.errors({ stack: true }),
        format.splat(),
        format.prettyPrint(),
        format.timestamp({ format: () => moment().unix() }),
        format.label(`[${config.get('app')}]`),
        format.errors({ stack: true })
      ),
      transports: [
        new (transports.DailyRotateFile)({
          level: 'debug',
          json: true,
          filename: 'logs/app-%DATE%.log',
          datePattern: 'YYYY-MM-DD-HH',
          zippedArchive: false,
          maxSize: '20m',
          maxFiles: '14d',
          format: format.combine(
            format(info => {
              info.message = strip(info.message);
              return info;
            })(),
            format.json()
          )
        })
      ]
    });

    if (process.env.NODE_ENV !== 'production') {
      this.logger.add(new transports.Console({
        level: 'debug',
        format: format.combine(
          format.colorize(),
          format.printf(info => {
            let content = (info.requestId ? `[req_id: ${info.requestId}]` : '') + `${moment.unix(info.timestamp).format("HH:mm")} ${info.level} [${info.filename}:${info.line}]: ${info.message}`;
            let base = path.dirname(path.dirname(require.main.filename)) + '/';
            if (info.stack) {
              content += '\n' + info.stack.split(base).join("");
            }
            return content;
          })
        )
      }));
    }
  }

  debug(...args) {
    this.logger.debug(...args, this.getCallerInfo(2));
  }

  info(...args) {
    this.logger.info(...args, this.getCallerInfo(2));
  }

  warn(...args) {
    this.logger.warn(...args, this.getCallerInfo(2));
  }

  error(...args) {
    this.logger.error(...args, this.getCallerInfo(2));
  }

  getCallerInfo(level) {
    let p = stackTrace.get()[level];
    let filename = p.getFileName();
    let base = path.dirname(require.main.filename);
    filename = filename.replace(base, '').slice(1);
    let line = p.getLineNumber();
    return { filename, line }
  }

  async query() {
    const options = {
      from: moment().subtract(1, 'hours').unix(),
      until: moment().unix(),
      limit: 10000,
      start: 0,
    };

    return new Promise((resolve, reject) => {
      this.logger.query(options, function (err, results) {
        if (err) {
          reject();
        }
        resolve(results.dailyRotateFile);
      });
    });
  }

  getWinstonLogger() {
    return this.logger;
  }
}

module.exports = Logger;
```

最终形成的日志文件:
```
{"filename":"server/WebServer.js","line":65,"level":"info","message":"Server started, please visit: http://:9000 (with development mode)","timestamp":1576831366}
{"filename":"server/WebServer.js","line":68,"level":"info","message":"WebSocket server started, visit: ws://:9000","timestamp":1576831366}
{"requestId":"9a852397-a816-48a1-b59e-fa8058c90d14","filename":"server/middleware/koa-winston.js","line":9,"level":"info","message":"<-- GET /","timestamp":1576831371}
{"requestId":"9a852397-a816-48a1-b59e-fa8058c90d14","filename":"server/middleware/koa-winston.js","line":13,"level":"info","message":"--> GET / 200","timestamp":1576831371}
{"requestId":"94a3d36b-02f4-4725-8cee-0659ddbad446","filename":"server/middleware/koa-winston.js","line":9,"level":"info","message":"<-- GET /","timestamp":1576831372}
{"requestId":"94a3d36b-02f4-4725-8cee-0659ddbad446","filename":"server/middleware/koa-winston.js","line":13,"level":"info","message":"--> GET / 200","timestamp":1576831372}
{"requestId":"d3ea05e4-3cc2-4ee1-b39b-56acfa29ed91","filename":"server/middleware/koa-winston.js","line":9,"level":"info","message":"<-- GET /mqtt","timestamp":1576831373}
```