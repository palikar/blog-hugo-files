+++
title = "Emacs windows resizing"
author = ["Stanislav Arnaudov"]
description = "A short walkthrough of a Emacs package for windows resizing that I recently wrote."
date = 2018-06-29T00:00:00+02:00
keywords = ["emacs", "frames", "windows", "resizing", "buffers"]
lastmod = 2019-10-20T20:05:38+02:00
categories = ["emacs"]
draft = false
weight = 100
+++

## Abstract {#abstract}

Recently I've been introduced to the concept of a [tiling windows manager](https://en.wikipedia.org/wiki/Tiling%5Fwindow%5Fmanager). One key feature that caught my attention is how you can quickly resize the different windows and create the desired windows configuration with just a few executions of some keybindings. That got me to wonder how it would cool if I also have that in Emacs. I googled something like "resize emacs windows" but didn't really (I didn't want really) find a package that can do that. So, of course, I used this as an excuse to write a very simple package that would help me achieve what I want - quickly resizing Emacs' windows - and in the process would teach me the same new things about [Emacs Lisp programming](https://en.wikipedia.org/wiki/Emacs%5FLisp). <br /> <br /> Oh yeah, and by the way, I created a whole minor Emacs mode for the job. Yes, maybe it is a little bit of an overkill to create a whole mode for this but hey, new knowledge about Emacs and Emacs Lisp never hurts now, does it.


## The existing way {#the-existing-way}

Before you jump at me, screaming that such functionality already exists in Emacs and theirs no need for a whole new minor mode - yes, I know. Or... I found eventually... after I've already implemented my thing. Again, Emacs Lisp experience - **always good**! <br /> <br /> If you want to take the easy road, several nice functions already exist.

-   `enlarge-window` grows your current window vertically and it's bound to `C-x ^` by default.
-   `enlarge-window-horizontally` does the same thing but horizontally (shocking, I know) with default keybinding `C-x }`

There are also the functions `shrink-window` and `shrink-window-horizontally` for making the window smaller in the desired dimension. The latter one is bound to `C-x {` and the former is surprisingly not bound by default. You can find more about resizing [at the official wiki page](https://www.emacswiki.org/emacs/WindowResize). <br /> <br /> With all that and quickly setting up some simple keybindings, you can quickly create your desired windows configuration from the comfort of your keyboard. Maybe something like this: ![](/ox-hugo/emacs_windows-config.png)


## The "my way" {#the-my-way}

So, if you are interested in Emacs programming, you can have this section as a gentle introduction to mode writing.

<br />


### Mode definition {#mode-definition}

We start by defining a couple of things that we will need for our new minor mode whose definition comes in right after. Each mode deserves its own [group](https://www.gnu.org/software/emacs/manual/html%5Fnode/elisp/Group-Definitions.html). You know, the name of you give while you call `customize-group` when you want to configure some new package that you've downloaded. Our group will be used to house the one [custom](https://www.gnu.org/software/emacs/manual/html%5Fnode/emacs/Easy-Customization.html) that we have. The group definition is really easy with the macro `defgroup`. It's basically a one liner:

```lisp
(defgroup framer nil "Custom variables for framer-mode")
```

"framer" is the name, the second argument is a list of the customs that are in the group but it's more convenient to define them later by specifying the group that each custom belongs to. The third argument is a string that will be displayed near the top while customizing the package's customs. <br /> <br /> _Note:_ Customs are the configurable options of a given package in Emacs. Emacs makes it easy to write such customizable variables that later can be configured by the user of your package. <br /> <br /> After the group, we need a [keymap](https://www.gnu.org/software/emacs/manual/html%5Fnode/elisp/Keymap-Basics.html#Keymap-Basics) for our mode. In it, we'll define all the keybindings that will be activated when the mode is active. The definition is again relatively straight forward:

```lisp
(defconst framer-mode-map
  (let
      (
       (map (make-keymap))
       )
    (define-key map (kbd "S-<up>") 'framer-increase-height)
    (define-key map (kbd "S-<down>") 'framer-decrease-height)
    (define-key map (kbd "S-<right>") 'framer-increase-width)
    (define-key map (kbd "S-<left>") 'framer-decrease-width)
    map)
  "Keymap for Framer minor mode.")
```

The map is defined in a variable named `framer-mode-map`. The actual keymap creation is done with the function `make-keymap`. After that, in the body of the `let`-block we define the individual keybinding in the keymap and which functions they call. We haven't defined the functions (`framer-increase-height`, ...) yet but don't worry. <br /> <br /> At this point we are ready to define our mode. As many other packages, we'll actually define two modes:

-   one global that will take effect in the whole "Emacs process"
-   and one that will only affect the current buffer.

The actual definition is pretty easy and is done through the macro `define-minor-mode`. The definitions of our two modes:

```lisp
(define-minor-mode framer-mode
  "Mode that enables manual control over emacs frames' sizes."
  :lighter " Framer"
  :keymap framer-mode-map
  :group 'framer)

(define-minor-mode global-framer-mode
  "Mode that enables manual control over emacs frames' sizes."
  :lighter " Framer"
  :keymap framer-mode-map
  :global t
  :group 'framer)
```

Here we use the already defined group and keymap and we pass them to the appropriate key-word attributes - `group` and `keymap` respectively. What is pass to `:lighter` is the thing that will be displayed in the modeline while the mode is active. Be sure to have that leading space or the text of your mode will be "glued" to the text of the previous mode in the modeline. `:global` indicates of the mode is global or not (yes, I bet you needed that explanation). <br /> <br /> For more information on how to write modes for Emacs, check out [this](https://www.gnu.org/software/emacs/manual/html%5Fnode/elisp/Defining-Minor-Modes.html) page from the official documentation. <br /> <br /> Ok, one last thing before we actually define our functions for resizing - we'll create one custom that will indicate how big is the resizing gap with which we'll be changing the size of the windows. The name of it will be appropriate `resizing-step`, it'll have a default value of 50 and it will be an integer.

```lisp
(defcustom resizing-step 50
  "The amount with which the dimension of the current windows will be decreased/increased."
  :type 'integer
  :group 'framer)
```

The docstring, in the beginning, is what will be displayed to the user in the customization buffer near the name of the custom. <br /> <br /> And we that we are ready with me minor mode boilerplate. Now let's get to the actual problem.


### Core functions definition {#core-functions-definition}

There are a couple of handy functions in Emacs that make the resizing of windows easy. We'll use them and make them a tiny bit smarter. Those functions are `window-resizable` and `window-resize` (documentation [here](https://www.gnu.org/software/emacs/manual/html%5Fnode/elisp/Resizing-Windows.html)). The first one checks if the resizing is possible and the second one does the actual resizing. I figured that it is a good idea always to check if the resizing is allowed before calling `windows-resize`. So, resizing in width will look like:

```lisp
(if (window-resizable nil resizing-step t nil t)
        (window-resize nil resizing-step t nil t))
```

The first argument is the windows to be resized - if `nil`, the current windows will be considered. The second argument is obvious, the third whether the resizing is vertical or horizontal - `t` for horizontal, `nil` for vertical. The last `t` indicates that the given amount is in pixels. <br /> <br /> As said, we want to make the resizing smart and intuitive while doing it with arrow keys. This means that we with the same arrow must either shrink or grow the window depending on whether it is on the top or the bottom of other windows. Put simply - we want the size of the window to change in the direction we are pointing with the pressed arrow. So, the question becomes, how do we figure out where is the selected window. <br /> In a script from [Mathias Dahl](https://www.emacswiki.org/emacs/MathiasDahl) I've found those two nifty convenience functions that can tell you where the current window is located with respect to the other ones. For example, whether the window is on the left, right or in the middle, between two other windows. The functions are:

```lisp
(defun win-resize-left-or-right ()
  "Figure out if the current window is to the left, right or in the middle."
  (let* ((win-edges (window-edges))
     (this-window-x-min (nth 0 win-edges))
     (this-window-x-max (nth 2 win-edges))
     (fr-width (frame-width)))
    (cond
     ((eq 0 this-window-x-min) "left")
     ((eq (+ fr-width 2) this-window-x-max) "right")
     (t "mid"))))

(defun win-resize-top-or-bot ()
  "Figure out if the current window is on the top, bottom or in the middle."
  (let* ((win-edges (window-edges))
     (this-window-y-min (nth 1 win-edges))
     (this-window-y-max (nth 3 win-edges))
     (fr-height (frame-height)))
    (cond
     ((eq 0 this-window-y-min) "top")
     ((eq (- fr-height 1) this-window-y-max) "bot")
     (t "mid"))))
```

With `win-resize-left-or-right` the implementations of `framer-decrease-width` and `framer-increase-width` becomes clear. For decreasing the width of the window we first check where we are currently and depending on the location we grow or shrink the window. Remember our keymap. There we grow the width with the left arrow so

-   if on the right, we grow because the left arrow points in the "growing direction"
-   if on the left, we shrink for the exact opposite reason
-   if in the middle, we treat the window as it is on the right. Trust me, it makes sense if done this way.

For `framer-decrease-width` we do the exact opposite thing. There where we were shrinking, we grow and vice-versa. <br /> <br /> We perform the checks in a simple `cond` block and with that we have:

```lisp
(defun framer-increase-width ()
  "Make the current frame smaller in width."
  (interactive)
  (cond
   ((equal "right" (win-resize-left-or-right))
    (if (window-resizable nil (- resizing-step) t nil t)
        (window-resize nil (- resizing-step) t nil t)))
   ((equal "left" (win-resize-left-or-right))
    (if (window-resizable nil resizing-step t nil t)
        (window-resize nil resizing-step t nil t)))
   (t (if (window-resizable nil resizing-step t nil t)
          (window-resize nil resizing-step t nil t)))))

(defun framer-decrease-width ()
  "Make the current frame smaller in width."
  (interactive)
  (cond
   ((equal "right" (win-resize-left-or-right))
    (if (window-resizable nil resizing-step t nil t)
        (window-resize nil resizing-step t nil t)))
   ((equal "left" (win-resize-left-or-right))
    (if (window-resizable nil (- resizing-step) t nil t)
        (window-resize nil (- resizing-step) t nil t)))
   (t (if (window-resizable nil (- resizing-step) t nil t)
          (window-resize nil (- resizing-step) t nil t)))))
```

`framer-increase-height` and `framer-increase-height` are implemented more or less the same way.

```lisp

(defun framer-increase-height ()
  "Make the current frame smaller in width."
  (interactive)
  (cond
   ((equal "top" (win-resize-top-or-bot))
    (if (window-resizable nil (- resizing-step) nil nil t)
        (window-resize nil (- resizing-step) nil nil t)))
   ((equal "bot" (win-resize-top-or-bot))
    (if (window-resizable nil resizing-step nil nil t)
        (window-resize nil resizing-step nil nil t)))
   (t (if (window-resizable nil (- resizing-step) nil nil t)
          (window-resize nil (- resizing-step) nil nil t)))))

(defun framer-decrease-height ()
  "Make the current frame smaller in width."
  (interactive)
  (cond
   ((equal "top" (win-resize-top-or-bot))
    (if (window-resizable nil resizing-step nil nil t)
        (window-resize nil resizing-step nil nil t)))
   ((equal "bot" (win-resize-top-or-bot))
    (if (window-resizable nil (- resizing-step) nil nil t)
        (window-resize nil (- resizing-step) nil nil t)))
   (t (if (window-resizable nil resizing-step nil nil t)
          (window-resize nil resizing-step nil nil t)))))
```

<br /> <br /> And there you have it, reinventing the wheel in a timely wasteful manner. <span class="underline">Awesome</span>, amirite!


## References {#references}

-   [How to Make an Emacs Minor Mode](https://nullprogram.com/blog/2013/02/06/) - good starting point if you want to extend your Emacs with some custom minor modes.
