---
layout: post
title: "Using Sass in Clojure Ring apps"
author: ryan
description: So you want to use Sass instead of plain CSS in your Clojure Ring web app, but you're not sure how to get it set up?  No problem!  Let's walk through it together.
categories: blog
image: "/assets/img/posts/sass_clojure_ring_apps/sass-plus-clojure.png"
twitter_share: https://ctt.ac/BN2ke
---

So you want to use Sass instead of plain CSS in your Clojure Ring web app, but you're not sure how to get it set up?  No problem!  Let's walk through it together.

{% include post_img_border.html path='sass_clojure_ring_apps/sass-plus-clojure.svg' caption='Sass + Clojure' %}

According to the [official website](https://sass-lang.com), Sass is CSS with superpowers.  It's a stable and powerful CSS extension language with two different syntaxes, Sass, the original, and Sassy CSS (SCSS), a newer syntax that is a superset of CSS.  If you're not too familiar with Sass, check out [this tutorial](https://sass-lang.com/guide).

{::options parse_block_html="true" /}

<div class="post-toc">

{:.post-toc--header}
#### Contents

- [Install the sass binary](#install-the-sass-binary)
- [Set up a toy Clojure Ring app](#set-up-a-toy-clojure-ring-app)
- [Set up SCSS](#set-up-scss)

</div>

{::options parse_block_html="false" /}

## Install the sass binary

First off, you're going to need to [install](https://sass-lang.com/install) a Sass preprocessor.  To use Sass, you write `.sass` (if you're using the Sass syntax) or `.scss` (if you're using the Sassy CSS syntax) files and then compile them to plain ol' CSS using one of the many Sass compilers.

I'm using a Mac, so installing Sass is as easy as running this [Homebrew](https://brew.sh/) command:

{% highlight bash %}
{% raw %}
$ brew install sass/sass/sass
{% endraw %}
{% endhighlight %}

Now, one tricky thing is that Sass has a lot of [different implementations](https://sass-lang.com/implementation).  Sass was originally written in Ruby, so there's the now deprecated [Ruby Sass](https://sass-lang.com/ruby-sass).  Additionally, there is [LibSass](https://sass-lang.com/libsass), a C/C++ port of the Sass engine, [Dart Sass](https://sass-lang.com/dart-sass), which compiles to JavaScript, and many others.  It really doesn't matter which one you use as long as you've got one of them installed.

*For the rest of the tutorial, I'm going to assume that you've got Dart Sass, as that is the primary Sass implementation.  It's binary is called `sass`.*

## Set up a toy Clojure Ring app

To show you how to get Sassy with your CSS, let's start by setting up an example Clojure Ring app.  Assuming that you already have [Leiningen](https://leiningen.org) installed, run this in your favorite terminal app:

{% highlight bash %}
{% raw %}
$ lein new sassy-clj && cd sassy-clj
{% endraw %}
{% endhighlight %}

### Fix project.clj

Alright, now we can make sure the `project.clj` file is set up nice and neat.  To do that, we're going to need to change a couple of different things in the `defproject` macro.

* Add the Ring libraries to the `:dependencies` vector.
* Set up the Ring handler.
* Add in the [lein-ring](https://github.com/weavejester/lein-ring) and [lein-scss](https://github.com/bluegray/lein-scss) plugins.

All together, it should look something like this.  (I've added comments to show the things that you need to add.)

{% highlight clj %}
{% raw %}
(defproject sassy-clj "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "EPL-2.0 OR GPL-2.0-or-later WITH Classpath-exception-2.0"
            :url "https://www.eclipse.org/legal/epl-2.0/"}
  :dependencies [[org.clojure/clojure "1.10.0"]
                 ;; Include the Ring libraries.
                 [ring "1.8.0"]
                 ;; Include some nice app defaults.
                 [ring/ring-defaults "0.3.2"]]
  :repl-options {:init-ns sassy-clj.core}

  ;; Include the needed plugins.
  :plugins [[lein-ring "0.12.5"]
            [lein-scss "0.3.0"]]

  ;; Set up the Ring server handler.
  :ring {:handler sassy-clj.core/app})
{% endraw %}
{% endhighlight %}

After you're made those changes, don't forget to run `lein deps` in your project's source directory to download the needed dependencies.

### Set up the assets directories

Now then, let's make some folders to hold the HTML, SCSS, and generated CSS files.

{% highlight bash %}
{% raw %}
$ mkdir -p resources/html resources/scss resources/public/css
{% endraw %}
{% endhighlight %}

The `resources/scss` directory is where we'll keep the `*.scss` files that we'll actually be editing, and the `resources/public/css` directory will hold all of the generated CSS files.  If you guessed that `resources/html` is where we will keep our HTML files, you guessed right!

### Set up a sweet home page

Now let's make a tiny little homepage for our app.  First, make a new file called `resources/html/home.html` and put this in it.

{% highlight html %}
{% raw %}
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8"/>
    <link rel="stylesheet" href="css/main.css">
    <title>Sassy CSS for Clojure Ring Apps</title>
  </head>
  <body>
    <h1>Sassy Clj</h1>
    <p>Let's use Sassy CSS in a Clojure Ring app!</p>
  </body>
</html>
{% endraw %}
{% endhighlight %}

You can see that we've linked to the `css/main.css` stylesheet.  We won't be writing this by hand, rather we will set up Leiningen so that it will be generated automatically!

Now, edit the `sassy-clj.core` namespace found in `src/sassy_clj/core.clj` like so:

{% highlight clj %}
{% raw %}
(ns sassy-clj.core
  (:require [ring.middleware.defaults :refer [wrap-defaults site-defaults]]
            [ring.util.response :as response]))
{% endraw %}
{% endhighlight %}

This will let us use the `site-defaults`, which [among other things](https://github.com/ring-clojure/ring-defaults#customizing), will allow serving static assets in the `resources/public` folder.  Also, we want to use Ring's response helpers.

Next, set up a basic `handler` function to [respond](https://github.com/ring-clojure/ring/wiki/Concepts#responses) to [requests](https://github.com/ring-clojure/ring/wiki/Concepts#requests).  

{% highlight clj %}
{% raw %}
(defn handler [request]
  (-> (response/resource-response "/html/home.html")
      (response/content-type "text/html")))
{% endraw %}
{% endhighlight %}

This function will respond to all requests with our homepage.

Finally, we define an `app` var to be our app's main handler.  This matches what we specified in the `project.clj` file.

{% highlight clj %}
{% raw %}
(def app
  (wrap-defaults handler site-defaults))
{% endraw %}
{% endhighlight %}

You'll notice that I've used the `wrap-defaults` [middleware](https://github.com/ring-clojure/ring/wiki/Concepts#middleware) function around the handler we wrote.  This is to get those sweet `site-defaults` in the response.

### Start up a development server

By now we should have a working app.  Let's check it out!  To do so, start up the server like so:

{% highlight bash %}
{% raw %}
$ lein ring server-headless
{% endraw %}
{% endhighlight %}

Browse to [http://localhost:3000/](http://localhost:3000/), and you should see our beautiful home page!

{% include post_img_border.html path='sass_clojure_ring_apps/1-basic-home-page.png' caption='A very basic homepage' %}

## Set up SCSS

### Edit project.clj again

Now that we have our test project, it's time to get sassy with some CSS.  We don't want to be compiling SCSS files by hand each time we edit them.  Instead, we will be using the `lein-scss` plugin that we included in our `project.clj` file earlier.  Before we can use it, we need a bit more set up.

We need to tell `lein-scss` how we want our SCSS files to be compiled.  We do that by adding an `:scss` key with hash map of options to the end of the `defproject` macro in `project.clj`.

{% highlight clj %}
{% raw %}
  :scss {:builds
         {:development {:source-dir "resources/scss"
                        :dest-dir "resources/public/css"
                        :executable "sass"
                        :args ["--style" "expanded"]}
          :production {:source-dir "resources/scss"
                       :dest-dir "resources/public/css"
                       :executable "sass"
                       :args ["--style" "compressed"]}}}
{% endraw %}
{% endhighlight %}

In the options map, we specify the `:builds` key and then another map where we can specify multiple different builds.  This is nice when you want different options for development and production.  For example, we've specified the `expanded` style for development, but the `compressed` style for production.

There are a couple of other things to note here.  We use the `:source-dir` key to specify that we will store our SCSS files in `resources/scss`, and the `:dest-dir` key to specify that we want the compiled CSS files to live in `resources/public/css`.  Finally, we tell `lein-scss` to use the `sass` executable, and add some command line arguments to be passed in to the `sass` program.

*Remember how I said there were a lot of different options for Sass compilers?  Well the `:executable "sass"` option is for using `sass`.  Of course, if you're using `sassc` or `scss` instead, you can use `:executable "sassc"` or `:executable "scss"`, and it'll work just fine!*

### Make a main.scss file

Once that is set up, make a new file called `main.scss` in the `resources/scss` folder and add the following to it:

{% highlight scss %}
{% raw %}
$font-color: #E47320;
$font-family: Courier;

body {
  font-family: $font-family;
  color: $font-color;
}
{% endraw %}
{% endhighlight %}

### Compile SCSS to CSS

Those are some excellent styles, but if you reload the homepage now, you'll see that they aren't being applied.  This is because we haven't told `lein-scss` to actually compile `main.scss` to `main.css` yet.  Here is how to do that.

{% highlight bash %}
{% raw %}
$ lein scss :development once
[23:36:01] Running once
[23:36:02] ./sassy-clj/resources/scss/main.scss
       --> ./sassy-clj/resources/public/css/main.css
Elapsed time: 226.240492 msecs [Total time]
{% endraw %}
{% endhighlight %}

*Note that we typed `:development` and not `development`.  The latter will not work.*

If you reload the homepage again, you'll see our beautiful styles have been applied!

{% include post_img_border.html path='sass_clojure_ring_apps/2-styled-home-page.png' caption='A quite stylish homepage!' %}

### Setting up auto-compilation

Now you probably don't want to be manually running `lein scss` every time you edit your SCSS files.  To avoid this, `lein-scss` comes with an `auto` mode that watches your SCSS source directory for changes and automatically recompiles the CSS as necessary.  You can run it like this:

{% highlight bash %}
{% raw %}
$ lein scss :development auto
{% endraw %}
{% endhighlight %}

Finally, if you're ready for production, you can pass in `:production` instead of `:development` and you'll be good to go!

{% highlight bash %}
{% raw %}
$ lein scss :production once
{% endraw %}
{% endhighlight %}

And that's it!  Go forth and be sassy!
