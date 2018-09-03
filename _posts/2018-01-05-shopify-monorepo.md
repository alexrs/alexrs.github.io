---
title:  "Scaling your monorepo for an efficient CI"
description: Characteristics for an effective Continuous Integration with monorepos.
date: 2018-01-05 14:00:00
layout: post
---
At [Shopify](https://shopify.com) we’re thinking about the idea of moving our mobile apps to a monorepo.
This model offers several advantages compared to having multiple repositories, such as:

- Atomic refactors across projects: It is possible to refactor code in different projects in a single commit.
- Easier dependency management: If all logical projects are located in a single repository,
the main apps will always use the latest version of the dependencies.
- Tooling: If all the projects and dependencies lie in the same repository,
writing tools is easier, as those tools don’t need to understand the relationships between the projects.
- Easier set up: When all the apps and dependencies are located in the same repo,
the developer just need to clone and run it, without thinking about external dependencies and versioning.

Twitter, Facebook, Google or Mozilla use monorepos. But the monorepo model
is not the solution to all and every problem, as it introduces new challenges
that should be faced to make it scalable, efficient and fast on your Continuous Integration system.

### Selective tests
Running all the tests for all the projects in the repository can be
impractical depending on its size and the number of tests. Moreover,
some projects can be penalized if the tests of a completely unrelated
project inside the repo run slower.

### Different pipelines for CI
Having multiple projects in the same repo doesn't mean that those
projects must be coupled. One of the problems to solve is to have
multiple isolated projects in the same repo that can share code. If those projects share the same
Continuous Integration pipeline file, in which all the steps for CI are declared,
this goal is not being met.
Most of the CI systems only allow one file in which all the steps to run in CI are declared,
which will cause that we must invest some time developing tools to tackle this limitation, so we can
have one pipeline for each project in the repository.

### Faster git clones
Git is the most common version control system (although some companies such as Facebook or Mozilla
use Mercurial), and it doesn't scale well with huge repositories. We should avoid cloning the entire repository, with all
branches, and the entire git history. Nowadays, getting parts of a repository is not possible with git, but we can
speed up the `git clone` process doing a shallow clone, which will only clone the latest version of the specified branch.

### Parallel pipeline execution
If we have different pipelines for each project, the next step is to execute them in parallel. It doesn't make sense
to execute unrelated pipelines sequentially if they don't depend on each other. Moreover, executing only the tests
related to the changes made will avoid executing unrelated pipelines.


In conclusion, moving to a monorepo model is not for everyone. It makes sense
if you have several dependencies used in different projects, the size of the
project is increasing, and you're finding difficulties in sharing code and keeping
the dependencies up to date. But the migration to a monorepo model also
entails an investment in tooling that it's not worth for everyone.

### References
- [Advantages of monolithic version control](https://danluu.com/monorepo/)
- [Why Google Stores Billions of Lines of Code in a Single Repository](https://www.youtube.com/watch?v=W71BTkUbdqE)
