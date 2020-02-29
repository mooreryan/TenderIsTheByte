---
layout: post
title: "Notes from the Messyverse:  How to tidy nested lists in R"
author: ryan
description: You've got nested lists in R.  In fact, you've got lists of nested lists, but ggplot wants data frames or tibbles.  Have no fear, this post will show you how to tidy up your nested lists by converting them to data frames!
categories: blog
image: "/assets/img/posts/tidying_nested_lists/emacs_watermelon_data_11.jpg"
twitter_share: https://ctt.ac/Q59Rg
---

{:.gray}
*You're a fairly recent convert to the Tidyverse, and you're still using an unholy amalgamation of [Tidy verbs](https://teachingr.com/content/the-5-verbs-of-dplyr/the-5-verbs-of-dplyr-article.html) and base R throughout your code.  What's more, you've got reams of legacy code that's not going to magically tidy itself up anytime soon.  Don't feel bad about you're heathen love of the [apply](https://www.guru99.com/r-apply-sapply-tapply.html) family (after all, even [Hadley](http://hadley.nz) says that [`map` is just a fancier version of `lapply`](https://adv-r.hadley.nz/functionals.html#map)).  Rather, embrace those old-school list of lists!  All your nice, modern [tibbles](https://tibble.tidyverse.org) are just a few short verbs away.*

So you're walking down the hall to the office breakroom when you overhear a conversation some of your colleagues are having.  You're coming closer and things must be starting to get a little intense in there.  One of them starts to shout.  "[By Jove](https://en.wiktionary.org/wiki/by_Jove), it's [lists all the way down](https://en.wikipedia.org/wiki/Turtles_all_the_way_down#Notable_modern_allusions_or_variations)!"  By this time you're starting to hurry...are they really talking about...?  You fly around the corner and see a grad student with [Professor Gamgee](http://tolkiengateway.net/wiki/Gaffer_Gamgee) and his flowing [grey beard](https://www.urbandictionary.com/define.php?term=grey%20beard) huddled in front of an old [CRT](https://en.wikipedia.org/wiki/Cathode-ray_tube) workstation, with what looks like--*gasp*--an [Emacs](https://ess.r-project.org) [buffer](https://www.gnu.org/software/emacs/manual/html_node/emacs/Buffers.html) filled top to bottom with--no, wait is that a [JSON](http://www.json.org) file?--the biggest, most nested, list-of-lists you've ever seen! 

"Oh good, you made it," one of them says.

"Who, me?"

"Yes, you're the one who makes all those pretty graphs right?  With something called the [ggplot](https://ggplot2.tidyverse.org/reference/ggplot.html)?"  

You cough, "Maybe a little."

"Well take a seat then!  Take a seat!"

The CRT flickers as you take the offered chair.  Gamgee gives it a quick *thwack*.  It clears up, and you give it a look:

{% include post_video_border.html path='tidying_nested_lists/emacs_watermelon_data.mp4' caption="Emacs on a CRT..." %}

"What's with the all-caps variable names?"

"Huh?  Oh that, well, you see, I used to love [Common Lisp](https://common-lisp.net), and, well, [the reader was always case converting](https://www.cliki.net/Case%20sensitivity), so...."

"Wow, Common Lisp, eh?  I guess, that explains the Emacs...."

"Alright kid, so we've got the watermelon data in...you know the experiment, right?  No?  Well, we were testing out some new fertilizer on our melons.  As you can see, we did a couple of different experiments.  Each experiment has two groups.  The `Control` group used the standard husbandry procedures, and the `Treatment` group got our new fertilization strategy.  Got it?  Alright then, we'll leave the plotting to you.  See you in an hour or so."

Stomach growling, you shoot a sideways glance at the fridge.  Soups and sandwiches will just have to wait.

First things first, you get rid of all those upcased variable names.  A few quick [Emacs macros](https://www.emacswiki.org/emacs/KeyboardMacros) and you're in business:

{% highlight R %}
{% raw %}
> experiments <- list(
  A = list(
    C1 = list(
      group = "Control",
      weight = 10
    ),
    C2 = list(
      group = "Control",
      weight = 8      
    ),
    T1 = list(
      group = "Treatment",
      weight = 15     
    ),
    T2 = list(
      group = "Treatment",
      weight = 14
    )
  ),
  B = list(
    C1 = list(
      group = "Control",
      weight = 8
    ),
    C2 = list(
      group = "Control",
      weight = 7
    ),
    T1 = list(
      group = "Treatment",
      weight = 15.2
    ),
    T2 = list(
      group = "Treatment",
      weight = 16
    )
  ),
  C = list(
    C1 = list(
      group = "Control",
      weight = 8.5
    ),
    C2 = list(
      group = "Control",
      weight = 9
    ),
    T1 = list(
      group = "Treatment",
      weight = 13.6
    ),
    T2 = list(
      group = "Treatment",
      weight = 15.2
    )
  )
)
{% endraw %}
{% endhighlight %}

Being a lover of all things [Tidy](https://tidyverse.tidyverse.org/articles/manifesto.html), you think, "Wow! lists are okay, but I really prefer [tibbles](https://tibble.tidyverse.org)....it's soo easy to plot them with ggplot!"  You remember someone mentioning a function to convert untidy things to tibbles.  What was that again?  Oh yeah, `as_tibble`.  Well, why not give it a try?  You decide to take it step-by-step so you pull out the first experiment and work with that to start.


{% highlight R %}
{% raw %}
> library(tidyverse)

> experiment_a <- experiments$A
> experiment_a %>% as_tibble

# A tibble: 2 x 4
  C1        C2        T1        T2       
  <list>    <list>    <list>    <list>   
1 <chr [1]> <chr [1]> <chr [1]> <chr [1]>
2 <dbl [1]> <dbl [1]> <dbl [1]> <dbl [1]>
{% endraw %}
{% endhighlight %}

Hmm, that's not quite what you want--it looks like there is a column for each list.  You seem to remember a function for applying other functions to elements of vectors.  Ah yes, it's called `map` from the [purrr](https://purrr.tidyverse.org) package.  But wait, you think, I have lists not vectors, and the title of the [help page](https://purrr.tidyverse.org/reference/map.html) is clear:  

> Apply a function to each element of a vector

It turns out that [lists](https://adv-r.hadley.nz/vectors-chap.html#lists) are just vectors.  (Try running `list() %>% is.vector`.)  Oh and look at that, there are special versions of map called `map_dfr` and `map_dfc` which return data frames by [row-binding and column-binding](https://dplyr.tidyverse.org/reference/bind.html) respectively.  

In general, `map` returns a vector the same length as its first argument (e.g., `map` returns a `list`, `map_int` returns an integer vector, and `map_dfr` returns a data frame by row binding.)  Since you want each of the nested lists to be a row in your data frame, you'll need `map_dfr`.  But what kind of function to you need to apply to each of the lists?  It turns out that you need a function that returns its argument as is, somthing like `function(l) l`.  You give that a try.

{% highlight R %}
{% raw %}
> experiment_a %>% map_dfr(function(l) l)

# A tibble: 4 x 2
  group     weight
  <chr>      <dbl>
1 Control       10
2 Control        8
3 Treatment     15
4 Treatment     14
{% endraw %}
{% endhighlight %}

Hey that worked!  But writing that whole `function(l) l` seems kind of unnecessary, so you wonder if there is a better way.  The help page for `map` mentions that you can write [anonymous](https://en.wikipedia.org/wiki/Anonymous_function) functions with formulas (e.g., `~ .x + 2`, would be converted to `function(x) x + 2`).  Given your love of [syntax sugar](todo), you give it a shot.

{% highlight R %}
{% raw %}
> experiment_a %>% map_dfr(~ .x)

# A tibble: 4 x 2
  group     weight
  <chr>      <dbl>
1 Control       10
2 Control        8
3 Treatment     15
4 Treatment     14
{% endraw %}
{% endhighlight %}

Now, you could write `~ .` since it is just a single argument function, but [Hadley says](https://adv-r.hadley.nz/functionals.html) you should avoid this.  The `~ .x` thing looks kind of cool, but it is definitely a little obscure for someone who isn't too familiar with the Tidyverse.  

Just then, you remember that base R has a function called `identity`, which returns its argument as is.  [Identity functions](https://emilvarga.com/posts/2016/08/01/using-identity-functions) are much beloved by [functional programmers](https://en.wikipedia.org/wiki/Functional_programming) and [mathematicians](http://mathworld.wolfram.com/IdentityFunction.html) alike, and [Tidyverse feels pretty functional](https://tidyverse.tidyverse.org/articles/manifesto.html#embrace-functional-programming).  Not to mention that it's clearer to [be explicit](https://tidyverse.tidyverse.org/articles/manifesto.html#design-for-humans) about what you're doing rather than to use sweet syntax sugar.  So you adjust your code once more.

{% highlight R %}
{% raw %}
> experiment_a %>% map_dfr(identity)

# A tibble: 4 x 2
  group     weight
  <chr>      <dbl>
1 Control       10
2 Control        8
3 Treatment     15
4 Treatment     14
{% endraw %}
{% endhighlight %}

That's not too shabby.  But wait...isn't it kind of overkill to use `map_dfr` if you're just passing in the identity funciton anyway?  Back on the help page you see this:

> `map_dfr()` and `map_dfc()` return data frames created by row-binding and column-binding respectively.

So if you're only passing in the identity function to `map_dfr`, then really you're just exploiting `map_dfr` for its ability to make data frames (or tibbles!) through row-binding.  In that case, couldn't you just use `bind_rows` directly?  

{% highlight R %}
{% raw %}
> experiment_a %>% bind_rows

# A tibble: 4 x 2
  group     weight
  <chr>      <dbl>
1 Control       10
2 Control        8
3 Treatment     15
4 Treatment     14
{% endraw %}
{% endhighlight %}

Yes!  [Now this is programming](https://knowyourmeme.com/memes/now-this-is-podracing)!  

{% include post_img_border.html path='tidying_nested_lists/now_this_is_programming.jpg' caption="Now this is <s>podracing</s> programming!" %}

The `bind_rows` trick works with your data because each sublist has names.

{% highlight R %}
{% raw %}
> experiment_a %>% map(names)

$C1
[1] "group"  "weight"

$C2
[1] "group"  "weight"

$T1
[1] "group"  "weight"

$T2
[1] "group"  "weight"
{% endraw %}
{% endhighlight %}

If you remove the names and try it again, you'll see that `bind_rows` doesn't work anymore.

{% highlight R %}
{% raw %}
> experiment_a %>% map(unname) %>% bind_rows

Error: Argument 1 must have names
{% endraw %}
{% endhighlight %}

Luckily, your collegues are very well-organized and each of the lists you were given has names, so you stick with the `bind_rows` method.

Now, you're thinking that it would be a good idea to include the sample name and the experiment name in the tibble.  That will make it easier to color and group elements of the figure.  For that, you use [mutate](https://dplyr.tidyverse.org/reference/mutate.html).

{% highlight R %}
{% raw %}
> experiment_a %>% 
  # Convert the list to a tibble.
  bind_rows %>% 
  # Add in a column with sample names and experiment name.
  mutate(sample = names(experiment_a),
         experiment = "A")

# A tibble: 4 x 4
  group     weight sample experiment
  <chr>      <dbl> <chr>  <chr>     
1 Control       10 C1     A         
2 Control        8 C2     A         
3 Treatment     15 T1     A         
4 Treatment     14 T2     A                  
{% endraw %}
{% endhighlight %}

While that's a good solution for a single expeiment, you remember that you've got a whole list of experiments.  Remember that `map` will return something that is the same size as your input, which will be the `experiments` list.  

Now, that list contains three experiments, each of which is just like your `experiment_a` test case.  What you want to do is map that little pipeline that you used to convert `experiment_a` into a tibble onto each of the lists in `experiments`.  Sound good?  But first, you decide to encapsulate the process into a named function so it's easier to work with.


{% highlight R %}
{% raw %}
> experiment_to_tibble <- function(experiment, experiment_name) {
  experiment %>% 
    bind_rows %>% 
    mutate(sample = names(experiment), 
           experiment = experiment_name)
}
{% endraw %}
{% endhighlight %}

Now you're ready to map this function onto each of the elements of the `experiments` list.

{% highlight R %}
{% raw %}
> experiments %>% map_dfr(experiment_to_tibble)

Error: argument "experiment_name" is missing, with no default
{% endraw %}
{% endhighlight %}

Whoops, that's not quite right.  Back to the `map` [help page](https://purrr.tidyverse.org/reference/map.html).  Okay, so it looks like you can pass additional arguments to the mapped function.  Something like this:

{% highlight R %}
{% raw %}
> experiments %>% map_dfr(experiment_to_tibble, experiment_name = "A")

# A tibble: 12 x 4
   group     weight sample experiment
   <chr>      <dbl> <chr>  <chr>     
 1 Control     10   C1     A         
 2 Control      8   C2     A         
 3 Treatment   15   T1     A         
 4 Treatment   14   T2     A         
 5 Control      8   C1     A         
 6 Control      7   C2     A         
 7 Treatment   15.2 T1     A         
 8 Treatment   16   T2     A         
 9 Control      8.5 C1     A         
10 Control      9   C2     A         
11 Treatment   13.6 T1     A         
12 Treatment   15.2 T2     A         
{% endraw %}
{% endhighlight %}

It worked at least, but you want the actual experiment names in there.  So you try something else:

{% highlight R %}
{% raw %}
> experiments %>% map_dfr(experiment_to_tibble, experiment_name = names(experiments))

Error: Column 'experiment' must be length 4 (the number of rows) or one, not 3
{% endraw %}
{% endhighlight %}

Nope, that doesn't work either.  Hold on...what you need is a function sort of like `map` except that it can map over multiple inputs instead of just one.  You pop over to your web browser and search for `map over multiple arguments tidyverse`.

{% include post_video_border.html path='tidying_nested_lists/tidyverse_map2_search.mp4' caption="You're feeling lucky..." %}

The [purrr](https://purrr.tidyverse.org) package has a function for this:  `map2`.  It works more or less the same way as the regular `map` variants, except that you can [iterate over multiple arguments simultaneously](https://purrr.tidyverse.org/reference/map2.html).  This way, you can pass in the experiment names as an additional argument *before* the function argument.  (Arguments that come before the function argument `.f` will all be vectorized, but function arguments coming after the `.f` are supplied to each call directly.)  Something like this:

{% highlight R %}
{% raw %}
> experiments %>% map2_dfr(names(experiments), experiment_to_tibble)

# A tibble: 12 x 4
   group     weight sample experiment
   <chr>      <dbl> <chr>  <chr>     
 1 Control     10   C1     A         
 2 Control      8   C2     A         
 3 Treatment   15   T1     A         
 4 Treatment   14   T2     A         
 5 Control      8   C1     B         
 6 Control      7   C2     B         
 7 Treatment   15.2 T1     B         
 8 Treatment   16   T2     B         
 9 Control      8.5 C1     C         
10 Control      9   C2     C         
11 Treatment   13.6 T1     C         
12 Treatment   15.2 T2     C         
{% endraw %}
{% endhighlight %}

That works, but wouldn't it be nice if there was a `map` variant that could apply a function to each element in the `experiments` along with the name of the experiment automatically?  You flip over to Google once more and are rewarded with the `imap` function.  The first line of the help function looks promising:

> Apply a function to each element of a vector, and its index

As before, you want to iterate over a list and not a vector, but that's okay, since you know that deep down, lists are just special vectors.  In fact, the [help page](https://purrr.tidyverse.org/reference/imap.html) assures you that all will be well:

> imap_xxx(x, ...), an indexed map, is short hand for map2(x, names(x), ...) if x has names

That's perfect for your use case.  In fact, that's exactly what you were doing, so you just replace the `map2_dfr` with `imap_dfr` and drop the `names(experiments)` bit like so:

{% highlight R %}
{% raw %}
> experiments %>% imap_dfr(experiment_to_tibble)

# A tibble: 12 x 4
   group     weight sample experiment
   <chr>      <dbl> <chr>  <chr>     
 1 Control     10   C1     A         
 2 Control      8   C2     A         
 3 Treatment   15   T1     A         
 4 Treatment   14   T2     A         
 5 Control      8   C1     B         
 6 Control      7   C2     B         
 7 Treatment   15.2 T1     B         
 8 Treatment   16   T2     B         
 9 Control      8.5 C1     C         
10 Control      9   C2     C         
11 Treatment   13.6 T1     C         
12 Treatment   15.2 T2     C         
{% endraw %}
{% endhighlight %}

Alright, [now you're cooking with Crisco](https://popculturekings.blog/2017/06/12/just-in-the-nick-of-time-a-history-of-interesting-idioms-and-colloquial-phrases-part-5/)!  Finally you've got a nice tibble ready to pipe into ggplot.

{% highlight R %}
{% raw %}
> experiments %>% 
  imap_dfr(experiment_to_tibble) %>% 
  ggplot(aes(x = experiment, y = weight, fill = group)) +
  scale_fill_manual(name = "Group", values = c("#E47320", "#BD74A8")) +
  geom_boxplot() +
  xlab("Experiment") +
  ylab("Weight") +
  theme_bw() + 
  theme(panel.grid = element_blank(),
        panel.border = element_blank(),
        axis.line = element_line(colour = "#333333"))
{% endraw %}
{% endhighlight %}

{% include post_img_border.html path='tidying_nested_lists/experiment_box_plot.svg' caption="The Watermelon Plot" %}

Phew!  That wasn't so bad, was it?  Right on cue, Professor Gamgee turns the corner into the breakroom and sits down at your table.  "How's the data looking, kid?"  

"Tidy professor.  It's looking Tidy."