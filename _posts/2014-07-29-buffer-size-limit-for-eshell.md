---
layout: post
title: "Automatically cleanup the buffer for Eshell"
description: ""
category:
tags: [Emacs, shell]
---
{% include JB/setup %}

Keep writing some simple thing in English, for I will have less chance for writing English words in daily working.

I will always run eshell for shell tasks, because this is really like the normal buffer in Emacs, so all the command for Emacs will keep working for this buffer. This is convenient for some actions.

The problem annoying me is that if the size of buffer for eshell is too big, Emacs will gets more and more slow. Emacs essentially is a sole process program. So I have some digg and written a trivial elisp code like this solved the problem.

{% highlight ruby %}
(defun clear-and-send-input()
  (interactive)
  (if (> (count-lines 1 (point)) 800)
      (let ((inhibit-read-only t))
        (message "Clear the eshell now !")
	(erase-buffer)))
  (eshell-send-input))

(add-hook 'eshell-mode-hook
	  (lambda ()
	  (local-set-key (kbd "<return>") 'clear-and-send-input)))

{% endhighlight %}

clear-and-send-input is a wrapper for eshell-send-input, I set the maximal number of eshell buffer to 800, and I bind this function to <return>, so every time if the buffer size is too big, this wrapper will automatically clean up the buffer.

And yesterday I found this article [Mastering Emacs in one year guide](https://github.com/redguardtoo/mastering-emacs-in-one-year-guide)is really thought-provoking, Hope this may help you.
