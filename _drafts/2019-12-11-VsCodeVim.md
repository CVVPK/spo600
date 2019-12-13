---
layout: post
title: VsCodeVim
date: 2019-12-11
---
First solution:

```ts
const getText = (): string => {
      const endPosition = cursorPosition[0].getDocumentEnd(this.vimState.editor);
      const midLine = Math.floor(this.vimState.editor.document.lineCount / 2);
      const quartLine = Math.floor(midLine / 2);
      const quartPosition = new Position(quartLine, 0);
      const quartPosition2 = new Position(quartLine, 1);
      const midPosition = new Position(midLine, 0);
      const midPosition2 = new Position(midLine, 1);
      const sQuartPosition = new Position(midLine + quartLine, 0);
      const sQuartPosition2 = new Position(midLine + quartLine, 1);

      const range1 = new Range(new Position(0, 0), quartPosition);
      const range2 = new Range(quartPosition2, midPosition);
      const range3 = new Range(midPosition2, sQuartPosition);
      const range4 = new Range(sQuartPosition2, endPosition);

      return (
        (this.vimState.editor &&
          this.vimState.editor.document &&
          this.vimState.editor.document.getText(new vscode.Range(range1.start, range1.stop)) +
            this.vimState.editor.document.getText(new vscode.Range(range2.start, range2.stop)) +
            this.vimState.editor.document.getText(new vscode.Range(range3.start, range3.stop)) +
            this.vimState.editor.document.getText(new vscode.Range(range4.start, range4.stop))) ||
        ''
      );
    };

    const newText = getText();
```

Break it into smaller ranges to see if it could help, it doesn't work because of js is single threaded if there were a way to make this process multithreaded then we might have a way to solve it. In this case the combined times that took to get each range of text added up to the same time that it takes to get the whole document.

Second solution:

```ts
 let newText = '';
    // if (this.vimState.currentMode === Mode.Insert) {
    const lineNum = cursorPosition[0].line;
    const charNum = cursorPosition[0].character;
    const offset = this.vimState.editor.document.offsetAt(new vscode.Position(lineNum, charNum));
    if (this.vimState.editor && this.vimState.editor.document) {
      const newLineRange = this.vimState.editor.document.lineAt(lineNum).rangeIncludingLineBreak;
      const newLine = this.vimState.editor.document.getText(newLineRange);
      const longerNewLine = charNum <= newLine.length;
      const sol = longerNewLine ? offset - charNum : offset - charNum + 1;
      // const eol = longerNewLine ? sol+newLine.length: ;
      // const possibleEol = this.oldText.indexOf('\n', sol);
      const eol = sol + newLine.length;
      const oldLine = this.oldText.substring(sol, eol + 1);
      newText = this.oldText.substring(0, sol) + newLine + this.oldText.substring(eol);
      // const oldLine = this.oldText.substring(sol, eol).split('\n', lineNum);
      // const oldTextLines = this.oldText.split('\n');
      // console.log('oldLine: ', oldLine);
      // console.log('newLine: ', newLine);

      // console.log('charNum', charNum);
      // console.log('nLength: ', newLine.length);
      // console.log('offset ', offset);
      // console.log('sol', sol);
      // console.log(this.oldText.substring(0, sol));
      // console.log('eol', eol);
      // console.log('possibleEOL ', possibleEol);
      console.log('oldText ', this.oldText);
      console.log('newText ', newText);
      console.log(newLine.length);
      console.log('-------');
      // console.log('enter? ', this.vimState.keyHistory);
      // console.log(sol, ' ', eol, ' ', offset, ' ', oldTextLines[lineNum]);
      // oldTextLines[lineNum] = newLine;
      // newText = oldTextLines.join();
    }

```

Doesn't work for the use case because it only affects one line at a time, if we perform an action that affects multiple lines then we'll need to scan each individual line for the state to be correct. Otherwise there will be a desync between the editor and the vimstate machine.

Other Options:
Multi threading: considered, electron does support multithreading through web workers. unfurtunately it cannot be implemented because vscode has webworkers disabled, which limits us.?




