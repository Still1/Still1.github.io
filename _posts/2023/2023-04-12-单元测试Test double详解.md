---
title: 单元测试Test double详解
tags: [单元测试, 概念原理]
---

## Test double

> In computer programming and computer science, programmers employ a technique called automated unit testing to reduce the likelihood of bugs occurring in the software. Frequently, the final release software consists of a complex set of objects or procedures interacting together to create the final result. In automated unit testing, it may be necessary to use objects or procedures that look and behave like their release-intended counterparts, but are actually simplified versions that reduce the complexity and facilitate testing. A test double is a generic (meta) term used for these objects or procedures.

在计算机编程或计算机科学中，程序员使用一种名叫自动化单元测试的技术，以减少软件中出现bug的可能性。在绝大部分的情景中，最终发布的软件由交互复杂的对象和程序组合而成。进行自动化单元测试时，必须使用一些按设计意图表现且经过简化的对象和程序，以降低复杂性并更有利于测试。Test double是一个总称，指代那些对象和程序。

> Fakes, stubs, and mocks all belong to the category of _test doubles_. A test double is any object or system you use in a test _instead of_ something else. Most automated software testing involves the use of test doubles of some kind or another. Some other kinds of test doubles include **dummy values**, **spies**, and I/O **blackholes**.

Fake，stub，mock这些都属于test double。Test double代指在测试中用以替代真实依赖的任何对象或系统，大部分自动化软件测试都会使用到一种或多种test double。另外还有一些其它的test double，包括dummy，spy和I/O blackhole。

## Fake

> A **fake** is an implementation that behaves "naturally", but is not "real". These are fuzzy concepts and so different people have different understandings of what makes things a fake. 
>
> One example of a fake is an in-memory database (e.g. using sqlite with the `:memory:` store). You would never use this for production (since the data is not persisted), but it's perfectly adequate as a database to use in a testing environment. It's also much more lightweight than a "real" database.
>
> As another example, perhaps you use some kind of object store (e.g. Amazon S3) in production, but in a test you can simply save objects to files on disk; then your "save to disk" implementation would be a fake. (Or you could even fake the "save to disk" operation by using an in-memory filesystem instead.)
>
> As a third example, imagine an object that provides a cache API; an object that implements the correct interface but that simply performs no caching at all but always returns a cache miss would be a kind of fake.
>
> **The purpose of a fake is _not_ to affect the behavior of the system under test**, but rather to _simplify the implementation_ of the test (by removing unnecessary or heavyweight dependencies).

Fake的实现是一个真实的简化实现，不适用于生产环境，但对于测试环境是足够的。简化的目的是在不改变在测系统行为的情况下，降低测试的复杂性。例如使用内存数据库代替真实的数据库，使用本地文件系统代替OSS系统，去掉缓存等
{:.info}

## Stub

> A **stub** is an implementation that behaves "unnaturally". It is preconfigured (usually by the test set-up) to respond to specific inputs with specific outputs.
>
> **The purpose of a stub is to get your system under test into a specific state.** For example, if you are writing a test for some code that interacts with a REST API, you could _stub out_ the REST API with an API that always returns a canned response, or that responds to an API request with a specific error. This way you could write tests that make assertions about how the system reacts to these states; for example, testing the response your users get if the API returns a 404 error.
>
> A stub is usually implemented to only respond to the exact interactions you've told it to respond to. But the key feature that makes something a stub is its _purpose_: a stub is all about setting up your test case.

Stub的实现并非真实实现，通过预先对某些特定的输入指定输出，从而让在测系统处于某种特定的状态，可看作测试用例设置的一部分。例如使用stub代替真实的远程API调用，只返回一些指定的响应
{:.info}

## Mock

> A **mock** is similar to a stub, but with _verification_ added in. **The purpose of a mock is to make assertions about how your system under test interacted with the dependency**.
> For example, if you are writing a test for a system that uploads files to a website, you could build a _mock_ that accepts a file and that you can use to assert that the uploaded file was correct. Or, on a smaller scale, it's common to use a mock of an object to verify that the system under test calls specific methods of the mocked object.
> Mocks are tied to _interaction testing_, which is a specific testing methodology. People who prefer to test _system state_ rather than _system interactions_ will use mocks sparingly if at all.

Mock的实现是什么都不做，但会记录在测系统与它的交互，使得在测试时可以断言在测系统与依赖之间的交互。例如使用mock代替真实的文件上传系统，可以验证上传的文件是否正确。使用mock代替真实依赖的对象，可以验证在测系统是否调用了依赖对象的某个方法
{:.info}

一些单元测试框架如Mockito，它的mock对象实际上可兼具stub和mock的特性
{:.info}

## 参考文档

* [https://stackoverflow.com/questions/346372/whats-the-difference-between-faking-mocking-and-stubbing](https://stackoverflow.com/questions/346372/whats-the-difference-between-faking-mocking-and-stubbing)
* [https://en.wikipedia.org/wiki/Test_double](https://en.wikipedia.org/wiki/Test_double)