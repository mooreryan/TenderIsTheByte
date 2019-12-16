---
layout: post
title: "Beginning Bioinformatics:  What's a terminal?  What's the command line?"
description: If you're new to bioinformatics and just getting started, you're going to need to learn a lot of background info before you can really get started analyzing your data.  In this post, we go back to the basics talking about terminals and the command line.
categories: blog
image: "/assets/img/posts/command_line_terminal/find-cli.png"
twitter_share: https://ctt.ac/3xrBO
---

Earlier today, I was working on a blog post about how to install command line programs for bioinfomatics beginners.  As I was going through it, I realized that there is a lot of background information you need to know just to install a typical bioinformatics program!  Here are some examples:   What's the command line?  What's a terminal?  How to I change directories?  What is an archive file?  What does the `.tar.gz` thing at the end of the filename mean and how do I open this kind of file?  What do all these installation instructions keep telling me to type `make`?  Anyway, you get the point!  That's a lot of prerequisite knowledge just to get started with *installing* bioinformatics software.

But what if you're a beginner?  I would guess that if you're a true beginner to bioinformatics, you probably don't know the answer to most those questions.  And that's okay!  Everyone is a beginner at some point.  You just need things explained at a basic level and lots of practice!

Let's start by talking about terminals and the command line.

## Graphical vs. command line interfaces

You're probably reading this blog in a web-browser.  Whether it's on a phone or on a laptop, your web-browser is a [graphical user interface](https://en.wikipedia.org/wiki/Graphical_user_interface) (GUI).  We interact with GUIs by clicking around with the mouse, or if we're using a mobile device, by tapping and swiping on the screen.  Most of the programs we use on our phones and computers are GUIs.  Finder on a Mac and Windows Explorer (or File Explorer) on a PC are GUIs that let you browse and manage files on your computer.  Chrome, Safari, and Internet Explorer are GUIs for browsing the web.  So a GUI is a program with a graphical user interface, and make up the majority of the programs you probably use on a daily basis.  

{% include post_img_border.html path='command_line_terminal/firefox-gui.png' caption='Firefox is a GUI for web browsing' %}

Compare this to programs with so-called [command-line interfaces](https://en.wikipedia.org/wiki/Command-line_interface) (CLI).  Rather than pointing and clicking, you interact with these programs by typing things at the command line, generally through a [terminal](https://askubuntu.com/questions/38162/what-is-a-terminal-and-how-do-i-open-and-use-it).  One example a program with a command-line interface is [find](https://en.wikipedia.org/wiki/Find_(Unix)), which *find*s files based on some user-specified criteria.  Most bioinformatics programs don't have graphical user interfaces.  If you want to learn to do bioinformatics, you're almost certainly going to have to get comfortable with the command line.

{% include post_img_border.html path='command_line_terminal/find-cli.png' caption='The find program\'s CLI' %}

Some programs have both a graphical and a command line interface, like [Cytoscape](https://manual.cytoscape.org/en/3.5.0/Programmatic_Access_to_Cytoscape_Features_Scripting.html), a program for visualizing networks.  Why have both?  Well, there are some tasks that are easier to accomplish using a graphical user interface and some that are easier with a command line interface.  For example, if you need to explore your network--color it, change the size of nodes and edges, make it look nice and pretty--you're probably going to want to use the Cytoscape GUI for that.  If you have an algorithm or process that you want to apply to hundreds of networks, then you're definitely going to want to use the command line interface instead.

## The terminal and the command line

A terminal is a text-based interface to your computer.  Depending on who you're talking to, you might hear the terminal called a couple of different things.  The console, the shell, the command prompt--whatever they call it, people are generally talking about the same thing:  a place where you enter commands and interact with command-line programs.  Of course, all of these terms have [more precise definitions](https://askubuntu.com/questions/506510/what-is-the-difference-between-terminal-console-shell-and-command-line), (just ask a [systems admin](https://en.wikipedia.org/wiki/System_administrator)!).  For now though, let's just agree to call it the terminal and not worry too much about it.

As I mentioned earlier, you control a program with a command line interface by typing commands into a terminal.  Here is an example of how you might use the find command:

{% highlight bash %}
{% raw %}
find . -name '*.txt'
{% endraw %}
{% endhighlight %}

Let's talk about this just a bit.  The first thing there is the word `find`.  `find` is the name of the command we're running.  *(If you're reading program documentation or a blog and you see a word in font that looks `like this`, then it generally means it's either a command, something you're typing at the terminal, or some snippet of code.)*  Next are the [arguments](https://www.computerhope.com/jargon/a/argument.htm) that we pass to the `find` command/program.  Arguments let us modify the behavior of a command or program.  In this case, the `.` tells `find` look in the current directory, and `-name '*.txt'` bit tells `find` to look for files that end with `.txt`.  Don't worry too much if that doesn't make sense right now.  We'll get into the details of actually running command line programs in a different post.  For now, just know that command line programs are those that you control by typing commands and arguments into the termial.

Let me just mention one more thing.  If you're reading program documentation or tutorials about the command line, you might see commands that look like they start with a `$` character like this:

{% highlight bash %}
{% raw %}
$ find . -name '*.txt'
{% endraw %}
{% endhighlight %}

The `$` character isn't actually part of the command.  Some authors will put it in front of the actual command to represent the command prompt (the place where you're actually typing in the terminal).  It's just there to make it clearer that what you see is a command that you should type into a terminal.

## How do I get a terminal?

If you're on a Mac, you should have a program called `Terminal` already installed.  To open it, click on the Launchpad and type `Terminal` into the search box and double click on its icon.  [iTerm2](https://iterm2.com) is another popular [terminal emulator](https://en.wikipedia.org/wiki/Terminal_emulator) for Macs.  If you're using [Linux](https://opensource.com/resources/linux), then you've got [tons of options](https://www.tecmint.com/linux-terminal-emulators/) for terminals as well.  Windows is a bit different from the other two, but it does have a terminal.  Check out [this software repository](https://github.com/microsoft/terminal) and [this guide](https://www.lifewire.com/command-prompt-2625840) for more information on the Windows command prompt.  I personally don't use a PC for work, but many people I know who do use PCs for bioinformatics use [Cygwin](https://www.cygwin.com), which let's you get a more Linux-y command line experience on your PC.

## Wrap up

In this post, we talked about graphical user interfaces versus command line interfaces, what is a terminal, what is the command line and how to actually get a terminal for your computer.  This is all foundational stuff that you'll be getting a lot more experience with as you learn more about bioinformatics.  Hopefully, this guide helps clear up any confusion you may have had!  

*If you want some more hands-on info about command line basics, check out [this nice tutorial](https://tutorial.djangogirls.org/en/intro_to_command_line/) from Django Girls.*