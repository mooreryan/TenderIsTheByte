---
layout: post
title: "Learn Rust: Parsing command line arguments with StructOpt"
author: ryan
description: A beginners guide to using StructOpt for parsing command line arguments.
categories: blog
image: "/assets/img/posts/learn_rust_cli_args/learn_rust_cli_args.png"
---

Lately, I've been learning [Rust](https://www.rust-lang.org/).  Like most people, I started out by reading through [The Rust Programming Language](https://doc.rust-lang.org/book/) (aka, the book), and working through the examples in [Rust By Example](https://doc.rust-lang.org/stable/rust-by-example/).  These are awesome resources, but I tend to get bored after a little while if I'm not using what I've learned to try and make some cool stuff.  

Being a person who does a lot of [bioinformatics](https://en.wikipedia.org/wiki/Bioinformatics), I use command line apps pretty much all day, every day.  Given that, the programs that I've been making are little command line apps to do various (semi) useful tasks.  A key part of these apps is good handling of command line arguments.  As it turns out, Rust has great support for this with the [clap](https://docs.rs/clap/) and [StructOpt](https://docs.rs/structopt) libraries.  So today, we are going to go over how to use StructOpt to parse command line arguments.  As you will see, it makes parsing command line arguments a breeze!

*Note: This post is definitely aimed more at Rust beginners (like myself!), but not necessarily beginners to coding in general.  I'm going to go into a lot of detail about Rust, including many of the things that I needed to look up and understand in my own learning process.  If you just want to see the final result, [skip down to the bottom](#wrapping-up).*

## Setting up a new project

If you don't already have a working installation of Rust on your computer, check out [this help page](https://www.rust-lang.org/tools/install) on the Rust website to get set up.

First, let's set up a new project with [cargo](https://doc.rust-lang.org/stable/cargo/), Rust's package manager.

{% highlight text %}
{% raw %}
$ cargo new parse_cli_args
     Created binary (application) 'parse_cli_args' package
{% endraw %}
{% endhighlight %}

This command makes a new binary program.  Let's check out what Cargo generated using [tree](http://mama.indstate.edu/users/ice/tree/), a program for recursive directory listing.

{% highlight text %}
{% raw %}
$ cd parse_cli_args
$ tree
.
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
{% endraw %}
{% endhighlight %}

Cargo made a couple of files for us: `Cargo.toml` and `src/main.rs`.  

### The manifest file

The `Cargo.toml` file is a manifest that contains metadata that Cargo uses to compile the package.  It is written in [TOML](https://github.com/toml-lang/toml), an easy-to-read format for config files.

Check out your `Cargo.toml` file, and you should see something like this:

{% highlight toml %}
{% raw %}
[package]
name = "parse_cli_args"
version = "0.1.0"
authors = ["Your Name <your@email.com>"]
edition = "2018"

[dependencies]
{% endraw %}
{% endhighlight %}

The `[dependencies]` section is where you can specify [crates](https://crates.io) that your project depends on.  Crates are basically packages made by other members of the rust community that you can use to [bring in all sorts of cool functionality](https://doc.rust-lang.org/stable/book/ch02-00-guessing-game-tutorial.html#using-a-crate-to-get-more-functionality) to your programs.

### The main file

The other file made by the `cargo new` command is `src/main.rs`.  It looks like this:

{% highlight rust %}
{% raw %}
fn main() {
    println!("Hello, world!");
}
{% endraw %}
{% endhighlight %}

For a simple project like this one, all of our code will live here.  In a real project, it would be better to worry more about [separating and organizing code](https://doc.rust-lang.org/book/ch12-03-improving-error-handling-and-modularity.html#separation-of-concerns-for-binary-projects), but for now, we can keep things simple by keeping our code in this file.

*To learn more about using Cargo to manage Rust packages, check out the [Cargo book](https://doc.rust-lang.org/stable/cargo/getting-started/first-steps.html).*

## Defining a struct to hold command line arguments

To parse command line arguments, we will be using [StructOpt](https://github.com/TeXitoi/structopt).  StructOpt is a [crate](https://docs.rs/structopt/0.2.15/structopt/) that builds on [clap](https://crates.io/crates/clap) (a popular command line argument parsing library) and lets you [parse command line arguments](https://rust-lang-nursery.github.io/cli-wg/tutorial/cli-args.html?highlight=structopt#parsing-cli-arguments-with-structopt) by defining a [struct](https://doc.rust-lang.org/book/ch05-01-defining-structs.html#defining-and-instantiating-structs).  StructOpt makes things really simple, but to use it we need to learn just a little bit about [structs](https://doc.rust-lang.org/book/ch05-00-structs.html) and [traits](https://doc.rust-lang.org/book/ch10-02-traits.html#traits-defining-shared-behavior).

Before we can use the StructOpt crate, first add the following to your `Cargo.toml` file.

{% highlight toml %}
{% raw %}
[dependencies]
structopt = "0.2.15"
{% endraw %}
{% endhighlight %}

Now let's learn a bit about structs.

### Structs

In Rust, a [struct](https://doc.rust-lang.org/book/ch05-00-structs.html) is a custom data type used to join related values together into a meaningful group.  Additionally, methods and functions can be associated with a struct to act on the struct's data.  By defining structs and the methods associated with them, we can create new types specific to our program's domain and take advantage of the Rust compiler's type checking.

So a struct is used to group together related values.  When making a struct, we should think about what kind of data groupings make sense for our program.  When used this way, structs [add extra meaning to our data](https://doc.rust-lang.org/stable/book/ch05-02-example-structs.html#refactoring-with-structs-adding-more-meaning) by labeling it.  For example, instead of having two separate variables for holding the height and width of a rectangle, you could make a `Rectangle` struct to bind those two pieces of data together.  This makes it clear that the two pieces of data are related to one another.  Then, rather than manage a height and a width variable separately for each rectangle we want to work with, we can just use a single instance of the `Rectangle` struct.

Okay, let's add a struct to hold the command line args!  To do this, we first need to bring StructOpt into scope with the [use](https://doc.rust-lang.org/book/ch07-02-modules-and-use-to-control-scope-and-privacy.html?highlight=use,keyword#the--use--keyword-to-bring-paths-into-a-scope) keyword.  Put the following at the top of the `src/main.rs` file:

{% highlight rust %}
{% raw %}
use structopt::StructOpt;
{% endraw %}
{% endhighlight %}

To actually [define](https://doc.rust-lang.org/book/ch05-01-defining-structs.html) the struct, we use the `struct` keyword.  Let's take a look at how this is done.  

{% highlight rust %}
{% raw %}
// Define a struct called Opts
struct Opts {
    // Make a field called infile, with type PathBuf
    infile: PathBuf,

    // Make a field called outfile, with type PathBuf
    outfile: PathBuf,
}
{% endraw %}
{% endhighlight %}

Now we've defined a struct called `Opts`, with fields `infile` and `outfile`, both of which have the [PathBuf](https://doc.rust-lang.org/std/path/struct.PathBuf.html) type, which is used to hold file paths.  The [path](https://doc.rust-lang.org/std/path/index.html) module provides lots of methods for working with paths using the local platform's path syntax (e.g., using `/` as the path separator on a Mac).  To use `PathBuf`, we will need to bring it into scope as well with `use std::path::PathBuf`.  The top of your `src/main.rs` file should now look like this:

{% highlight rust %}
{% raw %}
use std::path::PathBuf;
use structopt::StructOpt;
{% endraw %}
{% endhighlight %}

Now that we've defined the struct, let's see how to use it!  We can create an instance of our struct by specifying concrete values for each of its fields like this:

{% highlight rust %}
{% raw %}
// Assign an instance of Opts to the opts variable.
let opts = Opts {
    // Data to store in the infile field
    infile: PathBuf::from("/home/ryan/infile.txt"),

    // Data to store in the outfile field
    outfile: PathBuf::from("/home/ryan/outfile.txt"),
};
{% endraw %}
{% endhighlight %}

So this creates a new instance of the `Opts` struct with some `PathBuf` data stored in the `infile` and `outfile` fields and stores it in the `opts` variable.

Now that we have an instance of `Opts` created, how do we get values out when we need to use them?  To get the data from a struct, we use a `.` like this: `variable_name.field_name`.  So to get the data in the `infile` field of the struct we just created, we would use `opts.infile`.

If you want to change the value stored in a field you would do something like this `opts.infile = new_value`.  Let's try it out.

{% highlight rust %}
{% raw %}
opts.infile = PathBuf::from("file.txt");
{% endraw %}
{% endhighlight %}

If you were to [compile it](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=00edfef39bc7419c6db50ab984bc18ff), you would get an error!

{% highlight text %}
{% raw %}
error[E0594]: cannot assign to 'opts.infile', as 'opts' is not declared as mutable
  --> src/main.rs:18:5
   |
13 |     let opts = Opts {
   |         ---- help: consider changing this to be mutable: 'mut opts'
...
18 |     opts.infile = PathBuf::from("file.txt");
   |     ^^^^^^^^^^^ cannot assign
{% endraw %}
{% endhighlight %}

Because variables in Rust are [immutable by default](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html#variables-and-mutability), if we wanted to be able to change the values to something else we would need to use the `mut` keyword when defining the struct like this: 

{% highlight rust %}
{% raw %}
let mut opts = Opts {
    infile: PathBuf::from("/home/ryan/infile.txt"),
    outfile: PathBuf::from("/home/ryan/outfile.txt"),
};

opts.infile = PathBuf::from("file.txt");
{% endraw %}
{% endhighlight %}

After adding the `mut` keyword, our program will [successfully compile](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=a1252d5d5879f11c468f44904782b240).

By now, we've learned enough about structs to create and use a simple struct to hold our program's command line arguments!  To actually do this, we need to add a couple more things to the `Opts` struct so that StructOpt knows how to use it to parse command line arguments.  For this, we will need to learn a little bit about Traits.

### Traits

[Traits](https://doc.rust-lang.org/book/ch10-02-traits.html#traits-defining-shared-behavior) are used to tell the Rust compiler about what kind of functionality a type has.  For example, [Display](https://doc.rust-lang.org/std/fmt/trait.Display.html) is a trait used for formatting and printing things using macros like `println!` and `format!`.  Any type that implements the `Display` trait can be formatted or printed using curly brackets (`{}`) in one of the aformentioned macros (e.g., you can print the number `47` (`println!("num = {}", 47)`) because it implements the `Display` trait.)

What happens if we try and print `opts` using curly brackets?

{% highlight rust %}
{% raw %}
println!("opts: {}", opts);
{% endraw %}
{% endhighlight %}

If you [compiled that](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=6c7d2a58c560f60723283bb02f426aeb), you would get the following error

{% highlight text %}
{% raw %}
error[E0277]: 'Opts' doesn't implement 'std::fmt::Display'
  --> src/main.rs:25:20
   |
25 |     println!("{}", opts);
   |                    ^^^^ `Opts` cannot be formatted with the default formatter
   |
   = help: the trait 'std::fmt::Display' is not implemented for 'Opts'
   = note: in format strings you may be able to use '{:?}' (or {:#?} for pretty-print) instead
   = note: required by 'std::fmt::Display::fmt'
{% endraw %}
{% endhighlight %}

As you can see, the compiler tells us that an instance of `Opts` cannot be formatted with the default formatter because it doesn't implement `std::fmt::Display`.  The Rust compiler always tries to be helpful so it suggests that we try and print instead using `{:?}`, which tells the `println!` macro to use the `Debug` output format.  Let's try it!

{% highlight rust %}
{% raw %}
println!("opts: {:?}", opts);
{% endraw %}
{% endhighlight %}

This time [we get a different error](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=4b22cd843b41a96f962b0f5e5d732661):

{% highlight text %}
{% raw %}
error[E0277]: 'Opts' doesn't implement 'std::fmt::Debug'
  --> src/main.rs:25:22
   |
25 |     println!("{:?}", opts);
   |                      ^^^^ 'Opts' cannot be formatted using '{:?}'
   |
   = help: the trait 'std::fmt::Debug' is not implemented for 'Opts'
   = note: add '#[derive(Debug)]' or manually implement 'std::fmt::Debug'
   = note: required by 'std::fmt::Debug::fmt'
{% endraw %}
{% endhighlight %}

Whoops, `Opts` still can't be formatted!  This time we are missing the `std::fmt::Debug` trait.  The compiler says we can either implement it ourself or add `#[derive(Debug)]` to the code.  

Finally we get to the [derive](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html#appendix-c-derivable-traits) attribute!  The `derive` attribute can be applied to a struct definition to generate code that will implement a trait with a default implementation.  So if we add `#[derive(Debug)]` to the `Opts` struct, it will automatically implement the `Debug` trait, allowing it to be printed using `{:?}`!  Using the `derive` attribute with a trait like `Debug` is a way to "opt-in" to default functionality without having to implement the trait ourselves.

{% highlight rust %}
{% raw %}
#[derive(Debug)]
struct Opts {
    infile: PathBuf,
    outfile: PathBuf,
}
{% endraw %}
{% endhighlight %}

Now when we run `println!("{:?}", opts)`, our program [will print out](https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=dbf29b0e775525f08bf48e21753edc33)

{% highlight text %}
{% raw %}
Opts { infile: "/home/ryan/infile.txt", outfile: "/home/ryan/outfile.txt" }
{% endraw %}
{% endhighlight %}

Now that we know something about `Traits`, we can finally talk about how to use the `Opts` struct with StructOpt.

## Using the `Opts` struct to parse arguments

Before StructOpt can use the `Opts` struct for parsing command line arguments, we need to add StructOpt to the existing `derive` statement and tell StructOpt how to [parse the field types](https://docs.rs/structopt/0.2.15/structopt/#custom-string-parsers).  Let's see how that looks.

{% highlight rust %}
{% raw %}
#[derive(Debug, StructOpt)]
struct Opts {
    #[structopt(parse(from_os_str))]
    infile: PathBuf,

    #[structopt(parse(from_os_str))]
    outfile: PathBuf,
}
{% endraw %}
{% endhighlight %}

The `#[structopt(parse(from_os_str))]` lines tell StructOpt to use a custom string parser.  In this case, it parses the argument from an [OsStr](https://doc.rust-lang.org/std/ffi/struct.OsStr.html), which is a borrowed reference to a string in the operating system's native representation.

Now that we've added the derives and custom parsers to the `Opts` struct, we can use it to automatically parse command line arguments by using the `from_args` method.

{% highlight rust %}
{% raw %}
let opts = Opts::from_args();
{% endraw %}
{% endhighlight %}

The [from_args](https://docs.rs/structopt/0.2.15/structopt/trait.StructOpt.html#method.from_args) method uses the command line arguments to create an instance of `Opts`.  If it fails, it prints the error message and quits the program automatically.  Using StructOpt in this way will automatically generate a `--help` message and give nice error messages if the user does not provide the correct command line arguments to our program.

### Testing out the argument parsing

Let's stop and put together everything we've gone through so far.  Now, our `src/main.rs` file should contain the following code:

{% highlight rust %}
{% raw %}
use std::path::PathBuf;
use structopt::StructOpt;

#[derive(Debug, StructOpt)]
struct Opts {
    #[structopt(parse(from_os_str))]
    infile: PathBuf,

    #[structopt(parse(from_os_str))]
    outfile: PathBuf,
}

fn main() {
    let opts = Opts::from_args();

    println!("{:?}", opts);
}
{% endraw %}
{% endhighlight %}

We can run our program using `cargo run`.  Let's try running it without any arguments.

{% highlight text %}
{% raw %}
$ cargo run
error: The following required arguments were not provided:
    <infile>
    <outfile>

USAGE:
    parse_cli_args <infile> <outfile>

For more information try --help
{% endraw %}
{% endhighlight %}

Nice!  Let's see what the `--help` message looks like.  To pass arguments to `cargo run`, we add them after `--` like this: `cargo run -- --help`.

{% highlight text %}
{% raw %}
$ cargo run -- --help
parse_cli_args 0.1.0
Ryan Moore <moorer@udel.edu>

USAGE:
    parse_cli_args <infile> <outfile>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

ARGS:
    <infile>     
    <outfile>    
{% endraw %}
{% endhighlight %}

We can see that our program takes two required positional arguments `<infile>` and `<outfile>`.  That's pretty easy to use, but what if we wanted to specify the outfile using `-o` or `--outfile`?  We just need to add more attributes in addition to the `parse` attribute we added earlier.  If we want both a short (`-o`) and long (`--outfile`) option for `outfile` we would change `#[structopt(parse(from_os_str))]` to `#[structopt(short, long, parse(from_os_str))]` like this:

{% highlight text %}
{% raw %}
#[structopt(short, long, parse(from_os_str))]
outfile: PathBuf,
{% endraw %}
{% endhighlight %}

Doing so will change the `--help` message to something like this: 

{% highlight text %}
{% raw %}
parse_cli_args 0.1.0
Ryan Moore <moorer@udel.edu>

USAGE:
    parse_cli_args <infile> --outfile <outfile>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -o, --outfile <outfile>    

ARGS:
    <infile>
{% endraw %}
{% endhighlight %}

Finally, what if we wanted the `outfile` argument to be optional?  For example, we might want our program to print its output to a file if `--outfile` is provided, but print to `stdout` if no `outfile` argument is given.  To do that, we need to change the type of the `outfile` field from `PathBuf` to `Option<PathBuf>`.

{% highlight text %}
{% raw %}
#[structopt(short, long, parse(from_os_str))]
outfile: Option<PathBuf>,
{% endraw %}
{% endhighlight %}

Of course, the help message automatically updates to reflect this change.

{% highlight text %}
{% raw %}
$ cargo run -- --help
parse_cli_args 0.1.0
Ryan Moore <moorer@udel.edu>

USAGE:
    parse_cli_args [OPTIONS] <infile>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -o, --outfile <outfile>    

ARGS:
    <infile>    
{% endraw %}
{% endhighlight %}

*Note: The `Option` type is an enum that is used to encode when a value could be something or it could be nothing.  For example, when we pass something to the `outfile` argument, then we would get `Some` value (e.g., `Some(PathBuf)`), but if nothing is passed to the `outfile` argument, we would get `None`.  For a full explanation of the `Option` type, see [this section](https://doc.rust-lang.org/book/ch06-01-defining-an-enum.html#the-option-enum-and-its-advantages-over-null-values) in the Rust book.*

## Wrapping up

To finish up, let's test out the full program.  `src/main.rs` should now look like this:

{% highlight rust %}
{% raw %}
use std::path::PathBuf;
use structopt::StructOpt;

#[derive(Debug, StructOpt)]
struct Opts {
    #[structopt(parse(from_os_str))]
    infile: PathBuf,

    #[structopt(short, long, parse(from_os_str))]
    outfile: Option<PathBuf>,
}

fn main() {
    let opts = Opts::from_args();

    println!("{:?}", opts);
}
{% endraw %}
{% endhighlight %}

Now, let's build our app with `cargo build`, which will create a binary called `parse_cli_args` in the `target/debug` folder.

Finally, let's see what it looks like when we run the program with some actual arguments.

{% highlight rust %}
{% raw %}
$ ./target/debug/parse_cli_args --outfile out.txt in.txt
Opts { infile: "in.txt", outfile: Some("out.txt") }

$ ./target/debug/parse_cli_args in.txt
Opts { infile: "in.txt", outfile: None }
{% endraw %}
{% endhighlight %}

And that's all you need to get started using StructOpt!  Of course, there are a lot more options and settings for parsing command line arguments that we didn't get into here.  To learn more about them, I encourage you to check out the [StructOpt documentation](https://docs.rs/crate/structopt/0.2.15) and the [Rust command line apps book](https://rust-lang-nursery.github.io/cli-wg/tutorial/cli-args.html?highlight=structopt#parsing-command-line-arguments)!