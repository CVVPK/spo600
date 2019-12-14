---
layout: post
title: The State Of Things
date: 2019-12-11
categories: Project
---

After my [previous post], I was pretty unmotivated with the state of my project as it seemed everything I was trying to do on the Opus encoder was absolutely pointless. Shortly after that post went up I talked with a member of the community about my idea of vectorizing the `silk_warped_autocorrelation` and I learned that such a change was heavily discouraged. It seemed like all along I was following the wrong path to trying to optimize the Opus encoder. 

For a few days I kept banging my head against a wall, trying to come up with an idea. I really wanted to get a change upstream. Unfortunately, it just didn't seem likely with a project such as Opus were a lot of optimizations have already happened. I have abandoned the Opus project for now as it seemed unlikely that I'd be able to make a change that would be accepted on the upstream within the time frame I had. 

Because of this, I started to toy with the idea of working on my secondary option: [VSCodeVim/Vim]. This is a plug in for VsCode that emulates the Vim editor. I use it on a daily basis, since VsCode is my editor of choice. 

Initially I had disqualified the Vim plug in as a possible project because I was afraid that finding an optimization would be extremely difficult due to it being written in a very high level language (TypeScript) and many of its downfalls being because of the VsCode API. On the other hand, Opus had some pocket sized functions that I could try to do something about.

I've switched gears and over the past few days I have been focusing on VSCodeVim. I wish had chosen it as my first option, after all it filled the three rules I had described on ['The Quest To Finding A Project']. Since I started working on it I have submitted two PRs ([1],[2]) that have gone upstream, both of them fixing major bugs introduced on the latest version.

However, my objective on SPO600 is not to fix bugs but to apply an optimization. So over the next few days I'll go over the profiling, what I have found, what I tried, what didn't work, and most importantly what worked.


[previous post]: {{site.baseurl}}{% post_url 2019-11-30-Fixed-Point-Implementation-Of-Opus-(Part-2) %}
[VSCodeVim/Vim]: [VSCodeVim/Vim]
['The Quest To Finding A Project']: {{site.baseurl}}{% post_url 2019-11-01-The-Quest-To-Finding-A-Project %}

[1]:https://github.com/VSCodeVim/Vim/pull/4335
[2]:https://github.com/VSCodeVim/Vim/pull/4337