Mockery
=======

[![Build Status](https://travis-ci.org/mockery/mockery.svg?branch=master)](https://travis-ci.org/mockery/mockery)
[![Latest Stable Version](https://poser.pugx.org/mockery/mockery/v/stable.svg)](https://packagist.org/packages/mockery/mockery)
[![Coverage Status](https://coveralls.io/repos/github/mockery/mockery/badge.svg)](https://coveralls.io/github/mockery/mockery)
[![Total Downloads](https://poser.pugx.org/mockery/mockery/downloads.svg)](https://packagist.org/packages/mockery/mockery)

Mockery is a simple yet flexible PHP mock object framework for use in unit testing
with PHPUnit, PHPSpec or any other testing framework. Its core goal is to offer a
test double framework with a succinct API capable of clearly defining all possible
object operations and interactions using a human readable Domain Specific Language
(DSL). Designed as a drop in alternative to PHPUnit's phpunit-mock-objects library,
Mockery is easy to integrate with PHPUnit and can operate alongside
phpunit-mock-objects without the World ending.

Mockery 是一个灵活的mock PHP对象的框架，可以与PHPUnit、PHPSpec或者其他测试框架的单元测试结合使用。
它使用可以定义所有对象操作和交互的简洁API，来提供测试替身框架。作为 PHPUnit 的 phpunit-mock-objects 的可替换库，
Mockery 很同意与 PHPUnit 集成，并永久与 phpunit-mock-objects 库共同合作。


Mockery is released under a New BSD License.
Mockery 在 New BSD License 下发布。

## Installation
## 安装

To install Mockery, run the command below and you will get the latest
version
运行以下命令来安装 Mockery 最新版本

```sh
composer require --dev mockery/mockery
```

⚠️️ The remainder of this README refers specifically to the master branch (1.0-dev).
⚠️️ 该 README 的提醒特别指向 master 分支 (1.0-dev).

## Documentation
## 文档
旧版本中该 README 文件就是 Mockery 文档，后来做了改善，并创建了扩展文档。请将本文档当做学习 Mockery 的起始点，一定要再去阅读扩展文档学习怎么使用 Mockery。
In older versions, this README file was the documentation for Mockery. Over time
we have improved this, and have created an extensive documentation for you. Please
use this README file as a starting point for Mockery, but do read the documentation
to learn how to use Mockery.

The current version can be seen at [docs.mockery.io](http://docs.mockery.io).
当前文档版本托管在 [docs.mockery.io](http://docs.mockery.io).

## Test Doubles
## 测试替身
测试替身(也常称为模仿) 模拟真是对象行为，它们一般被用来 提供测试隔离，代表还不存在的对象，或者考虑还不需要实现的API接口。
Test doubles (often called mocks) simulate the behaviour of real objects. They are
commonly utilised to offer test isolation, to stand in for objects which do not
yet exist, or to allow for the exploratory design of class APIs without
requiring actual implementation up front.

测试替身框架的好处是可以自由生成和配置测试替身，允许设置被调用方法和(或)返回值 使用能够捕捉每一个真实对象行为的灵活API，
The benefits of a test double framework are to allow for the flexible generation
and configuration of test doubles. They allow the setting of expected method calls
and/or return values     using a flexible API which is capable of capturing every possible real object behaviour 
in way that is stated as close as possible to a natural language description. 尽可能使用自然语言描述捕捉每一个真实对象行为
使用 `Mockery::mock` 方法来创建测试替身。
Use the `Mockery::mock` method to create a test
double.

``` php
$double = Mockery::mock();
```

If you need Mockery to create a test double to satisfy a particular type hint,
you can pass the type to the `mock` method.
如果你需要 Mockery 来满足特别的类型提示，你可以把该类型传进 `mock` 方法中。

``` php
class Book {}

interface BookRepository {
    function find($id): Book;
    function findAll(): array;
    function add(Book $book): void;
}

$double = Mockery::mock(BookRepository::class);
```
文档 [Creating test doubles](http://docs.mockery.io/en/latest/reference/creating_test_doubles.html) 部分对如何创建和使用测试替身有详细的描述。
A detailed explanation of creating and working with test doubles is given in the
documentation, [Creating test doubles](http://docs.mockery.io/en/latest/reference/creating_test_doubles.html)
section.

## Method Stubs 🎫
## 方法桩件 🎫
方法桩件是 使得测试替身 针对某个被调用方法提供返回值 的一种机制，使用方法桩件，你不必关心该方法被调用多少次，桩件是作为被测试系统的间接输入。  
A method stub is a mechanism for having your test double return canned responses
to certain method calls. With stubs, you don't care how many times, if at all,
the method is called. Stubs are used to provide indirect input to the system
under test.

``` php
$double->allows()->find(123)->andReturns(new Book());

$book = $double->find(123);
```
如果你之前使用过 Mockery，你会对上面的示例有新发现 &mdash; 我们使用 `allows` 方法创建方法桩件，而不是旧版本的 `shouldReceive` 方法，这是 Mockery v1 版本的新功能，
不过不用担心，`shouldReceive` 方法仍然可以使用。

If you have used Mockery before, you might see something new in the example
above &mdash; we created a method stub using `allows`, instead of the "old"
`shouldReceive` syntax. This is a new feature of Mockery v1, but fear not,
the trusty ol' `shouldReceive` is still here.

对于 Mockery 新用户，上面的示例也可以写为：
For new users of Mockery, the above example can also be written as:

``` php
$double->shouldReceive('find')->with(123)->andReturn(new Book());
$book = $double->find(123);
```
如果方法桩件不需要参数，也可以使用快捷方法来一次安装多次调用：
If your stub doesn't require specific arguments, you can also use this shortcut
for setting up multiple calls at once:

``` php
$double->allows([
    "findAll" => [new Book(), new Book()],
]);
```

or
或者

``` php
$double->shouldReceive('findAll')
    ->andReturn([new Book(), new Book()]);
```

You can also use this shortcut, which creates a double and sets up some stubs in
one call:

``` php
$double = Mockery::mock(BookRepository::class, [
    "findAll" => [new Book(), new Book()],
]);
```

## Method Call Expectations 📲

A Method call expectation is a mechanism to allow you to verify that a
particular method has been called. You can specify the parameters and you can
also specify how many times you expect it to be called. Method call expectations
are used to verify indirect output of the system under test.

``` php
$book = new Book();

$double = Mockery::mock(BookRepository::class);
$double->expects()->add($book);
```

During the test, Mockery accept calls to the `add` method as prescribed.
After you have finished exercising the system under test, you need to
tell Mockery to check that the method was called as expected, using the
`Mockery::close` method. One way to do that is to add it to your `tearDown`
method in PHPUnit.

``` php

public function tearDown()
{
    Mockery::close();
}
```

The `expects()` method automatically sets up an expectation that the method call
(and matching parameters) is called **once and once only**. You can choose to change
this if you are expecting more calls.

``` php
$double->expects()->add($book)->twice();
```

If you have used Mockery before, you might see something new in the example
above &mdash; we created a method expectation using `expects`, instead of the "old"
`shouldReceive` syntax. This is a new feature of Mockery v1, but same as with
`accepts` in the previous section, it can be written in the "old" style.

For new users of Mockery, the above example can also be written as:

``` php
$double->shouldReceive('find')
    ->with(123)
    ->once()
    ->andReturn(new Book());
$book = $double->find(123);
```

A detailed explanation of declaring expectations on method calls, please
read the documentation, the [Expectation declarations](http://docs.mockery.io/en/latest/reference/expectations.html)
section. After that, you can also learn about the new `allows` and `expects` methods
in the [Alternative shouldReceive syntax](http://docs.mockery.io/en/latest/reference/alternative_should_receive_syntax.html)
section.

It is worth mentioning that one way of setting up expectations is no better or worse
than the other. Under the hood, `allows` and `expects` are doing the same thing as
`shouldReceive`, at times in "less words", and as such it comes to a personal preference
of the programmer which way to use.

## Test Spies 🕵️

By default, all test doubles created with the `Mockery::mock` method will only
accept calls that they have been configured to `allow` or `expect` (or in other words,
calls that they `shouldReceive`). Sometimes we don't necessarily care about all of the
calls that are going to be made to an object. To facilitate this, we can tell Mockery
to ignore any calls it has not been told to expect or allow. To do so, we can tell a
test double `shouldIgnoreMissing`, or we can create the double using the `Mocker::spy`
shortcut.

``` php
// $double = Mockery::mock()->shouldIgnoreMissing();
$double = Mockery::spy();

$double->foo(); // null
$double->bar(); // null
```

Further to this, sometimes we want to have the object accept any call during the test execution
and then verify the calls afterwards. For these purposes, we need our test
double to act as a Spy. All mockery test doubles record the calls that are made
to them for verification afterwards by default:

``` php
$double->baz(123);

$double->shouldHaveReceived()->baz(123); // null
$double->shouldHaveReceived()->baz(12345); // Uncaught Exception Mockery\Exception\InvalidCountException...
```

Please refer to the [Spies](http://docs.mockery.io/en/latest/reference/spies.html) section
of the documentation to learn more about the spies.

## Utilities 🔌

### Global Helpers

Mockery ships with a handful of global helper methods, you just need to ask
Mockery to declare them.

``` php
Mockery::globalHelpers();

$mock = mock(Some::class);
$spy = spy(Some::class);

$spy->shouldHaveReceived()
    ->foo(anyArgs());
```

All of the global helpers are wrapped in a `!function_exists` call to avoid
conflicts. So if you already have a global function called `spy`, Mockery will
silently skip the declaring it's own `spy` function.

### Testing Traits

As Mockery ships with code generation capabilities, it was trivial to add
functionality allowing users to create objects on the fly that use particular
traits. Any abstract methods defined by the trait will be created and can have
expectations or stubs configured like normal Test Doubles.

``` php
trait Foo {
    function foo() {
        return $this->doFoo();
    }

    abstract function doFoo();
}

$double = Mockery::mock(Foo::class);
$double->allows()->doFoo()->andReturns(123);
$double->foo(); // int(123)
```

## Versioning

The Mockery team attempts to adhere to [Semantic Versioning](http://semver.org),
however, some of Mockery's internals are considered private and will be open to
change at any time. Just because a class isn't final, or a method isn't marked
private, does not mean it constitutes part of the API we guarantee under the
versioning scheme.

### Alternative Runtimes

Mockery will attempt to continue support HHVM, but will not make any guarantees.

## A new home for Mockery

⚠️️ Update your remotes! Mockery has transferred to a new location. While it was once
at `padraic/mockery`, it is now at `mockery/mockery`. While your
existing repositories will redirect transparently for any operations, take some
time to transition to the new URL.
```sh
$ git remote set-url upstream https://github.com/mockery/mockery.git
```
Replace `upstream` with the name of the remote you use locally; `upstream` is commonly
used but you may be using something else. Run `git remote -v` to see what you're actually
using.
