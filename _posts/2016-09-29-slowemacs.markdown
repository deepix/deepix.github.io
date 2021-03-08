---
layout: post
title: Debugging emacs slowness
tags: emacs elisp debugging python
newlink: /posts/2016-09-29-slowemacs/
---

I use Emacs (emerge) while merging files.  Today, when trying to merge
some Python code, I found that it was taking exceedingly long time.
It was blank for 5 minutes and counting.

<!--more-->

Googling revealed that one can print a stack trace under such cases,
by having the following line in the .emacs file:

```
(setq debug-on-quit t)
```

I did that, restarted Emacs, pressed C-g (Ctrl-g).  The stack showed:

```
Debugger entered--Lisp error: (quit)
  syntax-ppss()
  python-syntax-context(paren)
  python-info-line-ends-backslash-p()
  python-nav-end-of-statement()
  python-nav-end-of-defun()
  byte-code("\306\307!\203\250^@\310 ^X\311\216 \203^V^@\312    W\205^^@\n\2033^@\212\313^R^Kb\210\314 \210\315\307!\210\f^]\316^^^W\317^M!+\206^^@^K\212\312 ^N^X\\^$
  python-info-current-defun()
  python-imenu-prev-index-position()
  imenu-default-create-index-function()
  byte-code("^H\205\"^@ \n\235?\205\"^@^K\203^[^@\f^KW\204^[^@^K\306U\205\"^@\212^M )\211^V^G\207" [which-func-mode major-mode which-func-non-auto-modes which-func-m$
  which-func-ff-hook()
  run-hooks(find-file-hook)
  after-find-file(nil t)
  find-file-noselect-1(#<buffer ServiceDB.py> "file.py" nil nil "file2.py"$
  find-file-noselect("file.py")
  ediff-find-file(file-B buf-B ediff-last-dir-B startup-hooks)
  ediff-files-internal("file.py" "file2.py"$
  ediff-merge-with-ancestor("file.py" "file2.py$
  (let ((base (car (prog1 command-line-args-left (setq command-line-args-left (cdr command-line-args-left))))) (theirs (car (prog1 command-line-args-left (setq comma$
  command-line-merge("-merge")
  command-line-1(("-merge" "file.py" "file2.py$
  command-line()
  normal-top-level()
```

So it was stuck parsing the Python file.  This was a big file (144 KB in size), but still, an inordinate amount of time to parse.  The interesting line is `which-func-ff-hook()`.  This comes because I have which-func-mode enabled on my Emacs.  This shows which function the cursor is in in the status line:

```
(which-func-mode t)
```

I commented that line out, restarted Emacs, and of course, it opened the file in a jiffy!
