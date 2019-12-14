---
layout: post
title: Optimizing VSCodeVim (Part 2)
date: 2019-12-14 13:00:00
categories: Project
---

On [Part 1] I showed my proposed changes to improve the performance of `HistoryTracker.addChange()` while we are in Insert Mode. That change will help solve many reported problems of lag when editing large files. On this post I'll go over other changes that can further improve the performance of this function by ensuring we aren't unnecessarily processing the entire file.

The first thing I found, through some experimentation of the [vscode API], was that the bigger the chunk of data that we ask from `TextDocument.getText()` the worse this function performs. It is my suspicion that a `TextDocument` maintains its state in lines rather than a single string of text, thus when we ask for the text it first needs to build the string which is why it takes so long. 
 
One of the biggest disadvantages of JavaScript is its single threaded nature, multi threading is an after thought that can be achieved through [web workers]. In Node.js there
are [worker threads] which accomplish a similar purpose. I considered the possibility of using worker threads to split up the process of getting the entire document. However, getting the text is not a very CPU-intensive task as what we are doing is very simple operations, it takes a while because a lot of them need to be done, thus it is I/O-intensive. Worker threads are not recommended for I/O intensive tasks, since Node.js internally processes asynchronous code on separate threads to prevent blocking the event loop.

I figured we could find the size of the file, use that to determine the optimal `maxBits` a block should be, then use this to asynchronously call `TextDocument.getText()` to get small blocks of the file's contents rather than one single chunk. Leveraging from Node.js's internal optimizations that would resolve each of these promises on different threads, we could speed up the operation of getting the text quite a bit. The code looks like this:

```ts
const { editor } = this.vimState;
const documentEnd = cursorPosition[0].getDocumentEnd();
const eof = editor.document.offsetAt(documentEnd);

const maxBits = Math.ceil(eof / 15);
const gettingText = async (range: vscode.Range) => {
    return (
    (editor &&
        editor.document &&
        editor.document.getText(range)) ||
    ''
    );
};

// Make an array of promises, each promise gets a small range of data determined by the maxBits.
const getText = Array.from({ length: Math.ceil(eof / maxBits) }, (v, i) => {
    const startOffset = i * maxBits;
    const endOffset = (i + 1) * maxBits > eof ? eof : (i + 1) * maxBits;
    const startPosition = editor.document.positionAt(startOffset);
    const endPosition = editor.document.positionAt(endOffset);

    const range = new vscode.Range(startPosition, endPosition);

    return gettingText(range);
});

// Wait until all promises are resolved and then join up all the blocks.
const newText = await Promise.all(getText).join('');

/* Current Implementation of addChange() goes here */
```

This solution results in about a 5-10% improvement from the performance of `TextDocument.getText()`. However, this is counterbalanced by the overhead added from creating the array, and mostly by `Promise.all()` and `join()`. In fact this has a null effect on performance on large files, and a slightly negative effect on smaller files. So it has been discarded as an option.

Since it seems we do need to check the whole document in some cases, and because there might not be a better way to get it than using `TextDocument.getText()`, I concluded that the best approach would be to delay making this call as much as possible. One possible solution to this is to first check the line around the cursor and find if that's where the changes happened. One of my failed attempts at doing this is below:

```ts
let newText = '';

for (const cursor of cursorPosition){
    const offset = this.vimState.editor.document.offsetAt(cursor);

    if (this.vimState.editor && this.vimState.editor.document) {
        const newLineRange = this.vimState.editor.document.lineAt(lineNum).rangeIncludingLineBreak;
        const newLine = this.vimState.editor.document.getText(newLineRange);
        const longerNewLine = charNum <= newLine.length;
        const sol = longerNewLine ? offset - charNum : offset - charNum + 1;
        const eol = sol + newLine.length;
        const oldLine = this.oldText.substring(sol, eol + 1);
        newText = this.oldText.substring(0, sol) + newLine + this.oldText.substring(eol);
    }
}
```

There are many issues with this. For instance, it cannot handle adding a new line or removing one very well or changes concurrent changes happening outside the cursor position's line that weren't triggered by the user. There are many special cases that need to be considered and for now I have scratched the whole idea. Though, I do think it would be great to find a way of checking the text around the cursor position first and avoid getting all the text.

Preventing the unnecessary execution of `TextDocument.getText()` remained the main priority. Through the changes I have listed above and some others I forgot to record, I realized something. The zero based offset returned by `TextDocument.offsetAt()` represents the location that follows the last character in the document. Thus, it is equals to the length of the document's text. 

Previously, we were getting the document's text as the first step then comparing to the old text and using this result to determine if there are new changes that need to be processed. But, if the new text's length is equals to old text length then we have no changes to process. My proposed solution is as follows:

```ts
    // Get the current document's text length
    // which is conveniently equals to the zero based offset at the document's end.
    const documentEnd = cursorPosition[0].getDocumentEnd();
    const newTextLength = this.vimState.editor.document.offsetAt(documentEnd);

    // Check if the document's text length changed.
    if (newTextLength === this.oldText.length) {
      return;
    }
```

This has a major impact on navigating through large files. Before these changes, simply holding down a navigation key would incur multiple calls to get the document's change. This resulted in about 40% of our time being spent inside `HistoryTracker.addChange()`. With this change performance is improved by the same amount, since we will really spend no time inside `HistoryTracker.addChange()`. Here's a cpuprofile of these changes:

[newNavigation.cpuprofile.txt]({{site.baseurl}}{% link /assets/newNavigation.cpuprofile.txt %})


[Part 1]:{{site.baseurl}}{% post_url 2019-12-13-Optimizing-VSCodeVim-(Part-1) %}
[vscode API]:https://code.visualstudio.com/api
[web workers]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers
[worker threads]: https://nodejs.org/api/worker_threads.html