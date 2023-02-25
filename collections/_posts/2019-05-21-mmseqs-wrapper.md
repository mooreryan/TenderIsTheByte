---
layout: post
title: "Running MMseqs2 on CPUs with SSE4.1 or AVX2 instruction sets"
author: ryan
description: MMseqs2, a program for searching and clustering proteins, requires a CPU with either AVX2 or SSE4.1 to run.  In this post, we walk through writing a script to automatically determine the instruction set on your computer and run the correct version of MMseqs2.
categories: blog
twitter_share: https://ctt.ac/AVJxt
last_updated: 2023-02-24
---

[MMseqs2](https://github.com/soedinglab/MMseqs2) is a software suite for searching and [clustering](https://doi.org/10.1038/s41467-018-04964-5) giant protein and nucleotide datasets.  It is [very fast while still being very sensitive](https://doi.org/10.1038/nbt.3988), and if you are using [BLAST](https://blast.ncbi.nlm.nih.gov/Blast.cgi) for homology search, I definitely recommend giving MMseqs2 a try!

## Single instruction, multiple data (SIMD)

MMseqs2 requires a 64-bit system with either SSE4.1 or AVX2 instruction sets to run.  [SSE](https://en.wikipedia.org/wiki/Streaming_SIMD_Extensions) (Streaming SIMD Extensions) and [AVX](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#Advanced_Vector_Extensions_2) (Advanced Vector Extensions) are instruction sets that take advantage of a CPU's ability to execute multiple instructions simultaneously ([instruction level parallelism](http://www.cs.uu.nl/docs/vakken/magr/2017-2018/files/SIMD%20Tutorial.pdf)).  CPUs with these instruction sets can use data level parallelism, which allows a program to [execute several computations at the same time](https://en.wikipedia.org/wiki/SIMD).

## Picking the correct version of MMseqs2

Basically, SSE4.1 and AVX2 allow MMseqs2 to do multiple data operations simultaneously, speeding up the program.  Not every CPU has SSE4.1 or AVX2, so to use MMseqs2, you will need to determine if your computer has one of these, and if so, which instruction set your computer has.  Then you can download and use the correct [pre-compiled release](https://github.com/soedinglab/MMseqs2/releases) for whichever instruction set your computer supports.  AVX is [generally faster than SSE](https://techblog.lankes.org/2014/06/16/AVX-isnt-always-faster-than-SEE/), so you will want to use the AVX2 version of MMseqs2 if you can.

If you are running MMseqs2 locally, this is no problem.  You can [check](https://github.com/soedinglab/MMseqs2#installation) whether your computer supports AVX2 or SSE4.1 and then pick the correct MMseqs2 version.  However, I generally run MMseqs2 on a computing cluster.  The cluster that our lab uses has a lot of nodes with various architectures.  Some nodes have SSE4.1, some have AVX2, and some have neither.  Because of this, a different version of MMseqs2 is needed depending on where the job is run.  

The cluster uses [slurm](https://slurm.schedmd.com/documentation.html) for job scheduling, which means that I could figure out which nodes have AVX2 and which have SSE4.1 and then specify the allowed nodes in my slurm submission script (for example, using `--nodelist=node_name`) whenever I want to run MMseqs2.  I got tired of doing this pretty quickly, so I thought it would be nice to write a little shell script to automatically pick the correct version of MMseqs2 depending on the instruction set of the CPU.  It turns out that this is pretty easy to do.  Let's walk through it!

*Note: Like my [last](https://www.tenderisthebyte.com/blog/2019/05/08/parsing-cli-args-with-structopt/) [two](https://www.tenderisthebyte.com/blog/2019/04/25/rotating-axis-labels-in-r/) posts, I'm going to go into a fair bit of detail so that it is accessible to beginners.  If you just want to see the final result, [skip down to the bottom](#wrapping-up).*

## Checking for instruction sets

First, we need a way to figure out which instruction set the computer we will run MMseqs2 on has.  It is a bit different depending on whether you are running on Linux or on a Mac.  Here is how you would check for the instruction sets on Linux:

{% highlight bash %}
{% raw %}
# Check for AVX2
$ grep avx2 /proc/cpuinfo

# Check for SSE4.1
$ grep sse4_1 /proc/cpuinfo
{% endraw %}
{% endhighlight %}

And on a Mac... 

{% highlight bash %}
{% raw %}
# Check for AVX2
$ sysctl -a | grep machdep.cpu.leaf7_features | grep AVX2

# Check for SSE4.1
$ sysctl -a | grep machdep.cpu.features | grep SSE4.1
{% endraw %}
{% endhighlight %}

*([Here](http://doc.callmematthi.eu/static/webArticles/Understanding%20Linux%20_proc_cpuinfo.pdf) is some more info on interpreting the `/proc/cpuinfo` file.)*

If your computer has either of the instruction sets, then the `grep` command will print a line to the terminal.  If not, nothing will be shown.  For example, on my Mac, when I check for `SSE4.1` here is the result

{% highlight bash %}
{% raw %}
sysctl -a | grep machdep.cpu.features | grep SSE4.1
machdep.cpu.features: FPU VME DE PSE TSC MSR PAE MCE CX8 APIC SEP MTRR PGE MCA CMOV PAT PSE36 CLFSH DS ACPI MMX FXSR SSE SSE2 SS HTT TM PBE SSE3 PCLMULQDQ DTES64 MON DSCPL VMX SMX EST TM2 SSSE3 CX16 TPR PDCM SSE4.1 SSE4.2 POPCNT AES PCID
{% endraw %}
{% endhighlight %}

Since `grep` returned a line with `SSE4.1` in it, I know that my laptop supports SSE4.1.  On the other hand, when I run command to check for `AVX2`, nothing is printed, so my laptop doesn't support AVX2.  Since my computer has SSE4.1, but not AVX2, I would go and grab a [precompiled SSE4.1 binary for Mac](https://github.com/soedinglab/MMseqs2/releases/download/9-d36de/MMseqs2-MacOS-SSE4_1.tar.gz) from the MMseqs2 [GitHub page](https://github.com/soedinglab/MMseqs2) and use that.

## A wrapper script for MMseqs2

As I mentioned above, it's simple to pick an MMseqs2 binary for my own computer, but our computing cluster has some nodes with AVX2 and some with SSE4.1.  To deal with this, we will write a little shell script that determines which instruction set a computer has, and then runs the correct version of MMseqs2 for that instruction set.

First, download both the SSE4.1 and the AVX2 versions of MMseqs2 [from the website](https://github.com/soedinglab/MMseqs2/releases).  Then, put each of those in its own location.  For example, I have the AVX2 binary here

{% highlight bash %}
{% raw %}
/home/moorer/software/MMseqs2_binaries/mmseqs2_avx2/bin/mmseqs
{% endraw %}
{% endhighlight %}

and the SSE4.1 binary here

{% highlight bash %}
{% raw %}
/home/moorer/software/MMseqs2_binaries/mmseqs2_sse41/bin/mmseqs
{% endraw %}
{% endhighlight %}


The first thing our script needs is the [Shebang](https://bash.cyberciti.biz/guide/Shebang) line, which specifies the interpreter for our script, and some variables to hold the locations of the different MMseqs2 binaries.

{% highlight bash %}
{% raw %}
#!/bin/bash

# Variables to hold the different mmseqs binaries
mmseqs_avx2=/home/moorer/software/MMseqs2_binaries/mmseqs2_avx2/bin/mmseqs
mmseqs_sse41=/home/moorer/software/MMseqs2_binaries/mmseqs2_sse41/bin/mmseqs
{% endraw %}
{% endhighlight %}

### Checking for AVX2 instructions

Since we want to use the AVX2 version of MMseqs2 if possible, we will check for that instruction set first.

{% highlight bash %}
{% raw %}
grep avx2 /proc/cpuinfo > /dev/null
{% endraw %}
{% endhighlight %}

That line is the same command to check for AVX2 on Linex shown above, but with the output [redirected](http://www.westwind.com/reference/os-x/commandline/pipes.html#redir-output) to `/dev/null`.  `/dev/null` is the null device, a [special file](https://medium.com/@codenameyau/step-by-step-breakdown-of-dev-null-a0f516f53158) that discards anything written to it.  If we didn't redirect the `grep` program's output to `/dev/null`, then every time we ran our script, it would spit out a long line of stuff to the terminal, which we don't want.  So redirecting the output in this way will keep our script's output nice and clean.

Now we need to check whether the command was successful.  For this, we can check the [exit code](https://shapeshed.com/unix-exit-codes/) of the `grep` command.  If you check the [man page](https://linux.die.net/man/1/grep) for `grep`, the Exit Status section states that `grep` returns `0` if there were any selected lines, and something else otherwise.  So, if we check for a zero exit code from the `grep` command, we will find out whether or not the computer has the AVX2 instructions.  Here is an example.  *(The `$` means whatever follows is a command entered into the terminal.)*

{% highlight bash %}
{% raw %}
# This computer does not have AVX2 instructions
$ grep avx2 /proc/cpuinfo > /dev/null
$ echo $?
1 # <= grep returned exit code 1, meaning failure

# This computer does have AVX2 instructions
$ grep avx2 /proc/cpuinfo > /dev/null
$ echo $?
0 # <= grep returned exit code 0, meaning success
{% endraw %}
{% endhighlight %}

[$? returns the exit status of the last command](https://www.tldp.org/LDP/abs/html/exit-status.html) that was run (in this case, that would be `grep`).  In the first example, running `$?` returns `1` indicating failure (this computer doesn't have the AVX2 instruction set), whereas in the second case, it returns `0`, indicating success (this computer does have AVX2 instructions).

Alright, so let's add a check like this into our bash script.  For this, we can use an [if statement](http://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO-6.html).  In bash, it will look something like this.

{% highlight bash %}
{% raw %}
if [ $? -eq 0 ]; then
    # do something if the check is true
else
    # do something else if the check is false
fi
{% endraw %}
{% endhighlight %}

The `[ $? -eq 0 ]` (don't forget the spaces around the brackets!) part checks whether the exit code of the last command was zero (i.e., was successful).  If the first `grep` command is successful, then the computer has AVX2 instructions, so we want to run the AVX2 version of MMseqs2.  Here is how that would look.

{% highlight bash %}
{% raw %}
# Check the exit code of the last grep command.
if [ $? -eq 0 ]; then
    # Run the AVX2 version of MMseqs2.
    "$mmseqs_avx2" "$@"
else
    # do something else
fi
{% endraw %}
{% endhighlight %}

The `$@` variable [gives all command line parameters separated by spaces](https://stackoverflow.com/a/3816747).  That way, any arguments passed in to our wrapper script will get passed in to the actual `mmseqs` program.  For example, if we name our wrapper script `mmseqs_wrapper.sh` and call it like this

{% highlight bash %}
{% raw %}
$ mmseqs_wrapper.sh easy-search -h
{% endraw %}
{% endhighlight %}

then the `"$mmseqs_avx2" "$@"` line will basically be like writing this

{% highlight bash %}
{% raw %}
$ /home/moorer/software/MMseqs2_binaries/mmseqs2_avx2/bin/mmseqs easy-search -h
{% endraw %}
{% endhighlight %}

### Checking for SSE4.1 instructions

If the computer doesn't have AVX2, we want to check to see if it has SSE4.1.  We can use this command `grep sse4_1 /proc/cpuinfo > /dev/null` for that.  We add it to the else branch like this:

{% highlight bash %}
{% raw %}
# Check the exit code of the first grep command.
if [ $? -eq 0 ]; then
    # Run the AVX2 version of MMseqs2.
    "$mmseqs_avx2" "$@"
else # This computer doesn't have AVX2!
    # Check for SSE4.1 instruction set instead.
    grep sse4_1 /proc/cpuinfo > /dev/null
fi
{% endraw %}
{% endhighlight %}

Now we can check the exit code of that command, and if successful, run the SSE4.1 version of MMseqs2.  To do that, we will add another `if/else` statement similar to the first one.  (There may be a cleaner way to do this avoiding the nested if statements, but we can keep it like this for now.)

{% highlight bash %}
{% raw %}
# Check the exit code of the first grep command.
if [ $? -eq 0 ]; then
    # Run the AVX2 version of MMseqs2.
    "$mmseqs_avx2" "$@"
else # This computer doesn't have AVX2!
    # Check for SSE4.1 instruction set.
    grep sse4_1 /proc/cpuinfo > /dev/null

    # Check the exit code of the second grep command.
    if [ $? -eq 0 ]; then
        # Run the SSE4.1 version of MMseqs2.
        "$mmseqs_sse41" "$@"
    else # This computer doesn't have SSE4.1.
        # Do something.
    fi
fi
{% endraw %}
{% endhighlight %}

### Handling CPUs without AVX2 or SSE4.1

Finally, we need to do something in the last `else` branch.  If the computer has neither AVX2 or SSE4.1, then MMseqs2 will not work.  So let's print a message to [stderr](https://www.jstorimer.com/blogs/workingwithcode/7766119-when-to-use-stderr-instead-of-stdout) and return a failing exit code.  Since `0` is usually used to denote success, we will use `1` to indicate failure.

In bash, to [print something to stderr](https://stackoverflow.com/a/23550347), we can use the `>&2` operator with the `echo` command like this: `>&2 echo "hello"`.  Check out [this article](https://wiki.bash-hackers.org/howto/redirection_tutorial) for more info on redirection and the difference between `stdin` and `stderr`.

To exit our script with a failing error code, we can use the `exit` command with an argument.  For example, `exit 1` would terminate the script and return an exit code of `1`.

Adding those two things to our `if/else` statement, we now have 

{% highlight bash %}
{% raw %}
# Check the exit code of the first grep command.
if [ $? -eq 0 ]; then
    # Run the AVX2 version of MMseqs2.
    "$mmseqs_avx2" "$@"
else # This computer doesn't have AVX2!
    # Check for SSE4.1 instruction set.
    grep sse4_1 /proc/cpuinfo > /dev/null

    # Check the exit code of the second grep command.
    if [ $? -eq 0 ]; then
        # Run the SSE4.1 version of MMseqs2.
        "$mmseqs_sse41" "$@"
    else # This computer doesn't have SSE4.1.
        # Print an error message.
        >&2 echo "ERROR -- you don't have sse4.1 or avx2.  MMseqs2 will not work!"

        # Exit the script, returning exit code 1.
        exit 1
    fi
fi
{% endraw %}
{% endhighlight %}

## Wrapping up

Alright that's everything!  The final script looks like this:

{% highlight bash %}
{% raw %}
#!/bin/bash

mmseqs_avx2=/home/moorer/software/MMseqs2_binaries/mmseqs2_avx2/bin/mmseqs
mmseqs_sse41=/home/moorer/software/MMseqs2_binaries/mmseqs2_sse41/bin/mmseqs

grep avx2 /proc/cpuinfo > /dev/null

if [ $? -eq 0 ]; then
    "$mmseqs_avx2" "$@"
else
    grep sse4_1 /proc/cpuinfo > /dev/null

    if [ $? -eq 0 ]; then
        "$mmseqs_sse41" "$@"
    else
        >&2 echo "ERROR -- you don't have sse4.1 or avx2.  MMseqs2 will not work!"
        exit 1
    fi
fi
{% endraw %}
{% endhighlight %}

If you want to download and use the above code, here is a [link to the code](https://gist.github.com/mooreryan/b46fdafa7d56046dbe126bdb27bd5640).