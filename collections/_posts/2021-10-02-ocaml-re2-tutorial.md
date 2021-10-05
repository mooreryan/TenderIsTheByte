---
layout: post
title: An introduction to the re2 regular expression library for OCaml
author: ryan
description: In this post, I give an introduction and guide to help you get started using OCaml's re2 regular expression library, which provides OCaml bindings for Google's regular expression library, RE2.
categories: blog
image: "/assets/img/posts/ocaml_re2_tutorial/ocaml_regex.png"
twitter_share: https://ctt.ac/OadcW
last_updated: 2021-10-05
---

In this tutorial, we will talk about [re2](https://github.com/janestreet/re2), an OCaml library providing bindings to [RE2](https://github.com/google/re2), Google's regular expression library.  

This post is intended for newer OCaml programmers, or those who want to use the `re2` library, but could use a couple of examples to help get started.  This is not a general introduction to regular expressions, however.  If you have never used regular expressions before, read up a little bit on the syntax before tackling this post.

{::options parse_block_html="true" /}

<div class="post-toc">

{:.post-toc--header}
#### Contents

* [Overview](#overview)
* [Creating regular expressions](#creating-regular-expressions)
* [Checking for a match](#checking-for-a-match)
* [Finding matches](#finding-matches)
* [Finding submatches](#finding-submatches)
* [Splitting strings](#splitting-strings)
* [Replacing](#replacing)
* [Miscellaneous info](#miscellaneous-info)
* [Wrap up](#wrap-up)

</div>

{::options parse_block_html="false" /}

## Overview

The there are few choices for regular expression libraries available for OCaml on [Opam](https://opam.ocaml.org/).  Some of the most popular include

* [re](https://opam.ocaml.org/packages/re), a pure OCaml library (installed 7667 times last month),
* [pcre](https://opam.ocaml.org/packages/pcre), bindings to the Perl Compatibility Regular Expressions library ([PCRE](https://www.pcre.org/)), (installed 1115 times last month), and
* [re2](https://opam.ocaml.org/packages/re2), OCaml bindings for RE2, Google's regular expression library (installed 114 times last month).

The first two are by far the most popular in terms of raw Opam install counts.  However, `re2` integrates nicely into the Jane Street Base/Core/Async ecosystem (it's a Jane Street package after all!), and is covered under the MIT license rather than the [LGPL with OCaml linking exception](https://spdx.org/licenses/OCaml-LGPL-linking-exception.html), which may be appealing depending on your situation.

*Note: According to this [blog post](https://blog.janestreet.com/what-the-interns-have-wrought-2020/) and this [GitHub issue](https://github.com/janestreet/re2/issues/26#issuecomment-395870146), Jane Street is phasing out its use of re2. The [re2 GitHub](https://github.com/janestreet/re2) does have recent commits, though, so your mileage may vary.*

One issue that newcomers may face when getting started with the `re2` library is the slightly terse [API documentation](https://ocaml.janestreet.com/ocaml-core/latest/doc/re2/Re2/index.html).  While it is detailed and thorough, it can be hard to get started with if you're not already used to reading Jane Street `mli` files and source code.

*Note: if you want to follow along, you can paste the examples into the toplevel (or [utop](https://opam.ocaml.org/blog/about-utop/)).  However, don't paste in lines starting with `- :`.  These lines show the type of the expression as reported by `utop`.*

## Creating regular expressions

You create regular expressions with `Re2.create` and `Re2.create_exn`.  The former returns `Re2.t Or_error.t` and the latter `Re2.t`.

{% highlight OCaml %}
{% raw %}
let re = Or_error.ok_exn @@ Re2.create "apple";;
let re = Re2.create_exn "apple";;
{% endraw %}
{% endhighlight %}

### Matching options

You can control how regular expression matching works by passing the `options` argument to the `create` and `create_exn` functions.  If you omit this argument, the default options will be passed.  Here they are:

{% highlight OCaml %}
{% raw %}
Re2.Options.default;;
- : Re2.Options.t =
{
  Re2.Options.case_sensitive = true;
  dot_nl = false;
  encoding = Re2.Options.Encoding.Utf8;
  literal = false;
  log_errors = false;
  longest_match = false;
  max_mem = 8388608;
  never_capture = false;
  never_nl = false;
  one_line = false;
  perl_classes = false;
  posix_syntax = false;
  word_boundary = false;
}
{% endraw %}
{% endhighlight %}

For a more detailed description of these options, see the [re2.h](https://github.com/janestreet/re2/blob/89373a48bc786be9b2a7f530dd5954222515c048/src/re2_c/libre2/re2/re2.h#L509) header filer.

By default, `re2` uses case-sensitive matching.  To create a case-insensitive regex, pass in an options map like so.

{% highlight OCaml %}
{% raw %}
let re_i =
  let options = { Re2.Options.default with case_sensitive = false } in
  Re2.create_exn ~options "abc"
{% endraw %}
{% endhighlight %}

## Checking for a match

Perhaps the most basic regex task is to check if a string matches a given regular expression.  You can use `Re2.matches` for this.

{% highlight OCaml %}
{% raw %}
(* Case sensitive *)
let re = Re2.create_exn "apple" in
assert (Re2.matches re "apple pie");
assert (not (Re2.matches re "Apple pie"));;

(* Case insensitive *)
let re =
  let options = { Re2.Options.default with case_sensitive = false } in
  Re2.create_exn ~options "apple" 
in
assert (Re2.matches re "apple pie");
assert (Re2.matches re "Apple pie");;
{% endraw %}
{% endhighlight %}

## Finding matches

To find all matches of a regular expression in a string, you can use the `find_*` functions.

### Find first match

To return the first match in the query string, use `find_first` or `find_first_exn`.  These functions return matched string rather than the underlying `Re2.Match.t`.

{% highlight OCaml %}
{% raw %}
let re = Re2.create_exn "apple" in
  Re2.find_first_exn re "apple pie is made from apples";;
- : string = "apple"

let re = Re2.create_exn "[ab]{2}" in
Re2.find_first_exn re "ababa";;
- : string = "ab"
{% endraw %}
{% endhighlight %}

### Find all matches

While `find_first` returns the first match in a query string, `find_all` and `find_all_exn` return lists of all non-overlapping matches in the query string.

{% highlight OCaml %}
{% raw %}
let re = Re2.create_exn "apple" in
Re2.find_all re "apple pie";;
- : string list Or_error.t = Result.Ok ["apple"]

let re = Re2.create_exn "apple" in
Re2.find_all_exn re "apple pie is made from apples";;
- : string list = ["apple"; "apple"]
{% endraw %}
{% endhighlight %}

Like most of the functions in the `Re2` module, the `find` functions come in both `Or_error.t` returning and exception raising versions.  If the regular expression doesn't match, `find_all` returns a `Result.Error.t` whereas `find_all_exn` raises an exception.

{% highlight OCaml %}
{% raw %}
let re = Re2.create_exn "apple" in
Re2.find_all re "peach pie";;
- : string list Or_error.t =
Result.Error
 ("Re2__Regex.Exceptions.Regex_match_failed(\"apple\")")

let re = Re2.create_exn "apple" in
Re2.find_all_exn re "peach pie";;
Exception: Re2__Regex.Exceptions.Regex_match_failed("apple").
(* ...output omitted... *)
{% endraw %}
{% endhighlight %}

It is important to rember that the `find_all` functions return *non-overlapping* matches.

{% highlight OCaml %}
{% raw %}
let re = Re2.create_exn "[ab]{2}" in
Re2.find_all_exn re "ababa";;
- : string list = ["ab"; "ab"]
{% endraw %}
{% endhighlight %}

## Finding submatches

You will often want to use the simpler interface of `find_submatches` or `find_submatches_exn`.  These return the first match in the query string.  The match is returned as a `string option array`, where the first element is the whole match, and subsequent elements are submatches as defined by any capturing groups.   

{% highlight OCaml %}
{% raw %}
let re = Re2.create_exn "a([bc])([de])" in
Re2.find_submatches_exn re "abdace";;
- : string option array = [|Some "abd"; Some "b"; Some "d"|]
{% endraw %}
{% endhighlight %}

You may wonder why `find_submatches_exn` returns a `string option array` and not simply a `string array`.  `find_submatches_exn` uses `Match.get` [under-the-hood](https://github.com/janestreet/re2/blob/72e01a088b48791aa6387dc3a093d3806122e2bd/src/regex.ml#L307).  Basically, `find_submatches_exn` processes a `Match.t Sequence.t` of matches, calling `get` on each one.  And the `Match.get` function [returns](https://ocaml.janestreet.com/ocaml-core/latest/doc/re2/Re2/Match/index.html#val-get) a `string option`.

This little code snippet will hopefully give you an idea of what's going on.

{% highlight OCaml %}
{% raw %}
let re = Re2.create_exn "a([bc])([de])" in
let match_ = Re2.first_match_exn re "abdace" in
[|
  Re2.Match.get match_ ~sub:(`Index 0);
  Re2.Match.get match_ ~sub:(`Index 1);
  Re2.Match.get match_ ~sub:(`Index 2);
  Re2.Match.get match_ ~sub:(`Index 3);
|]
;;
- : string option array = [| Some "abd"; Some "b"; Some "d"; None |]
{% endraw %}
{% endhighlight %}

If the `Index` you pass to `~sub` is higher than the of capturing groups plus one (e.g., the number returned from `Re2.num_submatches`), `None` is returned.

### More complicated submatch interface

If you want to work with the `Re2.Match.t` directly, you can use functions from the [complicated interface](https://ocaml.janestreet.com/ocaml-core/latest/doc/re2/Re2/index.html#complicated-interface) like [first_match](https://ocaml.janestreet.com/ocaml-core/latest/doc/re2/Re2/index.html#val-first_match) and [get_matches](https://ocaml.janestreet.com/ocaml-core/latest/doc/re2/Re2/index.html#val-get_matches).

If you need to work with submatches of every match in a string rather than just the first, you will want to use `get_matches` or `get_matches_exn`.  Let's try it out with a weird, little example.

Say we have a string made up of chunks.  Each chunk is a number followed by an `A` (for add) or an `S` (for subtract) (e.g., `50A` and `3S`).  The chunk describes an arithmetic operation: `12A` means add 12 to the previous total; `3S` means subtract 3 from the previous total.  

A full string then might look something like this: `10A5S2S3A`, which represents the following sequence of operations: `0 + 10 - 5 - 2 + 3`.

One way to solve this little problem using regexes and the `get_matches` function.  Let's see how it might go.

{% highlight OCaml %}
{% raw %}
let total =
  let s = "10A5S2S3A" in
  (* Make the regex *)
  let re = Re2.create_exn "([0-9]*)([AS])" in
  (* Get a Match.t list *)
  let matches = Re2.get_matches_exn re s in
  (* Fold over the matches to get the total. *)
  List.fold matches ~init:0 ~f:(fun total m ->
      (* The first capturing group is the "count". *)
      let number = Int.of_string @@ Re2.Match.get_exn m ~sub:(`Index 1) in
      (* The second capturing group represents the operation. *)
      match Re2.Match.get_exn m ~sub:(`Index 2) with
      | "A" -> total + number
      | "S" -> total - number
      | _ -> assert false)
;;

assert (total = 0 + 10 - 5 - 2 + 3);;
{% endraw %}
{% endhighlight %}

*Note: This weird format is actually loosely based on the [CIGAR](https://en.wikipedia.org/wiki/Sequence_alignment#Representations) strings found in [SAM files](http://samtools.github.io/hts-specs/SAMv1.pdf) describing [biological sequence alignments](https://en.wikipedia.org/wiki/Sequence_alignment).*

### Controlling submatches

In the last two examples, we used the `sub` argument along with a polymorphic variant to select capture groups.  Let's take a closer look at the type used for that.

To select submatches, we use [id_t](https://ocaml.janestreet.com/ocaml-core/latest/doc/re2/Re2/index.html#type-id_t), which looks like this:

{% highlight OCaml %}
{% raw %}
type id_t = [ `Index of int | `Name of string ]
{% endraw %}
{% endhighlight %}

This type is used to refer to submatches.  E.g., `` ` Index 1`` would be the result of first capturing group, `` ` Index 2`` the 2nd, etc.  Remember that  `` ` Index 0`` refers to the whole match.  

In addition to referring to submatches/capturing groups by index, you can refer to them by name.

{% highlight OCaml %}
{% raw %}
let re = Re2.create_exn "a(?P<second_letter>[bc])" in
let m = Re2.first_match_exn re "abc" in
let x = Re2.Match.get_exn m ~sub:(`Name "second_letter") in
let y = Re2.Match.get_exn m ~sub:(`Index 1) in
assert String.(x = y);;
{% endraw %}
{% endhighlight %}

When using a complicated regular expression with multiple capturing groups, it is often less error prone to use named submatches rather than numbered ones.

*Note:  It is not a compile-error to try an access a capturing group that doesn't exist in the regular expression.  Depending on the function, you may get `None` or raise an exception.*

### Using `id_t` to control match efficiency

Many of the regex matching functions take a `?sub:id_t` argument.

In some cases, you can increase the efficiency of matching by restricting the number of submatches.  If you only care about whether a pattern matches, and not about submatches, you could pass in ``~sub:(` Index -1)`` to many of the above functions.  

You can get increasingly more information by increasing the `n` to the index.

{% highlight OCaml %}
{% raw %}
(* Get only the whole match. *)
~sub:(`Index 0)

(* Get the whole match and first submatch. *)
~sub:(`Index 1)
{% endraw %}
{% endhighlight %}

[This section](https://ocaml.janestreet.com/ocaml-core/latest/doc/re2/Re2/index.html#type-id_t) of the documentation has more info on how specifying the `sub` argument can have an impact on regex performance, and which functions are affected by its usage.

## Splitting strings

Another common regex task is splitting an input string based on a regular expression pattern.  `Re2` provides the `split` function for this purpose.

{% highlight OCaml %}
{% raw %}
let re = Re2.create_exn "[.,! ]+" in
Re2.split re "Hello, world! I like pie.";;
- : string list = ["Hello"; "world"; "I"; "like"; "pie"; ""]
{% endraw %}
{% endhighlight %}

If you need to include the actual matches in the output, you can.  Passing `~include_matches:true` ensures the "separators" are in there with the rest of the output.

{% highlight OCaml %}
{% raw %}
let re = Re2.create_exn "[.,! ]+" in
Re2.split ~include_matches:true re "Hello, world! I like pie.";;
- : string list =
["Hello"; ", "; "world"; "! "; "I"; " "; "like"; " "; "pie"; "."; ""]
{% endraw %}
{% endhighlight %}

Just be aware of that final empty string at the end!

You can also limit the number of matches with the `max` argument.  You could use this to get the first value separated from the remaining values in a string of tab-separated values, for example.

{% highlight OCaml %}
{% raw %}
let re = Re2.create_exn "\t" in
Re2.split ~max:1 re "apple\tpie\tis\tgood";;
- : string list = ["apple"; "pie\tis\tgood"]
{% endraw %}
{% endhighlight %}

If the regular expression has no matches in the query string, then a one element list is returned.

{% highlight OCaml %}
{% raw %}
let re = Re2.create_exn "\t" in
Re2.split ~max:1 re "apple pie is good";;
- : string list = ["apple pie is good"]
{% endraw %}
{% endhighlight %}

## Replacing

### Using `rewrite`

The simpler interface for regex replacement consists of the `rewrite` and `rewrite_exn` functions.  The `template` argument defines how you want to replace any matches in the query string.  In this case, we replace any matches with a capital A.

{% highlight OCaml %}
{% raw %}
let re = Re2.create_exn "a" in
Re2.rewrite_exn re ~template:"A" "apple peach";;
- : string = "Apple peAch"
{% endraw %}
{% endhighlight %}

You can reference the submatches in the template string using the syntax `\\n`.  Check it out.

{% highlight OCaml %}
{% raw %}
let re = Re2.create_exn "([ae])" in
Re2.rewrite_exn re ~template:"( \\1 )" "apple peach";;
- : string = "( a )ppl( e ) p( e )( a )ch"
{% endraw %}
{% endhighlight %}

If you have multiple submatches, just keep referring to them in the same way: `\\1 ... \\2 ...` etc.

If you need to check if your rewrite template is valid before running `rewrite`, use `valid_rewrite_template` function.

{% highlight OCaml %}
{% raw %}
let re = Re2.create_exn "([ae])([io])([uy])" in
let template = "\\3 - \\2 - \\1" in
Re2.valid_rewrite_template re ~template;;
- : bool = true
{% endraw %}
{% endhighlight %}

### Using `replace`

The `re2` library also provides more powerful replacing functions:  `replace` and `replace_exn`.  You can use them if you need direct access to the `Match.t`.

Here is a silly example that picks a different replacement value depending on the match.

{% highlight OCaml %}
{% raw %}
let re = Re2.create_exn "([ae])" in
Re2.replace_exn re "apple peach" ~f:(fun m ->
  match Re2.Match.get_exn m ~sub:(`Index 1) with
  | "a" -> "u"
  | "e" -> "o"
  | _ -> assert false)
;;
- : string = "upplo pouch"
{% endraw %}
{% endhighlight %}

While the `replace` function is more complicated than `rewrite`, it gives you more control and has a few [other options](https://ocaml.janestreet.com/ocaml-core/latest/doc/re2/Re2/index.html#val-replace) you may find useful.

## Miscellaneous info

### Escaping strings for regular expressions

Properly escaping regular expressions can sometimes be tricky, especially if you want to avoid illegal backslash characters in your strings.

`Re2` provides a function `escape` that escapes its input in such a way that if you create a regex from the resulting escaped string, it would match the original string.  Here's how it works.

{% highlight OCaml %}
{% raw %}
Re2.escape "Apple. (Pie)!!";;
- : string = "Apple\\.\\ \\(Pie\\)\\!\\!"

Re2.matches
  (Re2.create_exn @@ Re2.escape "Apple. (Pie)!!")
  "Apple. (Pie)!!";;
- : bool = true
{% endraw %}
{% endhighlight %}

Depending on how many special characters are in the string you use to build the regex, escaping can be pretty noisy!  In these cases, `escape` is especially useful.

### Infix matching operator

If you're feeling nostalgic for Perl, feel free to use the `=~` infix operator!

{% highlight OCaml %}
{% raw %}
let re = Re2.create_exn "ab";;

Re2.Infix.("abc" =~ re);;
- : bool = true

(* Let's get crazy and open the module! *)
open Re2.Infix;;

"abc" =~ re;;
- : bool = true
{% endraw %}
{% endhighlight %}

### "Precompiling" your regular expressions

Unless you have a good reason not to, you will probably want to create your regular expression outside of the function that will be using it.

To see why, let's check out this little benchmark program that compares two functions.  The first one reuses a regex that is created outside of the function, whereas the second one creates a new regex each time the function is called.

*Note:  This benchmark program uses Jane Street's [core_bench](https://github.com/janestreet/core_bench) micro-benchmarking library.*

{% highlight OCaml %}
{% raw %}
open! Core
open! Core_bench

let re = Re2.create_exn "a([bc])"

let find re s = Re2.find_first_exn re s
let find' s = Re2.find_first_exn (Re2.create_exn "a([bc])") s

let () =
  Command.run
    (Bench.make_command
       [
         Bench.Test.create ~name:"outside" (fun () ->
             find re "abcabcabc");
         Bench.Test.create ~name:"inside" (fun () ->
             find' "abcabcabc");
       ])
{% endraw %}
{% endhighlight %}

| Name    |    Time/Run | mWd/Run | Percentage |
|---------|------------:|--------:|-----------:|
| outside |   272.60 ns |  2.00 w |      3.74% |
| inside  | 7_281.55 ns | 91.00 w |    100.00% |

As you can see, reusing a regex rather than creating a new one each time a function is called makes a big difference in this benchmark.  Keep in mind that this is a micro-benchmark, and that this difference may not be that important to the run time of your program as a whole.  That said, if you had the slow version of the above function in a hot loop, it could really be wasting a lot of CPU cycles.

## Wrap up

Hopefully this overview helps you get started with using `re2`!

To get more info about using `re2`, check out the [API docs](https://ocaml.janestreet.com/ocaml-core/latest/doc/re2/Re2/index.html).  Additionally, the `re2` [source code](https://github.com/janestreet/re2/tree/master/src) is quite readable.  I encourage you to take a look at how the functions are defined--it may help clear up any additional questions you have!
