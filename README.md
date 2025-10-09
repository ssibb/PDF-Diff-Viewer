# PDF-Diff-Viewer

PDF Diff Viewer, a side-by-side, visual highlight, sync-scroll, PDF comparer, written in Python. Open source, mostly powered by PyMuPDF and Tkinter. Optional support for git diff, for a better comparison algorithm.

![screenshot](/screenshot.gif)

#### Install & Run

Windows binaries are provided; while no installation is needed, you need to decompress everything and then run "pdf_viewer_app.exe" within the folder "pdf_viewer_app". Make sure you have writing permission for the folder where you place the app, since some features require the usage of temporary files (git diff; comparison of Word files or from the clipboard).

However, if you prefer running directly the script, first you need to install the libraries as follows:


```bash
pip install pymupdf Pillow klembord tkinterdnd2 pywin32 pyautogui
```


Then, just download the script and run on Python. 

```bash
python pdf_viewer_app.py
```

Tested on Python 3.12 on Windows. Should work on Linux as well, though untested till now; possibly with small changes.

For better comparison uses git diff when available; the binary release for Windows already includes the git diff binaries (taken from git-for-windows, the PortableGit release. If git diff command is not available, uses Python built-in difflib. (Still unsure if this works also with a generic git diff installation; I think colors of moves can be customized in git diff; if so, it will probably broke the moves logic within the script).

##### Mac

For Mac some extra steps might be necessary to get 'tcl-tk' working properly
```bash
brew install tcl-tk@8
export TCL_LIBRARY=$(brew --prefix tcl-tk@8)/lib/tcl8.6
export TK_LIBRARY=$(brew --prefix tcl-tk@8)/lib/tk8.6
```

If you get the error `_tkinter.TclError: Can't find a usable init.tcl in the following directories:`, make sure to re-run the exports above.

If you get errors like the following:
```terminaloutput
/opt/homebrew/opt/tcl-tk/lib/tk8.6/tk.tcl: version conflict for package "Tk": have 8.6.12, need exactly 8.6.17
version conflict for package "Tk": have 8.6.12, need exactly 8.6.17
```

You might need to run the following commands:
```bash
sed -i '' 's/package require -exact Tcl 8\.6\.17/package require -exact Tcl 8.6.12/' $(brew --prefix tcl-tk@8)/lib/tcl8.6/init.tcl
sed -i '' 's/package require -exact Tk  8\.6\.17/package require -exact  Tk 8.6.12/' $(brew --prefix tcl-tk@8)/lib/tk8.6/tk.tcl
```

After fixing any Tcl/Tk issues, you can run the project using [uv](https://docs.astral.sh/uv/)
```bash
uv sync
uv run pdf_viewer_app.py
```

#### Features



* Side-by-side compare, with sync scroll (sync scroll is based on the first word shown in the top left corner of the panel that is being scrolled)
* Word-based compare, useful for comparing generic text from documents. This is opposed to line-based compare, which is widely available and most useful for tracking changes on source code
* Differences are highlighted in RED (left pane) and GREEN (right pane)
* Supports moves (only when git diff is available). A move is just an insertion paired by an equal deletion. Sometimes moves might be more informative than just the corresponding raw deletions and insertions
* For documents with dark background, right click > toggle dark mode (changes the blend mode of the highlight)
* Works with PDF, but (using Microsoft Word, when installed) can automatically print to PDF .docx and .rtf documents
* Accept HTML text from clipboard (right click > paste). Plain text is supported as well
* Comparison can ignore case changes, quotes type (useful when comparing OCR documents where you don't care whether it's " or ”), and "f" ligatures (a strange feature that substitutes two or more characters with a similar looking one; see [https://en.wikipedia.org/wiki/Ligature\_(writing)#Ligatures\_in\_Unicode\_(Latin\_alphabets)](Wikipedia) for a more comprehensive discussion)
* quick jump to next and previous change (note that it's a "screen based" next or previous change; meaning it will take you to the next/previous change that is not currently shown at screen)
* Double click for enable/disable the one-finger smooth scroll. This is a workaround for Tkinter that doesn't support the two-finger scrolling gesture. When enabled, you will be able to vertically scroll (horizontal is disabled) just moving the cursor with a single finger. The cursor will then snap back to its starting position, allowing for further scrolling. Another solution would be to rewrite the script using kivy, but it's not on my plans
* Supports drag and drop
* Files can be loaded also through command line (you can pass either one or two files; if two files are provided, they are automatically compared)





#### Things to know



Word based comparison means just that. Say, you have this text:



| Questions             | YES  |  NO |
| --------------------- | ---- | --- |
| Is this a question?   |  X   |     |



and you compare to this one:





| Questions             | YES  |  NO |
| --------------------- | ---- | --- |
| Is this a question?   |      |  X  |



..and, guess what? You won't find any difference. Why? Because words are the same, and are in the same order. It's just the X that has changed its location.



This is done on purpose: imagine a word that, between the two versions, changes its location because it goes to the next line, and here you want to see it as unchanged. Therefore, its a feature, not a bug. Changes in the font (color, formatting, etc.) is ignored in the same way. In other words, changes everything else that is not text (say, images, shapes, etc.) are ignored.

Anyway, you should be aware of this behaviour, because it might not always be what you expect.

Given this script only compares the text within the PDFs, files have to contain text. Scanned PDF needs to be OCRed first (I have no plan to implement OCR, though PyMuPDF supports OCR through Tesseract). I also noted that some PDF generated by printing from browsers might contain "fake" text (meaning that test is actually rendered by shapes that looks like text, but are not text). For being able to compare these PDFs with this script you first need to OCR them as well.







#### Similar software



I'm not aware of any other open source solution for side-by-side, sync-scroll PDF comparison. Anyway, there are other solutions:



* [https://draftable.com/compare] has a nice free web-based comparison tool, as well as paid desktop software (I tried the free web-based, and it's pretty good, but might not be well suited for sensible documents you wouldn't like to upload)
* [https://www.textcompare.org/pdf/] another web-based solution that seems to process the comparison directly in browser (i.e.: no upload of the documents). Local processing is a plus, but results to me don't look as good as with Draftable.
* [https://www.pdf-xchange.com/knowledgebase/324-How-do-I-compare-documents-in-PDF-XChange-Editor] PDF-Xchange-Editor (which is mostly a PDF editor) has recently introduced this feature. 





#### The idea and the code



"I" created this because I couln't find any good open source solution for text comparison at word level. I mean, I know GNU wdiff, but I couldn't find nothing GUI-based and ready to use. See for example WinMerge; it may accept PDF (though retaining only plain text), but the comparison is still line-based, which doesn't make any sense if you are trying to compare text in natural language.

If no text comparison tool were available, PDF comparison seemed out of question. This despite the comparison seems to me a very basic software. So, I decided to ask Gemini 2.5 flash. I can fairly say that my aim was to obtain something useful as much as I want to experimentally see what were the real capabilities of LLM as of today (June-July 2025). And I am pretty impressed.

I guided Gemini with subsequent requests (say, the first request was to create a GUI a simple PDF viewer in python). Then, I prompted Gemini to add features (can't you add zoom? What about binding arrows on keyboard for scrolling?). When something was going wrong I gave back to Gemini the traceback of the error. Sometimes, code was without syntax errors, but still not working as expected; in these cases, I nudged Gemini to review the code adding print(), repr(), dir() as needed for generating output on the console with useful information for tracking where the code was not working as expected. In my opinion, this is the most interesting fact, as this is a very human-like way to debug code, and often this was enough for having Gemini to fix its own code. I said "often" because it did not solve everything (for example, I had fairly early a version of sync scroll, with a nasty bug that made a slow crawling of the panel upwards). I had to put effort in solving this, observing that the fixes suggested by Gemini were not working, and going on asking Gemini to implement a "debounce" logic (I didn't even know that what I described was named "debounce", Gemini told me, even if it didn't come to the solution by itself). Anyway, I had Gemini to generate a thousand or so of lines, with a somewhat working script. Then, I realized it wasn't "safe" anymore to feed Gemini with the entire script and asking to implement changes, as I observed some, *how can I call them?*, mutations. I observed Gemini to change irrelevant parts of the script (i.e.: rewriting the comments), but also merging two functions, close together, in one single function (i.e.: removing a def function() line), with could lead to catastrophic failure. From that moment on, I kept debugging the code as I found bugs in real life using the script. I also implemented new features with Gemini (for example, the one finger smooth scroll), but this time asking Gemini to create a demo of the function and then manually importing it in the main code.

At work, I have access to Copilot (which is based on ChatGPT) and I can say that its coding capabilities are similar to what Gemini could do probably 8 months ago. Meaning that Copilot would have never been able to produce that thing, and it mostly doesn't show any interest in interactive debugging. If I can extrapolate the trend, I think in a couple of years we should be able to just obtain something similar to this script just asking a LLM to create the whole software. While this is fascinating, it also raises some concerns. Gemini is quite stubborn and, unless cornered, it is VERY confident in being right. Speaking about coding, I'm not sure how we are going to deal with debugging. Anyway, this is definitely impressive. I managed to create this (I take ownership of the result mostly because of the debugging!!) in, say, 30 hours. Five years ago, if I were to create a similar program, it would have probably required me a month, because I would have had to study first, then to code, and lastly to debug. Coding this way was pretty similar to managing a brilliant, though young, collegue, having him to perform the menial work, while keeping control of the project.