// Solution1
    // const getText = (): string => {
    //   const endPosition = cursorPosition[0].getDocumentEnd(this.vimState.editor);
    //   const midLine = Math.floor(this.vimState.editor.document.lineCount / 2);
    //   const quartLine = Math.floor(midLine / 2);
    //   const quartPosition = new Position(quartLine, 0);
    //   const quartPosition2 = new Position(quartLine, 1);
    //   const midPosition = new Position(midLine, 0);
    //   const midPosition2 = new Position(midLine, 1);
    //   const sQuartPosition = new Position(midLine + quartLine, 0);
    //   const sQuartPosition2 = new Position(midLine + quartLine, 1);

    //   const range1 = new Range(new Position(0, 0), quartPosition);
    //   const range2 = new Range(quartPosition2, midPosition);
    //   const range3 = new Range(midPosition2, sQuartPosition);
    //   const range4 = new Range(sQuartPosition2, endPosition);

    //   return (
    //     (this.vimState.editor &&
    //       this.vimState.editor.document &&
    //       this.vimState.editor.document.getText(new vscode.Range(range1.start, range1.stop)) +
    //         this.vimState.editor.document.getText(new vscode.Range(range2.start, range2.stop)) +
    //         this.vimState.editor.document.getText(new vscode.Range(range3.start, range3.stop)) +
    //         this.vimState.editor.document.getText(new vscode.Range(range4.start, range4.stop))) ||
    //     ''
    //   );
    // };

    // const newText = getText();
    // console.log('oldImp');

    // console.log(cursorPosition);
    // cursorPosition.
    // const newText = this.oldText;
    // let changeEvents: vscode.TextDocumentChangeEvent;
    // vscode.workspace.onDidChangeTextDocument(e => console.log(e));
    // console.log(this.vimState.editor);

    // Solution2:
    // if (this.vimState.currentMode === Mode.Insert) {
    // const lineNum = cursorPosition[0].line;
    // const charNum = cursorPosition[0].character;
    // const offset = this.vimState.editor.document.offsetAt(new vscode.Position(lineNum, charNum));
    // if (this.vimState.editor && this.vimState.editor.document) {
    //   const newLineRange = this.vimState.editor.document.lineAt(lineNum).rangeIncludingLineBreak;
    //   const newLine = this.vimState.editor.document.getText(newLineRange);
    //   const longerNewLine = charNum <= newLine.length;
    //   const sol = longerNewLine ? offset - charNum : offset - charNum + 1;
    // const eol = longerNewLine ? sol+newLine.length: ;
    // const possibleEol = this.oldText.indexOf('\n', sol);
    // const eol = sol + newLine.length;
    // const oldLine = this.oldText.substring(sol, eol + 1);
    // newText = this.oldText.substring(0, sol) + newLine + this.oldText.substring(eol);
    // const oldLine = this.oldText.substring(sol, eol).split('\n', lineNum);
    // const oldTextLines = this.oldText.split('\n');
    // console.log('oldLine: ', oldLine);
    // console.log('newLine: ', newLine);

    // console.log('charNum', charNum);
    // console.log('nLength: ', newLine.length);
    // console.log('offset ', offset);
    // console.log('sol', sol);
    // console.log(this.oldText.substring(0, sol));
    // console.log('eol', eol);
    // console.log('possibleEOL ', possibleEol);
    // console.log('oldText ', this.oldText);
    // console.log('newText ', newText);
    // console.log(newLine.length);
    // console.log('-------');
    // console.log('enter? ', this.vimState.keyHistory);
    // console.log(sol, ' ', eol, ' ', offset, ' ', oldTextLines[lineNum]);
    // oldTextLines[lineNum] = newLine;
    // newText = oldTextLines.join();
    // }
    // }
    // const newText = this.oldText;

    // Solution3:
    // if (this.vimState.currentMode === Mode.Insert) {
    // return;
    // }
    
Promise:

    ```ts
const documentEnd = cursorPosition[0].getDocumentEnd();
    const eof = this.vimState.editor.document.offsetAt(
      documentEnd
      // new Position(documentEnd.line, documentEnd.character)
    );
    const maxBits = Math.ceil(eof / 15);
    const gettingText = async (range: vscode.Range) => {
      return (
        (this.vimState.editor &&
          this.vimState.editor.document &&
          this.vimState.editor.document.getText(range)) ||
        ''
      );
    };

    const getText = Array.from({ length: Math.ceil(eof / maxBits) }, (v, i) => {
      const startOffset = i * maxBits;
      const endOffset = (i + 1) * maxBits > eof ? eof : (i + 1) * maxBits;
      const startPosition = this.vimState.editor.document.positionAt(startOffset);
      const endPosition = this.vimState.editor.document.positionAt(endOffset);

      const range = new vscode.Range(startPosition, endPosition);

      return gettingText(range);
    });

    Promise.all(getText)
    .then(t => {
    const newText = t.join('');
....


    }).catch(error => {
    this._logger.warn('Did not save last history step');
    });
  }
    ```