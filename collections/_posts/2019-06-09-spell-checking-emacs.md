---
layout: post
title: "Spell Checking in Emacs"
author: ryan
description: Super simple guide to setting up spell checking in Emacs.
categories: blog
image: "/assets/img/posts/spell_checking_emacs/emacs_flyspell_example_100.png"
twitter_share: https://ctt.ac/68wZB
last_updated: 2021-05-20
---

I usually use [Overleaf](https://www.overleaf.com) for working on LaTeX documents, but just to mix things up, I decided to use [Emacs](https://www.gnu.org/software/emacs/) instead.  As I was happily typing away, I realized that I was making a ton of spelling errors.  "This would be so much better with a spell checker.  Should I use [TextEdit](https://support.apple.com/guide/textedit/welcome/mac) instead?".  Of course not...this is Emacs we're talking about!  It can be [pretty much](http://ergoemacs.org/emacs/emacs_fun.html) [whatever you want it to be](https://www.gnu.org/software/emacs/tour/).  It turns out that spell checking in Emacs was only a quick Google search away:  [Flyspell](https://www.emacswiki.org/emacs/FlySpell) is part of Emacs and provides on-the-Flyspell (get it?) checking through a [minor mode](https://www.gnu.org/software/emacs/manual/html_node/emacs/Minor-Modes.html).  Let's set it up!

## Setting up Flyspell

Setting up Flyspell is really easy.  Just add the following to your `.emacs` file (or wherever you keep your [Emacs initialization file](https://www.gnu.org/software/emacs/manual/html_node/emacs/Init-File.html)):

{% highlight elisp %}
{% raw %}
(dolist (hook '(text-mode-hook))
  (add-hook hook (lambda () (flyspell-mode 1))))
{% endraw %}
{% endhighlight %}

This will enable `flyspell-mode` for `text-mode` and any modes that are derived from `text-mode` such as `log-edit-mode` and `change-log-mode`.  If you don't want spell checking on certain modes derived from `text-mode` you can disable them.  For example, if we wanted to disable `flyspell-mode` for the two modes mentioned above, we could add this to our `.emacs` file:

{% highlight elisp %}
{% raw %}
(dolist (hook '(change-log-mode-hook log-edit-mode-hook))
  (add-hook hook (lambda () (flyspell-mode -1))))
{% endraw %}
{% endhighlight %}

If you're using a Mac, you may need to [add the following Elisp code](https://joelkuiper.eu/spellcheck_emacs) to your config file as well in order for Flyspell to pick up the two-finger clicks (right-clicks):

{% highlight elisp %}
{% raw %}
(eval-after-load "flyspell"
  '(progn
     (define-key flyspell-mouse-map [down-mouse-3] #'flyspell-correct-word)
     (define-key flyspell-mouse-map [mouse-3] #'undefined)))
{% endraw %}
{% endhighlight %}

Once you're finished editing your init file, don't forget to run `M-x eval-buffer` or to restart Emacs so that the changes take effect.

## Using Flyspell

Once you've made the above changes to your Emacs config file, using Flyspell is as easy as opening up a file for which Emacs uses `text-mode`.  Depending on which spell checking software is installed on your system, when opening a new text file, you should see a message at the bottom of the Emacs window about starting a new process with the default dictionary.  Something like this:

{% include post_img.html path='spell_checking_emacs/new_ispell_process.png' caption='Starting new Ispell process...' %}

If you try and open a text file and instead get an error,

> Error enabling Flyspell mode:
> (Searching for program no such file or directory ispell)

then you need to install a spell checking program on your computer.  There are many options, but Flyspell uses [Ispell](https://www.gnu.org/software/ispell/) by default, so I went with that one.  It's available in most package managers:  on a Mac you run `brew install ispell` and on Linux, `apt-get install ispell` or something similar will work.  Once Ispell is installed, restart Emacs and you should be good to go.

Working with Flyspell is easy as well.  Just type away and Flyspell will check your spelling on the fly.  Incorrectly spelled words will get a nice little red squiggly line underneath them.  To correct words or to add words to your personal dictionary, just right-click (or two-finger-click on a Mac) on the word you want to fix and an option box will pop up.  There, you can either pick from one of the suggested spellings or add the word to the dictionary.  Simple, right?  Here is a little gif showing all that in action:

{% include post_img.html path='spell_checking_emacs/emacs_flyspell_example.gif' caption='Spell checking!  In Emacs!' %}

### Removing words from the dictionary

What happens if you add a word to the dictionary on accident and want to remove it?  That's easy as well.  Checking `man ispell`, you will see a section called `FILES` that lists all the important files Ispell uses for its spell checking.  My default dictionary is English, so my personal Ispell dictionary can be found here: `$HOME/.ispell_english`.  To remove a word from the dictionary, just open that file in your favorite text editor (Emacs, obviously!) and remove the word.  For it to take effect, you will likely have to close and restart Emacs.

### Running Flyspell on any type of buffer

One last thing to mention is that if you want to run Flyspell on a buffer that isn't currently using `flyspell-mode` you can run `M-x flyspell-buffer` to check the spelling of the current buffer.  This can be useful if you want to do a quick check of some source code or something similar.

### Switching dictionaries

If you want to use a different dictionary, for example, you sometimes write in English but other times in Spanish, you can use the `ispell-change-dictionary` function.  The basic way to do it is to run `M-x ispell-change-dictionary`, then hit return, then type the name of the dictionary that you want to use.  For example: `M-x ispell-change-dictionary RET castellano` to change to the Spanish dictionary, or `M-x ispell-change-dictionary RET default` to switch back to the default dictionary.  After switching dictionaries, you will need to rerun Flyspell on the current buffer: `M-x flyspell-buffer`.

If you find yourself often needing to switch between dictionaries, you can write some custom elisp ([Emacs lisp](https://www.gnu.org/software/emacs/manual/html_node/elisp/)) functions and stick them in your `.emacs` file (or wherever you keep your custom config file).

Here is an example of that.

{% highlight elisp %}
{% raw %}
(defun flyspell-spanish ()
  (interactive)
  (ispell-change-dictionary "castellano")
  (flyspell-buffer))

(defun flyspell-english ()
  (interactive)
  (ispell-change-dictionary "default")
  (flyspell-buffer))
{% endraw %}
{% endhighlight %}

*Note that my default dictionary is English, but you can switch `default` to whatever else you would like.*

Now, if you put those functions into your `.emacs` and evaluate the buffer, you can use them to switch back and forth between dictionaries and rerun the spell checking in the current buffer in one command like this: `M-x flyspell-spanish` switches to the Spanish dictionary, and `M-x flyspell-english` switches back to the English dictionary.

## Wrap up

And that's it!  Of course, there is a lot more you can customize, like shortcuts, using different spell checking programs in place of Ispell, and even setting up spell checking for comments in source code!  See [the Flyspell entry in the Emacs wiki](https://www.emacswiki.org/emacs/FlySpell) for more information.
