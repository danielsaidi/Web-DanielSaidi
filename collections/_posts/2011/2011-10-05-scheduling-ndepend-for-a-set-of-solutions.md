---
title: Scheduling NDepend for a set of solutions
date:  2011-10-05 12:00:00 +0100
tags:  sponsored
---

I use NDepend to continuously run a scheduled code analysis on a bunch of solutions that make up a large part of the software infrastructure of a major Swedish company.

By scheduling them to run once a week, using previous results as a comparison baseline, it will be easier to detect what avoid and pin-point good practices that we want to embrace.

Although we use Team City, I can also run these analyses from my computer if needed.

The analysis workflow is triggered from a simple bat script that does the following:

- It first checks out each solution from source control.
- It then builds each solution with `devenv`.
- It then run a pre-created NDepend analysis for each solution.
- Each analysis is configured to publish the HTML report to a web serve.
- The web server is available for everyone in the project.

I have scheduled the script with the `Task Scheduler`, to run every Monday at 08.30. This works like a charm. The analyses runs each week and everyone is happy (at least I am).

After the first analysis, we noticed some things we could tweak to improve the architecture, reduce merge conflicts, avoid code duplication, etc. We look forward to improve over time.


## One small tweak

When running the script manually, we noticed that it failed to launch NDepend if the code was unchanged. This was confusing, but the solution was simple.

Under `Tools / Options / Anaysis`, you can tell NDepend to always run a full analysis:

![NDepend](/assets/blog/11/1005-2.png "NDepend")

The default setting is often correct, since it will run a full analysis at least once per day, but we will keep this checked while we evaulate the process.


## One final, small problem – help needed!

One remaining problem is that NDepend sometimes incorrectly complains that the solution DLLs are invalid. The last time this happened (after a major architectural change), I could not get it to work even by deleting and re-adding the DLL:s.

To solve this, I had to delete the NDepend project and re-create it from scratch. This is not ideal, so I hope that we will find a solution for it.