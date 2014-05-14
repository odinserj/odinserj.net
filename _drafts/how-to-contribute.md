---
layout: post
title: How to Contribute
---

Open source projects thrive on contributions from the developer community. Would like to get involved? There is plenty that you can do to help!

We appreciate any contribution and know, that guidelines sometimes make it harder to start to contribute. However you should remember that the reading takes much more time than the writing, and the number of readers is much more then the number of writers.

So, if you want to save our time, please read the following notes. If you experience any problem with them, ask a question on a [discussion forum](http://discuss.hangfire.io).

There are many ways you can help the project, and code contributions is only one of them. Here is the list of what you can do to help to make the project even more great.

### Submitting issues

HangFire uses [GitHub Issues Tracking](https://github.com/odinserj/HangFire/issues) to track issues (primarily bugs and contributions of new code). If you've found a bug in HangFire, this is a place to start.

When filing issues, please use our bug filing templates. But first make sure that it has not been previously submitted. The best way to get your bug fixed is to be as detailed as you can be about the problem. Providing a minimal project with steps to reproduce the problem is ideal. Here are questions you can answer before you file a bug to make sure you're not missing any important information.

1. Did you read the documentation?
2. Did you read the FAQ?
3. Did you include the snippet of broken code in the issue?
4. Can you reproduce the problem in a brand new project?
5. What are the EXACT steps to reproduce this problem?
6. What operating system are you using?
7. What version of IIS are you using?
8. What version of Visual Studio?

Github supports markdown, so when filing bugs make sure you check the formatting before clicking submit.

If you already know the way to fix an issue, please, open a pull request with the code fix and fill the information about the issue directly to it. But first read the section "Contributing code" to make it clear about contribution process.

When reporting a bug or issue, please include all pertinent information. This typically includes:

* Development platform, including .NET version and web server (Example: Mvc3 with .NET version 4.0 on IIS Express)
* Steps to reproduce the bug/example code
* Any error messages and stack trace
* It is also quite helpful to include the relevant portions of HangFire's log file. You can enable HangFire logging by adding the loggingEnabled attribute in web.config.

Bugs will be addressed as soon as humanly possible, but please allow ample time. For quicker responses, you may also choose to implement and contribute the bug fix.

### Discussing new features

New features sometimes require more discussion, and every new feature tends to be splitted into a couple of another features, or vice versa. It is always hard to maintain this discussion on a issue tracker and there is difficulties in understanding for the feature completion state.

That is why a [discussion forum](http://discuss.hangfire.io) is a great place for new feature requests.

The message should describe the contribution you are interested in making, and any initial thoughts on implementation. This will allow the community to discuss and become involved with you from the get go. If you receive positive feedback on the mailing list, go ahead and start implementation! You should also sign and return the Contributor License agreement, which is required for the HangFire team to accept your contribution.

After the feature is ready enough to be implemented, there should be a GitHub work item that needs to be implemented.

### Contributing code

HangFire maintains several issues that are good for first-timers tagged as Jump In on GitHub. If one peaks your interest, feel free to work on it and let us know if you need any help doing so.

* [Learn more about how "Jump In" issues work](http://nikcodes.com/2013/05/10/new-contributor-jump-in/)

HangFire follows a loose set of coding conventions. Chiefly among them:

* Ensure all unit tests pass successfully
* Cover additional code with passing unit tests
* Try not to add any additional StyleCop warnings to the compilation process
* Ensure your Git autocrlf setting is true to avoid "whole file" diffs.

Our workflow is loosely based on Github Flow. We actively develop in the dev branch. This means that all pull requests by contributors need to be developed and submitted to the dev branch. The master branch always matches the current release on nuget.org and we also tag each release. When the end of a milestone is coming up, we create a branch called release to stabilize the build for the upcoming release. The release is then merged into master and deleted and the cycle continues until the end of the next milestone.

### Sharing HangFire

If you love HangFire, tell others about it! Present HangFire at a company tech talk, your local user group or submit a proposal to a conference about how you are using HangFire or any extensions you may have written. This can help HangFire to be more stable and feature rich.

### Contributing to documentation

Documentation is a key differentiator between good projects and great ones. Whether you’re a first time OSS contributor or a veteran, documentation is a great stepping stone to learn our contribution process.

HangFire documentation is generated using Sphinx documentation builder. It uses RestructureText as a source format (it is a text format, like markdown), and placed on the Docs folder inside HangFire repository.

To make a contribution, clone the repository and make changes. If you like to see the changes, check the documentation readme and follow the given steps.

### Creating an extension

Get the best out of HangFire by writing your own extension to expose diagnostic data that is meaningful for your applications. Creating extensions is easy, check the docs or reference an open source extension to get started.

* Creating your own custom extensions and tabs
* Creating your own custom runtime policies