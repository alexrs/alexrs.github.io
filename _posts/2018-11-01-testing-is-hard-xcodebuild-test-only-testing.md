---
title:  "Testing is hard: xcodebuild test and the -only-testing flag"
description: xcodebuild has a -only-testing flag that will not behave as you expect.
date: 2018-11-01 23:00:00
layout: post
---

Some time ago I wrote [about why testing was hard when you have to execute a lot of tests](/2018/testing-is-hard-flakiness)
and I like the idea of having a series of posts under the title
_Testing is hard_ in which I write interesting things I discover about testing and 
why it is not as easy as some people can think.

Today I'm going to talk about Xcode. If you want to execute your tests from
the command line, you'll probably use `xcodebuild test`. This command
has an argument called `-only-testing` that receives a test identifier
following the format `target/class_name/test_name`.

That's cool. It allows us to only execute one test or a subset of our tests.
This can be useful in some situations, such as debugging a single test, or
retrying failing tests automatically on CI.

But imagine this hypothetical situation in which you execute your tests
on CI and you use [xcpretty](https://github.com/supermarin/xcpretty) to
generate a JUnit-style report. Then you read this report to indetify the
failing tests and retry them using the `-only-testing` flag.

It can happen that for some reason the test identifier that you pass to
`-only-testing` is not correct (...because `xcpretty` uses the messages printed
to the standard output by `xcodebuild test` to generate this report and that
is not reliable). 

So you end up with an identifier that is not valid and it doesn't _point to_ any test.
But let's recall how the identifier was formed:

```
target/class_name/test_name
```

Depending on where the error is located, the command will behave differently. 
If the target doesn't exist, the command will fail. But if it exists but doesn't 
contain a class or a test  with a name that matches `class_name` or `test_name`,
Xcode **will not execute that test, and it will NOT warn you about it.**

So you can end up with a green CI and a message saying:

```
Executed 0 tests, with 0 failures (0 unexpected) in 0.000 (0.000) seconds
```

Which is technically correct. None of the 0 test Xcode executed failed. But it 
is definitely not what you would expect.

