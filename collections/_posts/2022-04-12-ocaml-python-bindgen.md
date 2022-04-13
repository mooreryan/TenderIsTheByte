---
layout: post
title: Generating Python bindings for OCaml with pyml_bindgen
author: ryan
description: This post provides an introduction to using pyml_bindgen, a command line application that generates Python bindings via pyml directly from OCaml value specifications.
categories: blog
image: "/assets/img/posts/ocaml_python_bindgen/ocaml_pyml_bindgen.png"
twitter_share: https://ctt.ac/rB09u
---

`pyml_bindgen` is a command line app that generates Python bindings via [pyml](https://github.com/thierry-martinez/pyml) directly from OCaml value specifications.  While you could write `pyml` bindings by hand, it can get repetitive, especially if you are binding a decent sized Python library.

In this post, I will introduce `pyml_bindgen` and go through a couple of common tasks.

## Install

To get started with `pyml_bindgen`, you will need to install it.  It is available on [opam](https://opam.ocaml.org/packages/pyml_bindgen/) (`opam install pyml_bindgen`).  However, to follow along with this blog, you will need to install from the main branch on GitHub.

{% highlight ocaml %}
{% raw %}
$ git clone --depth 1 https://github.com/mooreryan/ocaml_python_bindgen.git
$ opam install .
{% endraw %}
{% endhighlight %}

_(As of Apr 12, 2022, you will need to install from the main branch rather than opam to follow along with this post.)_

## A simple example

Let's start with a simple example.

### Python code

Here is the Python class that we want to bind (`hobbit.py`).

{% highlight python %}
{% raw %}
class Hobbit:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __str__(self):
        return f'Hobbit -- {self.name}, {self.age}'
{% endraw %}
{% endhighlight %}

As you see, it's pretty simple! It's just the `__init__` method to create the class and the `__str__` method for converting it to a string with the Python `str` or `print` functions.

Here's an example of using it in Python.

{% highlight python %}
{% raw %}
from hobbit import Hobbit
bilbo = Hobbit('Bilbo', 111)
print(bilbo)
#=> Hobbit -- Bilbo, 111
{% endraw %}
{% endhighlight %}

### Write value specifications

To bind Python classes with `pyml_bindgen`, you first need to write value specifications to define the OCaml interface for the Python code we are binding.

To start, we will keep the functions and argument names the same.

{% highlight ocaml %}
{% raw %}
val __init__ : name:string -> age:int -> unit -> t
val __str__ : t -> unit -> string
val name : t -> string
val age : t -> int
{% endraw %}
{% endhighlight %}

There are a couple things call your attention to here:

- I haven't defined `type t` anywhere yet. Depending on the command line arguments you pass to `pyml_bindgen`, it will take care of this for you.
- For the `__init__` function, I have used all named arguments plus the `unit` argument.  The `unit` argument tells `pyml_bindgen` that you are binding a normal Python method or function call (as opposed to a Python attribute or property).
- The `__str__` function takes `t` as the first argument.  Value specifications that start with `t`, will bind to object method calls on the Python side.
- `name` and `age` both take `t` as the first and only argument.  If a value specification takes `t` and nothing else, it binds to the Python attribute of that name.

Save the above in a file called `hobbit.txt`.

### Generate bindings

Now, we're ready to generate the OCaml bindings.

Here's how you would run `pyml_bindgen` for this example.

{% highlight bash %}
{% raw %}
$ pyml_bindgen hobbit.txt hobbit Hobbit \
  --of-pyo-ret-type no_check \
  > hobbit.ml
{% endraw %}
{% endhighlight %}

Let's unpack that.

- The first three arguments are the path to the OCaml value specifications, the name of the Python module we are binding, and the Python class name.
  - Since we named the Python file `hobbit.py`, its module name is `hobbit`.
  - Depending on the directory structure you're using, this may change.
- `--of-pyo-ret-type` specifies the return type for functions that generate Python objects.
  - Using `no_check` means the generated functions will assume the Python object is the correct type.
  - You can also use `option` and `or_error` as well.
- The output is redirected to a file called `hobbit.ml`.  Thus, our generated code will be in a module called `Hobbit`.
- We did not tell `pyml_bindgen` that it should generate a full module with a signature, so it will just write the implementation.
  - In this example it is fine, but you will often want to generate the module and signature, so that your types will be abstract.
  - For example, you could use `--caml-module Hobbit --split-caml-module` to generate both an `ml` and `mli` file.
- If you look at the generated code, it will be kind of messy.  I usually run the output through `ocamlformat` if I need to edit the output, or check the generated code into version control or something like that.

### Test it out

Now we can make a program to test it out.  Don't forget to call [initialize](https://github.com/thierry-martinez/pyml#getting-started) before running the rest of your code!

{% highlight ocaml %}
{% raw %}
let () = Py.initialize ()

let bilbo = Hobbit.__init__ ~name:"Bilbo" ~age:111 ()

let () =
  assert ("Hobbit -- Bilbo, 111" = Hobbit.__str__ bilbo ());
  assert ("Bilbo" = Hobbit.name bilbo);
  assert (111 = Hobbit.age bilbo)
{% endraw %}
{% endhighlight %}

Since we didn't generate a signature to go with our implementation, the type of the value returned by `Hobbit.__init__` will be `Pytypes.pyobject`.  In this way, we can pass any `pyobject` to the `Hobbit.__str__` function.  Let's see.

{% highlight ocaml %}
{% raw %}
let x = Py.Int.of_int 1234

let () = print_endline @@ Hobbit.__str__ x ()
{% endraw %}
{% endhighlight %}

If you run that, it will print `1234`.  Huh?  Well, if you look at the generated code for the `Hobbit.__str__` function, it looks something like this:

{% highlight ocaml %}
{% raw %}
let __str__ t () =
  let callable = Py.Object.find_attr_string t "__str__" in
  let kwargs = filter_opt [] in
  Py.String.to_string
  @@ Py.Callable.to_function_with_keywords callable [||] kwargs
{% endraw %}
{% endhighlight %}

Without going into too much detail, essentially all it is doing is calling the `__str__` method on the Python object passed in.  While this is fine on the Python side, it doesn't work the way we might want it to on the OCaml side.  Ideally, we only want the `Hobbit` module functions to work on values of type `Hobbit.t`.

### Generating abstract types

If we were writing the bindings by hand, we would make `Hobbit.t` abstract.  With `pyml_bindgen`, we can do that using the `--caml-module` option.

{% highlight bash %}
{% raw %}
$ pyml_bindgen hobbit_specs.txt hobbit Hobbit \
  --of-pyo-ret-type no_check \
  --caml-module Hobbit \
  --split-caml-module . \
  > hobbit.ml
{% endraw %}
{% endhighlight %}

Notice that I also used `--split-caml-module .` which tells `pyml_bindgen` to split the implementation and signature into separate `ml` and `mli` files, and to put the output in the directory in which the command is run.  You can pass in whatever directory you want to this option.

Now if we tried something like this:

{% highlight ocaml %}
{% raw %}
let x = Py.Int.of_int 1234

let () = print_endline @@ Hobbit.__str__ x ()
{% endraw %}
{% endhighlight %}

It would be a compile-time error.

## Controlling the bindings

Let's clean up this example a little bit.

### Using different function names

While `__init__` and `__str__` are fine for OCaml function names, they don't feel quite right.  `pyml_bindgen` lets you bind Python functions to different names on the OCaml side using [attributes](https://ocaml.org/manual/attributes.html) on the value specifications.  To bind to a different function name, we use the `py_fun_name` attribute.  Check it out.

{% highlight ocaml %}
{% raw %}
val create : name:string -> age:int -> unit -> t
[@@py_fun_name __init__]

val to_string : t -> unit -> string
[@@py_fun_name __str__]
{% endraw %}
{% endhighlight %}

We bind the `__init__` function to an OCaml function called `create`, and the Python function `__str__` to the OCaml function `to_string`.  That's much more natural!

As you can see, the syntax is like this: `[@@attr-id attr-payload]`.  In this case, the attribute id is `py_fun_name` and the payload is the name of the Python function that we want to bind.  Put another way, the attribute payload should be the name of the function as it is defined in the Python library you are binding to (i.e., `__init__` is the name of the function on the Python side, not `create`).

Putting it together, you get `[@@py_fun_name __init__]` for the Python `__init__` function and `[@@py_fun_name __str__]` for the Python `__str__` function.

### Using different argument names

The other available attribute is `py_arg_name`.  With this, we can bind arguments to different names on the OCaml and Python sides.  This can be useful in situations in which Python argument names use reserved OCaml keywords, or simply to make the generated API feel more natural for use in OCaml.

For example, you may have a Python function that has an argument name `method`.

{% highlight python %}
{% raw %}
def cluster(method='ward'):
    ...
{% endraw %}
{% endhighlight %}

Since `method` is a [reserved keyword](https://ocaml.org/manual/lex.html#sss:keywords) in OCaml, we can't use it directly.  Instead, we want to name it `method_` in our OCaml code.

{% highlight ocaml %}
{% raw %}
val cluster : method_:string -> ...
[@@py_arg_name method_ method]
{% endraw %}
{% endhighlight %}

In this case, the payload is two items: the first is the argument name on the OCaml side, and the second is the argument name on the Python side.

Note that in cases in which you need [multiple attributes](https://github.com/mooreryan/ocaml_python_bindgen/tree/main/examples/attributes#multiple-attributes) per specification, they must be placed one per line.  (This is a `pyml_bindgen` specific restriction.)  E.g., something like this:

{% highlight ocaml %}
{% raw %}
val run_clustering : method_:string -> ...
[@@py_fun_name cluster]
[@@py_arg_name method_ method]
{% endraw %}
{% endhighlight %}

This will bind the OCaml function `run_clustering` to the corresponding Python function `cluster`.

## Binding cyclic Python classes

Often you will need to bind Python classes that refer to each other.  One way to bind these is to use [recursive modules](https://ocaml.org/manual/recursivemodules.html).  Let's update our Hobbit example to show how you can do this in `pyml_bindgen`.

{% highlight python %}
{% raw %}
class Hobbit:
    def __init__(self, name, age):
        self.name = name
        self.age = age
        self.house = None

    def __str__(self):
        return f'Hobbit -- {self.name}, age: {self.age}, house: {self.house.name}'

    def buy_house(self, house):
        self.house = house
        self.house.owner = self

class House:
    def __init__(self, name):
        self.name = name
        self.owner = None

    def __str__(self):
        return f'House -- {self.name}, owner: {self.owner.name}'
{% endraw %}
{% endhighlight %}

So this is a pretty silly example, but it's just to illustrate the point.  In this case, a `Hobbit` can own a `House` and a `House` can have a `Hobbit` for an owner.

To bind these classes, I will use the `gen_multi` and `combine_rec_modules` helper programs that come with `pyml_bindgen`.

### gen_multi

`gen_multi` is a wrapper script that runs `pyml_bindgen` multiple times to generate multiple OCaml modules in one go.  It takes a tsv file specifying the same set of options that you would pass in to `pyml_bindgen` if you used it directly.

Assume this is in a file called `gen_multi_cli.tsv`.

{:.scroll}
| signatures | py_module | py_class | associated_with | caml_module | split_caml_module | embed_python_source | of_pyo_ret_type |
|------------|-----------|----------|-----------------|-------------|-------------------|---------------------|-----------------|
| hobbit.txt | hobbit    | Hobbit   | class           | Hobbit      | NA                | hobbit.py           | no_check        |
| house.txt  | house     | House    | class           | House       | NA                | house.py            | no_check        |

The order of the columns must as shown above.  _(For more info on each of these options, run `pyml_bindgen --help`.)_

You will see that we refer to `hobbit.txt` and `house.txt`.  These are the value specifications for each of the Python classes.  Here are there contents.

`hobbit.txt`

{% highlight ocaml %}
{% raw %}
val create : name:string -> age:int -> unit -> t
[@@py_fun_name __init__]

val to_string : t -> unit -> string
[@@py_fun_name __str__]

val buy_house : t -> house:House.t -> unit -> unit
{% endraw %}
{% endhighlight %}

`house.txt`

{% highlight ocaml %}
{% raw %}
val create : name:string -> unit -> t
[@@py_fun_name __init__]

val to_string : t -> unit -> string
[@@py_fun_name __str__]
{% endraw %}
{% endhighlight %}

### combine_rec_modules

`combine_rec_modules` takes a file of OCaml modules and "converts" them into recursive modules.  It does this using a simple text transformation.

Often you will want to pipe the output of `gen_multi` directly into `combine_rec_modules`.

### Generate the modules & test it out

Now let's see it in action.

{% highlight bash %}
{% raw %}
$ gen_multi gen_multi_cli.tsv | combine_rec_modules /dev/stdin > lib.ml
{% endraw %}
{% endhighlight %}

We put that in a module called `Lib`.  And here is how we might use that.

{% highlight ocaml %}
{% raw %}
open Lib

let () = Py.initialize ()

let bilbo = Hobbit.create ~name:"Bilbo" ~age:111 ()

let bag_end = House.create ~name:"Bag End" ()

let () = Hobbit.buy_house bilbo ~house:bag_end ()

let () =
  assert (
    "Hobbit -- Bilbo, age: 111, house: Bag End" = Hobbit.to_string bilbo ())
{% endraw %}
{% endhighlight %}

## Other stuff

Let me mention a couple of other things before we go...

- In this post we ran `pyml_bindgen` (or its helper scripts) manually, it's not too hard to set up Dune [rules](https://dune.readthedocs.io/en/stable/dune-files.html#rule) to automatically generate bindings.  See the `dune` files in the [example](https://github.com/mooreryan/ocaml_python_bindgen/tree/main/examples) directory on the `pyml_bindgen` GitHub for more information.
- While I only showed how to bind to Python classes, you can also bind to functions associated with modules rather than with classes.
- Another cool feature is that you can embed Python source code directly into your generated OCaml modules.  See [here](https://github.com/mooreryan/ocaml_python_bindgen/tree/main/examples/embedding_python_source) for more details.

## Wrap-up

`pyml_bindgen` is a command line app for generating Python bindings using pyml.  It makes incorporating Python libraries into your OCaml projects as easy as writing regular OCaml value specifications.

To get more information on setting up and using `pyml_bindgen`, including ideas on how to structure your projects, check out the [examples](https://github.com/mooreryan/ocaml_python_bindgen/tree/main/examples), [tests](https://github.com/mooreryan/ocaml_python_bindgen/tree/main/test), and [docs](https://mooreryan.github.io/ocaml_python_bindgen/).
