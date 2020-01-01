+++
title = "构建完整web服务 - Schema"
date = 2020-01-01T01:05:40+08:00
draft = true
tags = []
categories = []
+++

我们都知道, javascript是弱类型的, 常规来说可以通过typescript或者flow来解决类型问题, 不过这里还是保留javascript原生态, 通过json schema来做类型检查。

<!--more-->

> JSON Schema is a powerful tool for validating the structure of JSON data. 

最简单的schema是`{}`, 所以任何对象都可以通过这个schema的验证, 这里用[ajv](https://ajv.js.org)来验证一下,

```js
require('dotenv').config();
const debug = require('debug')('app');
const ajv = new require('ajv')();

let schema = {};

debug(ajv.validate(schema, "hello world"));
```
=> true

加上type,
```js
let schema = {
  type: 'number'
};

debug(ajv.validate(schema, "hello world"));
```
=> false

这就是最简单的schema验证方法。
来看一个一般一点的schema, 另外补充一句, schema可以同时服务后端和前端,

```js
it('simple object', () => {
  let schema = {
    $schema: "http://json-schema.org/draft-07/schema#",
    $id: "http://space365.live/schemas/user.json",
    title: 'user-form',
    type: 'object',
    additionalProperties: false,
    required: ['user', 'pass', 'userType'],
    properties: {
      user: { type: 'string' },
      pass: { type: 'string' },
      userType: ['guest', 'normal', 'admin']
    }
  };

  let data = {
    user: 'nonocast',
    pass: '123456',
    userType: 'admin'
  }

  ajv.validate(data).should.eq(true);
});
```

- $schema 不是必填项, 但官网在real world中始终建议添加, 标识schema的版本号
- $id 不是必填项, 通常用url来标识schema
- additionalProperties 为false时表示出现properties外的属性时会验证失败
- type 必须是"integer", "string", "number", "object", "array", "boolean", "null"中的一个, 或者是一个array, 表示枚举 (所以这里就不建议对象中有type属性)
- 其中number包含integer和number, 区别是integer只能约束整数

```js
it('integer and number', () => {
  let numberSchema = { type: 'number' };
  let integerSchema = { type: 'integer' };

  ajv.validate(numberSchema, 1).should.eq(true);
  ajv.validate(integerSchema, 1).should.eq(true);
  ajv.validate(numberSchema, 1.5).should.eq(true);
  ajv.validate(integerSchema, 1.5).should.eq(false);
});
```

来看属性验证,
```js
it('property formats', () => {
  let schema = {
    type: 'object',
    additionalProperties: false,
    properties: {
      email: { type: 'string', format: 'email' },
      address: { type: 'string', format: 'ipv4' },
      host: { type: 'string', format: 'hostname' },
      homepage: { type: 'string', format: 'uri' },
      date: { type: 'string', format: 'date-time' },
      timestamp: { type: 'number', minimum: 0, maximum: 2461449600 },
      mobile: { type: 'string', pattern: "^[1]([3-9])[0-9]{9}$" }
    }
  };

  let data = {
    email: 'nonocast@gmail.com',
    address: '192.168.1.1',
    host: 'nonocast.cn',
    homepage: 'https://nonocast.cn',
    date: '2018-11-13T20:20:39+00:00',
    timestamp: 1577818716,
    mobile: '13817100000',
  };

  ajv.validate(schema, data).should.eq(true);
});
```

当需要复用一个定义的时候就需要用到$ref, 
```js
it('reuse', () => {
  let schema = {
    $schema: "http://json-schema.org/draft-07/schema#",
    definitions: {
      user: {
        type: 'object',
        properties: {
          name: { type: 'string' },
          email: { type: 'string', format: 'email' },
          gender: { type: 'boolean' }
        },
        required: ['name', 'email']
      },
      timestamp: {
        type: 'integer',
        minimum: 0,
        maximum: 2461449600
      }
    },

    type: "object",
    properties: {
      owner: { $ref: '#/definitions/user' },
      organizer: { $ref: '#/definitions/user' },
      begin: { $ref: '#/definitions/timestamp' },
      end: { $ref: '#/definitions/timestamp' },
    },
    allOf: [
      {
        not: {
          properties: {
            owner: { type: 'null' }
          }
        }
      },
      {
        not: {
          properties: {
            organizer: { type: 'null' }
          }
        }
      }
    ],
    required: ['owner']
  };

  ajv.validate(schema, { 
    owner: null, 
    organizer: null 
  }).should.eq(false);

  ajv.validate(schema, { 
    owner: { 
      name: 'x', 
      email: 'x@x.com' 
    }, 
    organizer: null }
  ).should.eq(false);

  ajv.validate(schema, { 
    owner: null, 
    organizer: { 
      name: 'x', 
      email: 'x@x.com' 
    } 
  }).should.eq(false);

  let data = {
    owner: {
      name: 'nonocast',
      email: 'nonocast@gmail.com'
    },
    organizer: {
      name: 'icy',
      email: 'naodaixiaoxiao@qq.com'
    },
    begin: moment().unix(),
    end: moment().unix()
  };

  ajv.validate(schema, data).should.eq(true);
});
```

这里也同时用到了allOf和not的组合，用来做对象的约束，可以用到的还有anyOf, oneOf, 参考: [Combining schemas — Understanding JSON Schema 7.0 documentation](https://json-schema.org/understanding-json-schema/reference/combining.html)

$ref中的'#'称为json pointer,

> The value of $ref is a URI, and the part after # sign (the “fragment” or “named anchor”) is in a format called JSON Pointer.

- "#/definitions/address" 当前文档
- "definitions.json#/address" 相对路径
- "http://xxx.com/definitions.json" 远程路径

#ref也可以通过$id进行定位。

```js
it('reuse by id', () => {
  let schema = {
    $schema: "http://json-schema.org/draft-07/schema#",
    definitions: {
      user: {
        $id: '#user',
        type: 'object',
        properties: {
          name: { type: 'string' },
          email: { type: 'string', format: 'email' },
          gender: { type: 'boolean' }
        },
        required: ['name', 'email']
      },
    },

    type: "object",
    properties: {
      owner: { $ref: '#user' },
      organizer: { $ref: '#/definitions/user' },
    },
    required: ['owner']
  };

  let data = {
    owner: {
      name: 'nonocast',
      email: 'nonocast@gmail.com'
    }
  };

  ajv.validate(schema, data).should.eq(true);
});
```