---
layout: post
title: A Look Into Open Source Contributions
date: 2019-09-16
categories: Lab
---

We were asked, as a part of [Lab 1](https://wiki.cdot.senecacollege.ca/wiki/SPO600_Code_Review_Lab), to look at a couple of Open Source projects and find out what the contribution workflow looks like from start to finish. I decided to take a look at two projects that use on a daily basis:

-   [VSCode](#vscode)

-   [Jekyll](#jekyll)

<!-- more -->

### VSCode

It is easy to confuse Visual Studio with VSCode, both are from Microsoft and the UI looks somewhat similar. However, they are vastly different software. Although VSCode can be used as an IDE, it is really intended to be more of a lightweight editor. On the other hand, Visual Studio is a fully fledged IDE, and as such it is a much heavier program.

The other big difference is the fact that VSCode is an open source project. Yes, even though it is backed by Microsoft anyone can help make it better.

The first thing I noticed when checking out the [VSCode Repo](https://github.com/microsoft/vscode) is that it is a massive project. With over 50,000 commits, almost 200 pull requests and more than 900 contributors. It has pretty much daily commits to its master branch, and although it looks like there are a couple of main contributors there appears to always be someone making a one off pull request. If contributing is what you are after, well there are nearly 6,000 issues open so there might be something that can get done at any one time.

Now that we have explored the size of the project and seen that they are open to new contributions lets find out how we can go about contributing.

##### How to contribute to VSCode

I'm not sure if it is because the project is backed by Microsoft or just my inexperience but if there's something I love about VSCode is its [documentation](https://code.visualstudio.com/docs). It is very detailed without sacrificing readability, unlike other projects in which you need to learn a new language just to understand their docs. It comes as no surprise to me that there's an entire section on their [wiki](https://github.com/microsoft/vscode/wiki) dedicated to [Contributions](https://github.com/microsoft/vscode/wiki/How-to-Contribute).

So on the VSCode wiki they let us know that there are different ways of contributing, from reporting bugs to making suggestions, as well as actually submitting pull requests. It appears anyone can take on any of the available issues, though they tell us that if we're trying to make a significant change we must discuss it with someone who has been assigned to overlook that issue. By taking a look at some of the issues it seems like this is where Microsoft employees come into play as they appear to be the ones who get assigned the main issues.

The wiki walks you through the entire process of setting up a developer environment, so I'm not going to repeat it here. I'm more interested in finding out what communication channels they use to report and track issues, and what the whole contribution process looks like.

They have various channels of communication. [GitHub Issues](https://github.com/Microsoft/vscode/issues) is used for reporting and tracking bugs and suggestions. They also have a [Slack workspace](https://vscode-dev-community.slack.com/join/shared_invite/enQtNTYzMzA5MTc0Njc0LTBkNzI4ZjhjYTMwMDNhZWNlYTM0ODE5OGFiZDMyYmM0Y2YyZDRkOGFhZDg0NmQ1MjYxZjg1MDc4NDU2ZDRkNjU) and [Gitter](https://gitter.im/Microsoft/vscode) chat rooms, but they're mainly for the community and not to interact with the VSCode team directly.

As I said before the more noteworthy changes need to go through a very rigorous code review by someone from the official team. Contributors are suggested to take on issues that are labeled `help-wanted` and `good first issue`. You're also required to sign a Contributor License Agreement before a pull request can be accepted.

Since a member of the official team needs to review each pull request they ask that we only make one pull request per issue and link the issue in the pull request. This way the code review process can be more efficient and the chances of our fix being implemented increased.

It is a good idea before thinking about making a contribution to the code to check their [Coding Guidelines](https://github.com/microsoft/vscode/wiki/Coding-Guidelines). For more information check the [VSCode Wiki](https://github.com/microsoft/vscode/wiki/How-to-Contribute).

### Jekyll

[Jekyll](https://github.com/jekyll/jekyll) is a static site generator, it renders Markdown and Liquid templates into a static website that can be served by any web server. It is the rendering engine behind GitHub Pages and is what I'm using to create this site.

What I love most about Jekyll so far is its simplicity and speed. I won't deny its initial configuration takes a bit of time but once you're set up the workflow is so much faster than any CMS I've tried before. Jekyll stays out of my way and doesn't try to do more than what is asked of it, which is to simply spit out a static site.

##### How to contribute to Jekyll

Jekyll is a much smaller project than VSCode, and probably due to there not being a corporation like Microsoft in the back, it seems to be a more open and friendly community. While the overall tone in the VSCode wiki seems to be professional and serious business, going through Jekyll's docs is more like talking to a friend. By this I don't mean they're still not thorough, but it does make you feel more welcomed.

Similarly to the VSCode project they use the `help-wanted` tag to track open issues and everyone is encouraged to participate in solving it. Contributions can range from reporting an issue, making a suggestion, creating a pull request or reviewing someone's pull request. Likewise smaller changes and only one change per pull request are preferred.

They appear to use GitHub Issues for tracking contributions, and for general help there is an [official forum](https://talk.jekyllrb.com/) and [Gitter](https://gitter.im/jekyll/jekyll).

There's a whole area dedicated to [Code Contributions](https://jekyllrb.com/docs/contributing/#code-contributions) in their docs, and you're encouraged to read it.

### Conclusions

Overall both project's contribution process seems very similar. Possibly because they're both on GitHub and the GitHub workflow forces them to be that way. The VSCode project seems to be being built with the power of the community but to me it looks like Microsoft still has a major say in what gets implemented and the direction of the overall project. Personally I prefer how the community in the Jekyll project appears to be more open to suggestions. I may just be biased against Microsoft for things they have done in the past, and that's why I see it things this way. Regardless I'll continue using VSCode as my main editor for now.
