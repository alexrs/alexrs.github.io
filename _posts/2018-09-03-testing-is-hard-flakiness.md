---
title:  "Testing is hard: flakiness"
description: Executing tests at scale and dealing with flakiness.
date: 2018-09-03 14:00:00
layout: post
---
A couple of days ago I was involved in a conversation in the
[Developer Productivity](https://devproductivity.io/) Slack channel
about testing, and it made me think about how to effectively test large
codebases with thousands (or millions) of tests.

So let me tell you a story on how to execute tests at scale.

But first, a secret: your tests are going to fail. Eventually.
Not only because writing tests is hard but also because executing tests is not easy.
[Murphy's law](https://en.wikipedia.org/wiki/Murphy%27s_law) also applies to testing.

Imagine the simplest test you can write. Do you have something in mind? This is the
simplest, most robust test that comes to my mind.

```ruby
def assert_the_truth
  assert true
end
```

And it ~~will~~ can fail.

As pretentious as it sounds, I have come up with a theorem that explains it:
**The Infinite Testing Theorem**.

> The Infinite Testing Theorem states that any test executed an infinite number of times will [almost surely](https://en.wikipedia.org/wiki/Almost_surely)
> fail.

But, why are you so sure? You'll be wondering. And what options do we have to execute tests reliably,
avoiding false negatives, false positives, and being able to trust in our test suite?

Well, fortunately we have many, because if we can't trust in our tests, the tests are useless.

## Single test execution
The most common approach when executing tests is to run `bundle exec rake test` or `cargo test` or
any other test runner used in your projects.

These test runners scan your project looking for tests to execute. Once they've found what they're looking for,
they run the tests, usually in a different order every time the command is called to find the tests that depend on other tests to pass. That's great. But sometimes it's not enough to ensure quality in your test suite.

A green test suite doesn't indicate that your tests will pass the next time.

Wait, what?

Yes. Flakiness is everywhere. We use computers, networks, compilers, runtimes, framewors.
All of those things can fail, resulting in a flaky test<sup>[1](#flaky_test_fn)</sup>.

There are two sources of flakiness we have to deal with:
- Flaky tests: This are the tests that could fail or pass for the same configuration. The most common
sources of flakiness are concurrency, [caches](https://martinfowler.com/bliki/TwoHardThings.html) or
environments in a bad or unexpected state.
- Infrastructure failures: Here I include bugs in the testing framework, the compiler,
your computer, the network, your CI system, [failures due to cosmic rays](https://stackoverflow.com/questions/2580933/cosmic-rays-what-is-the-probability-they-will-affect-a-program)
... I think you get the idea.

What can we do to address and fix this problems?
Well, here we need to make a decission: whether we care more about the quality
of our test suite, or about having a passing test suite.

## Execute tests multiple times
If we care about the quality of our test suite,  we can do two different things
when encountering a flaky test (that I can think of).
- Fail the build if we have a test failure. This is the most common action to do and the default behavior of
the test runners. But we can still end up with flaky tests that pass, although
they will fail eventually.
- Execute each test several times and fail the build if we find a test failure. This approach will reduce
the number of flaky test faster but your test execution will be *N* times slower, being *N* the time it takes
to execute the test suite.

This is a nice thing to do if the quality of your software can't be compromised and it's preferible
to spend more time ensuring the quality of your tests.

But every piece of software is different and sometimes it's better
to move faster even if that means dealing with flaky tests. Quoting Facebook's former motto,
*Move Fast and Break Things... with a passing test suite.*. And what about the infrastructure flakiness?

## Retries
Retries are a simple yet powerful mechanism to improve the pass rate of our test suite.
Let's see how retries can be used to address the two sources of flakiness previously described!

Infrastructure failures happen. And sometimes we don't own and can't control the infrastructure.
And even if we do, some failures are unavoidable. Luckily for us, those issues are easy to identify
and it is possible to retry them. An infrastructure failure doesn't indicate that the test is missbehaving,
so it wouldn't make sense to fail a build because something external failed.

But what about those tests with a legit flaky failure? Well, those ones can be retried too if you want to
increase the pass rate and you aim for an ever-green master branch.
The problem with this approach is that introducing flakiness will no longer be a problem, and developers
would potentially care less about the quality of their tests.
Moreover, if we allow flaky test in our test suite, developers will start assuming that
[any failure is probably just a flaky test](https://en.wikipedia.org/wiki/The_Boy_Who_Cried_Wolf).

## Remove failing tests
We can also execute the tests and deactivate all of the failing tests.

Yes.

If a test fail, you don't execute it (until it's fixed). As crazy as it sounds, it may be the right approach
under some circumstances. Imagine that you have a monorepo where thousands of developers
commit millions of code lines every day and execute millions of tests.
You cannot fail a build or a deploy because one test failed. But you can deactivate that test, report
it to the team owning that part of the code, and hoping for the best (which is usually the test being fixed
and no other team breaking something because that test was not being executed.)

## Staging test suite
Another interesting approach is to have two test suites, a *stable* one and a *staging* or *quarantine*
test suite.
If we detect a broken test, it is moved to the staging test suite. Also, new or refactored tests are placed
in *staging* until they prove to be reliable enough to move to *stable*.

## Run less tests
The best way to avoid test failures is not to run tests at all. But let's be honest, that's not
feasible. What we can do is to **run less tests**.

There are two approaches we can use to run less tests. The first one is not to run
tests that have not failed in a long period of time. At some point (randomly, from time to time...)
we should run those tests and if we find test failures, the failing ones will be executed again every time.
This will avoid infrastructure failures but not flaky failures, as the flaky tests will fail often enough
that they will never be considered to not be runned.

The second approach is to perform Regression Test Selection. This technique will analyze the code that
has been changed and will run only the tests that can fail (ie, the test that are related to the changes).
As you can imagine, this is not an easy thing to do, and has an extra cost of selecting the tests that need
to be run (sometimes this can take longer than running the entire test suite).

## Conclussions
Testing and automation is [hard](https://xkcd.com/1319/). There isn't a right or correct approach to
execute your tests or ensure the quality of your software through testing, and deciding which one to
use depends on what is better for you, your customers or your company. But only once you are comfortable
with the fact that your tests are going to fail, you can choose how to address that problem.

Happy testing!

## References
- [Flaky Tests - A War that Never Ends](https://hackernoon.com/flaky-tests-a-war-that-never-ends-9aa32fdef359)
- [Flaky Tests at Google and How We Mitigate Them](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)
- [Taming Google-Scale Continuous Testing](https://drive.google.com/file/d/0Bx-FLr0Egz9zYXJfMEZ6NERTbkU/view)
- [ Running an Automated Test Pipeline for the League Client Update ](https://engineering.riotgames.com/news/running-automated-test-pipeline-league-client-update)
- [Regression testing](https://en.wikipedia.org/wiki/Regression_testing#Regression_test_selection)
- [Eradicating Non-Determinism in Tests](https://martinfowler.com/articles/nonDeterminism.html)

<hr/>

<a name="flaky_test_fn">1</a>: A flaky test is a test that exhibits both a passing and a failing result with the same code.
