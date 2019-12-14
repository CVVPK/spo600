---
layout: post
title: Profiling VsCodeVim
date: 2019-12-11
categories: Project
---

As I mentioned on my [last post], I'm now working on the [VsCodeVim] extension as my project for SPO600. Hopefully, my professor will see this and have mercy on my for having abandoned all efforts on my previous project (Opus). 

Here is what I did for profiling VsCodeVim. I used multiple files ranging from 250 KB to 3 MB. I have been using VSCodeVim for over a year, and have always known that its performance on larger files is pretty lack luster. If you take a look at the issues tagged as performance in [GitHub][https://github.com/VSCodeVim/Vim/labels/area%2Fperformance], you'll see that a lot of them are quite similar. Large file and repeated actions slow down the editor.

I was initially enthralled by this particular issue: ['Investigate optimizing HistoryTracker.addChange()']. That gave me an idea of what to look for when doing my profiling. 

Profiling a VSCode 


[last post]:
[VsCodeVim]:
['Investigate optimizing HistoryTracker.addChange()']:https://github.com/VSCodeVim/Vim/issues/3920