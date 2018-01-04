---
title:  "Monorepo"
description: Monorepo
date: 2017-03-27 14:00:00
---
At Shopify we're thinking on the idea of moving our mobile apps to
monorepos. A monorepo offers several advantages compared to multirepo, such as:
- Atomic refactors across projects: It is possible to refactor code in different projects in a single commit.
- Easier dependency management: If all logical projects are located in a single repository, the main apps will always use the latest version of the dependencies.
- Tooling: If all the projects and dependencies lies in the same repository, writing tools is easier, as those tools don't need to understand the relationships between the projects.
- Easier set up: When all the apps and dependencies are located in the same repo, the developer just need to clone and run it, without thinking on external dependencies and versioning.

But the monorepo approach is not the solution to all and every problem, as
it introduces new challenges that should be faced to make it scalabe, efficient and fast.

<!-- Selective tests >
<!-- Different pipelines for CI >
<!-- Faster git clones >
<!-- Parallel pipeline execution >
<!-- Cache (Gradle, gems, apks...) >