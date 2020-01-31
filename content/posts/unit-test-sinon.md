+++
title = "Unit Test: Sinon"
date = 2020-01-31T16:57:00+08:00
draft = false
tags = []
categories = []
+++

Standalone test spies, stubs and mocks for JavaScript. 
Works with any unit testing framework.

<!--more-->

```js
require("chai").should();
const assert = require("chai").assert;
const debug = require("debug")("test");
const sinon = require("sinon");
const mongoose = require("mongoose");
require("sinon-mongoose");
const AppContext = require('../../src/AppContext');

describe.skip("hello sinon", () => {
  it("fake call", () => {
    let fake = sinon.fake();
    fake();
    fake.called.should.eq(true);
    fake.callCount.should.eq(1);
  });

  it("fake", () => {
    var fake = sinon.fake.returns("foo");
    let result = fake();
    result.should.eq("foo");
  });

  it("replace", () => {
    var fake = sinon.fake.returns("42");
    sinon.replace(console, "dir", fake);
    console.dir("apple pie").should.be.eq("42");
  });

  it("spy function", () => {
    let foo = () => "foo";
    let spy = sinon.spy(foo);
    spy();
    spy.called.should.eq(true);
  });

  it("spy object", () => {
    let foo = {
      bar: () => "bar"
    };

    let spy = sinon.spy(foo, "bar");
    foo.bar();
    spy.called.should.eq(true);
  });

  it("stub", () => {
    let stub = sinon.stub();
    stub("hello");
    stub.firstCall.args[0].should.eq("hello");
  });

  it("mock model", async () => {
    let expects = [{ name: "foo" }, { name: "bar" }];

    let Company = mongoose.model("Company");
    sinon.mock(Company).expects("find").resolves(expects);
    let result = await Company.find({});
    result.length.should.eq(2);
    result[0].name.should.eq("foo");
    result[1].name.should.eq("bar");
  });

  it("stub function", async () => {
    let hello = {
      foo: () => "foo",
      bar: (arg) => arg
    };

    sinon.stub(hello, 'foo').returns("MOCK-VALUE");
    sinon.stub(hello, 'bar').withArgs("kk").returns(">>>");

    debug(hello.foo());
    debug(hello.bar('kk'));
    debug(hello.bar('xyz'));
  });

  // resolves: 返回Promise
  // returns: 直接返回value
  it("mock args", async () => {
    let hello = {
      foo: () => "foo",
      bar: (arg) => arg
    };

    let mock = sinon.mock(hello);
    mock.expects("foo").returns("mock-foo");
    mock.expects("bar").withArgs("hello").returns("HELLO");

    debug(hello.foo());
    debug(hello.bar('hello'));
    debug(hello.bar('world'));
  });

  afterEach(() => {
    sinon.restore();
  });
});
```