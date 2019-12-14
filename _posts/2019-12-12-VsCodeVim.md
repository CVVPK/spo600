---
layout: post
title: Profiling VsCodeVim
date: 2019-12-12
categories: Project
---

As I mentioned on my [last post], I'm now working on the [VsCodeVim] extension as my project for SPO600. Hopefully, my professor will see this and have mercy on my for having abandoned all efforts on my previous project (Opus). 

Here is what I did for profiling VsCodeVim. I used multiple files ranging from 250 KB to 3 MB. I have been using VSCodeVim for over a year, and have always known that its performance on larger files is pretty lack luster. If you take a look at the issues tagged as performance in [GitHub](https://github.com/VSCodeVim/Vim/labels/area%2Fperformance), you'll see that a lot of them are quite similar. Large file and repeated actions slow down the editor.

I was initially enthralled by this particular issue: ['Investigate optimizing HistoryTracker.addChange()']. That gave me an idea of what to look for when doing my profiling. 

Profiling a VSCode extension is not very difficult, I won't go over the steps here, but if you are interested just follow this [link].  As mentioned before, I did multiple profilings and the results were pretty consistent with these: 

- [largeFileFastEditing.cpuprofile.txt]({{ site.baseurl }} {%link /assets/../../../_site/assets/largeFileFastEditing.cpuprofile.txt %})
- [largeFileSlowEditing.cpuprofile.txt]({{ site.baseurl }}{% link /assets/largeFileSlowEditing.cpuprofile.tx %})
- [oldNavigation.cpuprofile.txt]({{ site.baseurl }}{% link /assets/oldNavigation.cpuprofile.txt %})

For reference, here's the profile of vscode with the Vim extension disabled:
- [noVim.cpuprofile.txt]({{ site.baseurl }}{% link /assets/noVim.cpuprofile.txt %})

To View these files, simply remove the txt extension, open up the developer tools in chrome (anything based off of chromium works), open the menu next to the 'x', go to more tools, click on Javascript profiler and load up the cpuprofiles. 

Through the profiling I have identified three major hotspots inside of the `HistoryTracker.addChange()` function, from biggest to smallest they are:

 1) _getDocumentText()
 2) advancePositionByText()
 3) diff_main()

The first one `_getDocumentText()` makes a call the vscode.TextDocument.getText() to retrieve the text of the entire document. It is extremely inefficient, when working with large files. I suspect that TextDocument saves its state by saving lines in an array or similar data structure and only when we call the getText() method it joins them and gives us a string. The reason I think this is what is happening, is because when we get smaller ranges of text it performs better. However, joining up those smaller blocks of text takes about as much time a it was saved by not getting the entire document. Changes to how this works are not possible.

The second `advancePositionByText()` is pretty slow on large files because it scans really big chunks of text when changes are made, looking for '\n' to count the number of lines. On my next post I'll go over a possible way to make this better.

Lastly, `diff_main()`, the function call that spiked the whole idea of optimizing `HistoryTracker.addChange()`. Funnily enough, it is the least of our worries it seems. The guys behind the `js.diff` package have done an amazing job at optimizing it. It really doesn't hurt our performance as much as initially thought. 

Doing the profiling helped me get ideas of what needed to be changed, on my next post I'll go over a few things that I tried. Some of them worked, some of them didn't, but all of them helped me arrive at an optimization that actually works.


[last post]:{{site.baseurl}}{% post_url 2019-12-11-The-State-Of-Things %}
[VsCodeVim]:https://github.com/VSCodeVim/Vim
['Investigate optimizing HistoryTracker.addChange()']:https://github.com/VSCodeVim/Vim/issues/3920
[link]: https://github.com/microsoft/vscode/wiki/Performance-Issues

