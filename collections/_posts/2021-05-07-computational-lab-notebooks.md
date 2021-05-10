---
layout: post
title: "Computational lab notebooks using git and git-annex"
author: ryan
description: Managing a computational lab notebook is tricky.  Here I discuss a workflow and command line app for helping you to set up and manage your lab notebook with git and git-annex.
categories: blog
image: "/assets/img/posts/computational_lab_notebooks/git_log.png"
twitter_share: https://ctt.ac/fcl_L
last_updated: 2021-05-10
---

_Disclaimer: if you need a lab notebook for legal records, copyright,
patent rights, or anything like that, then you need to do some
research.  This post is **not** providing any recommendations for
that._

{::options parse_block_html="true" /}

<div class="post-toc">

{:.post-toc--header}
#### Contents

- [Overview](#overview)
- [Provenance tracking](#provenance-tracking)
- [A git-based lab notebook](#a-git-based-lab-notebook)
- [A CLI app to help manage git-based lab notebooks](#a-cli-app-to-help-manage-git-based-lab-notebooks)
- [A super simple example](#a-super-simple-example)

</div>

{::options parse_block_html="false" /}

## Overview

Keeping a good lab notebook for your computational work is important,
but it can be challenging.  A quick Google search will show you lots
of examples of people talking about it:

* [Ten Simple Rules for a Computational Biologist’s Laboratory Notebook](https://doi.org/10.1371/journal.pcbi.1004385)
* [Notebook & Data Management](https://ori.hhs.gov/education/products/wsu/data.html)
* [Lab Notebooks for Computational Science](https://scicomp.stackexchange.com/questions/35854/lab-notebooks-for-computational-science)
* [How to Keep a Lab Notebook for Bioinformatic Analyses](https://blog.addgene.org/how-to-keep-a-lab-notebook-for-bioinformatic-analyses)
* [Keeping a good lab notebook in a computational field?](https://www.reddit.com/r/labrats/comments/66dlgq/keeping_a_good_lab_notebook_in_a_computational/)

I have tried a lot of different methods, but they all more or less
boil down to a workflow sort of like this:

* Write down some summary of what I'm about to do and why.
* Run some commands, programs, or bash stuff.
* Copy what I did into a document. (e.g., [Markdown
notes](https://www.markdownguide.org/getting-started/) files,
[TiddlyWiki](https://tiddlywiki.com/), etc.)
* Write a bit more about what happened.
* Rinse and repeat.

Then, depending on my needs, I may clean up the analysis and put it
into an [R Markdown](https://rmarkdown.rstudio.com/) or [Jupyter
notebooks](https://jupyter.org/) notebook so it will be easier to
reproduce later.

One problem with this general workflow is that it requires tracking a
lot of things manually (e.g., copying and pasting).  Whenever you do a
lot of that, you will inevitably forget to paste a command into your
notebook.  You might make a mistake or typo when running a command,
and rather than noting it down in your notebook, you just rerun it and
pretty soon your lab notebook is out of sync with the commands that
you have actually run.  Another issue is that you may be running a
bunch of commands quickly, just testing some ideas out.  When doing
this, you end up needing to track a ton of things in an ad-hoc manner
leading to a messy lab notebook that you need to come back to later
and reorganize.

In other words, you've got to manually track a lot of information, and
it can be quite a challenge to keep it all straight!

## Provenance tracking

One approach to dealing with this problem is by tracking the
provenance of files.  An example of this is how [QIIME
2](https://doi.org/10.1038/s41587-019-0209-9) includes metadata in
their artifact files (`.qza` files) to [track things that were done in
an
analysis](https://docs.qiime2.org/2021.2/concepts/#data-files-qiime-2-artifacts).

I like the idea of provenence tracking, but even if you do use QIIME,
there are a lot of things you need to do outside of QIIME that will
need tracking.  While not quite the same, this sort of provenance
tracking reminds me a bit of using git or other version control
software.  [Git](https://git-scm.com/) is software used to track
changes in a set of files, and is often used by programmers during
software development.

{% include post_img_border.html path='computational_lab_notebooks/git_logo.png' caption="git -- a distributed version control system" %}

*Note: If you have never used git before, the [official
docs](https://git-scm.com/doc) have a lot of info that may be of use
to you.  I have also written a [small git
tutorial](https://mooreryan.github.io/computational_lab_notebooks/git/)
that you may find useful!*

While I had used git while working on software, I had never tried
using it to manage a computational lab notebook.  One reason is that
it [doesn't handle large files
well](https://stackoverflow.com/questions/3055506/git-is-very-very-slow-when-tracking-large-binary-files).
For computational work, whether bioinformatics or data science, you
will be dealing with a lot of large files.  Sequencing files easily
get over 10 GB in size, so using git alone is going to be problematic.
However, there are extensions to git like [Git Large File
Storage](https://git-lfs.github.com/) and
[git-annex](https://git-annex.branchable.com/) that help to address
this problem.  (Essentially, git-annex tracks [symbolic
links](https://en.wikipedia.org/wiki/Symbolic_link) in the git
repository rather than the file itself.  There is a lot more to it
than that, so you check out the [git-annex
walkthrough](https://git-annex.branchable.com/walkthrough/) if you
want to know more.)

## A git-based lab notebook

*Note: I'm not the first one to think of using git to help manage a
computational lab notebook.  In fact, you can find some interesting
discussion on whether version control is even useful for lab notebooks
[here](http://ivory.idyll.org/blog/is-version-control-an-electronic-lab-notebook.html),
[here](https://kbroman.org/blog/2013/08/20/electronic-lab-notebook/),
and
[here](https://yossadh.github.io/posts/2018/12/lab-notebook-part-2/).*

Using git and git-annex, I figured that I could get a pretty decent
workflow going for my computational lab notebook.  After playing
around with it for a while (and seeing that git-annex was a good
solution to git's large file problem), I settled into a pretty
familiar workflow:

* Run a program, script, whatever.
* Track any new files or changes with git.
* Commit the changes.
* Repeat.

One key difference from my "typical" workflow is that instead of
putting the commands that I ran and their explanations into some
external document like a markdown file, I would put all the
information into the commit message.  That way, all the info about how
and why I did something would be tracked in the git repository along
with the actual files and changes.

That works pretty well, but you still run in to the issue of having to
remeber what you ran, copy and paste it correctly into the commit
message, blah blah blah.  In other words, it's still a bit of a pain.
While you get the added benefits of git logs and history tracking, you
have to do a lot of repetative, annoying stuff to get things to work.
So, of course, I wrote a little program to help automate some of the
tedious stuff!

## A CLI app to help manage git-based lab notebooks

While working with the above workflow, in addition to QIIME's
provenance tracking, I was also reminded of [database
migrations](https://en.wikipedia.org/wiki/Schema_migration).
Basically, the way they work is that you write some script that says
how the database is supposed to change (e.g., add column `first_name`
to table `authors`), and then [some migration
tool](https://guides.rubyonrails.org/active_record_migrations.html#running-migrations)
handles actually making any changes to the database.  In theory, this
gives you a simpler way to track how your database has changed over
time--you can just follow the paper trail of your migration files.

The app I wrote works in a similar way, except that instead of making
incremental changes to a database, you are formalizing making changes
to the repository itself.  The app is called `cln` (it stands for
"computational lab notebooks"...clever, I know!).  You can find it on
[GitHub](https://github.com/mooreryan/computational_lab_notebooks).
There is also some pretty extensive [documentation
available](https://mooreryan.github.io/computational_lab_notebooks/)
to help you get started using the software.

While I suggest you check out the docs for a more detailed explanation
of its installation and usage, I want to show a quick, little
example to give you a flavor of how the `cln` program can help you
manage you git-based lab notebook.

## A super simple example

The `cln` command provides a couple of subcommands to help you manage
your lab notebook with git and git-annex.  (For more details on
individual subcommands, see
[here](https://mooreryan.github.io/computational_lab_notebooks/usage/)).

### Create a project

To start, you make a new project.

{% highlight text %}
{% raw %}
$ mkdir -p ~/projects/cln_example && cd ~/projects/cln_example
$ cln init 'Example Project'
$ tree -a -I .git
.
├── .actions
│   ├── completed
│   ├── failed
│   ├── ignored
│   └── pending
└── README.md
{% endraw %}
{% endhighlight %}

The `cln init` command initializes a new project, creates a git
repository, and generates some scaffolding for actions and git commit
templates.

### Prepare an action

Next, you prepare an action to run.  (Again, this is just a silly
example...for a more in depth tutorial, see the
[documentation](https://mooreryan.github.io/computational_lab_notebooks/)).

{% highlight text %}
{% raw %}
$ cln prepare 'printf "I like apple pie\n" > msg.txt'
{% endraw %}
{% endhighlight %}

In this case the action is just running a `printf` command and saving
the contents in a file.  Of course, you can prepare an action
containing anything that you would normally run at the command line.
For example, you could prepare a crazy action like this:

{% highlight text %}
{% raw %}
$ cln prepare "$(cat <<'EOF'
cut -f2 seq_information.seq_id_eco.tsv \
  | cut -d';' -f5 \
  | ruby -e 'h = Hash.new 0; \
      ARGF.each {|l| h[l.chomp] += 1 }; \
      h.sort_by {|_, count| count }.reverse. \
      each {|eco, count| puts "#{eco}\t#{count}" }' \
  | column -t \
  > seq_eco_counts.txt
EOF
)"
{% endraw %}
{% endhighlight %}

*Note: That's actually an action I prepared and ran in a real project.
Previously, I would have put that little ad-hoc
[Ruby](https://www.ruby-lang.org/en/) script into a file and ran it in
a way that is easier to track, but with the `cln` to help me manage
things, everything will be nicely tracked automatically.*

The `cln prepare` command creates an action file and a [git commit
template](https://git-scm.com/docs/git-commit/2.10.5#Documentation/git-commit.txt---templateltfilegt).
The action file is simply a bash script with the command you want to
run, but having it there in your repository as a standalone script
helps you see what is going on if you're running a complicated command
or when you come back to the project a couple of months later.

### Run the pending action

Next, you can check that everything is okay doing a [dry
run](https://en.wikipedia.org/wiki/Dry_run_(testing)).  It will spit
out some stuff to the terminal to let you know what's going on and
suggests what steps to take next.  *Note: I've edited the terminal
output a bit.*

{% highlight text %}
{% raw %}
$ cln run -dry-run
~~~
~~~
~~~ Hi!  I just previewed an action for you.
~~~
~~~ I plan to run this action file:
~~~   '.actions/pending/action__ ...'
~~~
~~~ It's contents are:
~~~
printf "I like apple pie\n" > msg.txt

~~~
~~~ If that looks good, you can run the action:
~~~   $ cln run
~~~
~~~
{% endraw %}
{% endhighlight %}

If it looks good, you can go ahead and run the action.

{% highlight text %}
{% raw %}
$ cln run
  ~~~
  ~~~
  ~~~ Hi!  I just ran an action for you.
  ~~~
  ~~~ * The pending action was '.actions/pending/action__REDACTED.sh'.
  ~~~ * The completed action is '.actions/completed/action__REDACTED.sh'.
  ~~~
  ~~~ Now, there are a couple of things you should do.
  ~~~
  ~~~ * Check which files have changed:
  ~~~     $ git status
  ~~~ * Add actions and commit templates:
  ~~~     $ git add .actions
  ~~~ * Unless they are small, add other new files with git annex:
  ~~~     $ git annex add blah blah blah...
  ~~~ * After adding files, commit changes using the template:
  ~~~     $ git commit -t '.actions/completed/action__REDACTED.gc_template.txt'
  ~~~
  ~~~ After that you are good to go!
  ~~~
  ~~~ * You can now check the logs with git log,
  ~~~   or use a GUI like gitk to view the history.
  ~~~
  ~~~
{% endraw %}
{% endhighlight %}

See how the `cln run` command gives you hints on what to do next?  I
tried to make all the `cln` commands spit out helpful info like that
to the terminal.

### Track and commit changes

Now, you will be able to see any files that were created or changed as
the result of running the action using `git status`.  Depending on the
size(s) of the file(s) that were created or changed, you can add them
to the [git
index](https://mooreryan.github.io/computational_lab_notebooks/git/#what-is-an-index)
with either `git add` or `git-annex add`.  Finally, you commit the
changes using the the git commit template that was made when you
prepared the action.

{% highlight text %}
{% raw %}
$ git commit -t '.actions/completed/action__REDACTED.gc_template.txt'
{% endraw %}
{% endhighlight %}

The template file will look something like this:

{% highlight text %}
{% raw %}
PUT COMMIT MSG HERE.

== Details ==
PUT DETAILS HERE.

== Command(s) ==
printf "I like apple pie\n" > msg.txt

== Action file ==
action__REDACTED.sh
{% endraw %}
{% endhighlight %}

When you run the `git commit` command, a text editor will pop up with
the contents of the git template file ready for you to fill out.  This
is nice because you can avoid manually copying in the commands you
ran.  For such a small example it's not really a big deal, but if
you're running some complicated bioinformatics software with a lot of
flags and options, it's pretty convenient!

### Browse the git history

After editing the message and saving the commit, you can browse
through your nicely organized repository history and see something
like this:

{% highlight text %}
{% raw %}
$ git log
commit ebf738 (HEAD -> master)
Author: Ryan Moore <moorer@udel.edu>
Date:   Mon Apr 5 18:44:54 2021 -0400

    Created the msg.txt file

    == Details ==
    I needed to create a file that describes something that I like.  I
    used the `printf` rather than `echo` because it is more portable.
    (See https://stackoverflow.com/a/11530298 for a discussion of this on
    stack overflow).

    == Command(s) ==
    /usr/bin/printf "I like apple pie\n" > msg.txt

    == Action file ==
    action__460986084__2021-04-05_18:02:37.sh

commit 1a2e90
Author: Ryan Moore <moorer@udel.edu>
Date:   Mon Apr 5 17:43:50 2021 -0400

    Initial commit
{% endraw %}
{% endhighlight %}

Notice how I put a short, descriptive commit message for the first
line, and then added in any additional details that I think I will
need later.  The `== Details ==` section would hold all the extra
stuff I would put in my lab notebook anyway, but it is really
convenient to have it right there in the git log.

Having the command that you ran, the details about that command, and
the changes that command effected in your repository opens up some
really powerful ways to track your analyses.

### Get individual file provenance info

For example, you can use the `git` cli app (e.g., `git whatchanged` or
`git log`) or a GUI like [gitk](https://git-scm.com/docs/gitk/) to get
detailed info about the provenence of any files in the repository.
You could run something like this to see all the history for the
`msg.txt` file.

{% highlight text %}
{% raw %}
$ git log --stat --follow -p -- msg.txt
commit ... (HEAD -> master)
Author: Ryan Moore <moorer@udel.edu>
Date:   ....

    Created the msg.txt file

    == Details ==
    I needed to create a file that describes something that I like.  I
    used the `printf` rather than `echo` because it is more portable.
    (See https://stackoverflow.com/a/11530298 for a discussion of this on
    stack overflow).

    == Command(s) ==
    printf "I like apple pie\n" > msg.txt

    == Action file ==
    action__467354640__.....sh
---
 msg.txt | 1 +
 1 file changed, 1 insertion(+)

diff --git a/msg.txt b/msg.txt
new file mode 100644
index 0000000..135d9d6
--- /dev/null
+++ b/msg.txt
@@ -0,0 +1 @@
+I like apple pie
{% endraw %}
{% endhighlight %}

As you can imagine, having output like that for all the files in your
project folder as well as the chronological logs is a very powerful
way to track your analyses and makes managing a computational lab
notebook so much easier.

## Wrap up

Managing a computational lab notebook is tricky.  I have found that
using git and git-annex can be a good way to keep all the info you
need right in the same directory as all your data files, scripts, and
analysis code.  To help you more easily manage lab notebooks using git
and git-annex, I created a command line app called `cln`.  You can
find the code on
[GitHub](https://github.com/mooreryan/computational_lab_notebooks).
Installation instructions and usage examples can be found in the
[documentation](https://mooreryan.github.io/computational_lab_notebooks/).
