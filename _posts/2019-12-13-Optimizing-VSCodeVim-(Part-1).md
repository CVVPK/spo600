---
layout: post
title: Optimizing VSCodeVim (Part 1)
date: 2019-12-13
categories: Project
---

I finally arrived to what to me is the most fun part of this whole project: The code. As I mentioned on my [last post] the particular function I'm working on optimizing is `HistoryTracker.addChange()`. This function's job is to simply log a change on to the internal history. We add these changes in steps, so that when we can go back in the history one step at a time. It runs on every key press, and currently logs every single step individually. To know if it needs to add a change it first gets the entire document's text and then checks it against the last record we had of it. Getting the entire document is an extremely heavy task, as it was revealed by the profiling. Furthermore, if there is a change that needs to be logged we will then compare the whole new text to the previous one, which on large files really adds up.

Because changes can sometimes occur outside the line where the cursor is at, we can't just compare the current line to the previous one and sometimes we may need to check the entire document. My primary objective is thus to delay processing the entire document until absolutely necessary. For reference, here is the current function:

```ts
public addChange(cursorPosition = [new Position(0, 0)]): void {
const newText = this._getDocumentText();

if (newText === this.oldText) {
    return;
}

// Determine if we should add a new Step.

if (
    this.currentHistoryStepIndex === this.historySteps.length - 1 &&
    this.currentHistoryStep.isFinished
) {
    this._addNewHistoryStep();
} else if (this.currentHistoryStepIndex !== this.historySteps.length - 1) {
    this.historySteps = this.historySteps.slice(0, this.currentHistoryStepIndex + 1);

    this._addNewHistoryStep();
}

// TODO: This is actually pretty stupid! Since we already have the cursorPosition,
// and most diffs are just +/- a few characters, we can just do a direct comparison rather
// than using jsdiff.

// The difficulty is with a few rare commands like :%s/one/two/g that make
// multiple changes in different places simultaneously. For those, we could require
// them to call addChange manually, I guess...

const diffs = diffEngine.diff_main(this.oldText, newText);

let currentPosition = new Position(0, 0);

for (const diff of diffs) {
    const [whatHappened, text] = diff;
    const added = whatHappened === DiffMatchPatch.DIFF_INSERT;
    const removed = whatHappened === DiffMatchPatch.DIFF_DELETE;

    let change: DocumentChange;

    if (added || removed) {
    change = new DocumentChange(currentPosition, text, !!added);

    this.currentHistoryStep.changes.push(change);

    if (change && this.currentHistoryStep.cursorStart === undefined) {
        this.currentHistoryStep.cursorStart = cursorPosition;
    }
    }

    if (!removed) {
    currentPosition = currentPosition.advancePositionByText(text);
    }
}

this.currentHistoryStep.cursorEnd = cursorPosition;
this.oldText = newText;

// A change has been made, reset the changelist navigation index to the end
this.changelistIndex = this.historySteps.length - 1;
}
```

A previous comment left behind already acknowledges that we should find a way to avoid processing the whole document, and possibly investigate leveraging the known cursorPosition to check for changes around this point as that's where we should most often expect them to have happened. This comment is what is referrenced in the [issue] that sparked my curiosity.

As I mentioned previously, the function processes every individual change. E.g. Entering into Insert mode, then typing abc and exiting Insert Mode results in a total of 5 calls to `_getDocumentText()`, and at least 3 runs of `diff_main()` and `advancePositionByText()` which really add up. Processing the entire document while making quick changes really slows down the editor when the document is relatively large. This appears to be the major cause of lag that has been [reported] for a long time. 

The interesting part is that when we go back in the history we are really only using the starting and end points of a particular history step. All the individual changes we spend so much time recording are simply disposed of. Thus, on the previous use case we should have only processed 2 changes instead of 3. That is to say we should process the first change that initiates a history step and the last change that finalizes that history step. This would greatly improve the performance as we wouldn't be wasting time while changes are still in progress inside Insert Mode.

To do this I thought we should simply add a condition so that we get out of the function when we are in Insert Mode and the current change is not the first one. To do this we do the following:

```ts
    if (
      this.currentHistoryStepIndex === this.historySteps.length - 1 &&
      this.currentHistoryStep.isFinished
    ) {
      this._addNewHistoryStep();
    } else if (this.currentHistoryStepIndex !== this.historySteps.length - 1) {
      this.historySteps = this.historySteps.slice(0, this.currentHistoryStepIndex + 1);

      this._addNewHistoryStep();
    } else if (this.vimState.currentMode === Mode.Insert) {
      return;
    }

```

The initial if block is already in the codebase, I simply added the final else if that checks the currentMode. This way the first change after entering Insert Mode will trigger `_addNewHistoryStep()`, it will then be processed as we previously were. When we get out of Insert Mode, since this function is called on every key press, then we will process that change as the last one. This has a major impact on performance when editing a document, we go from spending 40%-60% of our time in `addChange()` , while typing fast on a large file, to around 5% under the same conditions. 

Previously the faster we typed the worse the bigger the impact to performance we had, while with my proposed change we are basically running on vanilla VSCode while we are typing. Here's cpuprofile after applying this change:

- [improvedAdd.cpuprofile.txt](/_site/assets/improvedAddChange.cpuprofile.txt)


[last post]:{{site.baseurl}}{% post_url 2019-12-12-VsCodeVim %}
[issue]:https://github.com/VSCodeVim/Vim/issues/3920
[reported]: https://github.com/VSCodeVim/Vim/issues/2216