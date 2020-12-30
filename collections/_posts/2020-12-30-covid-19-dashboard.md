---
layout: post
title: "A simple dashboard for COVID-19 case counts"
author: ryan
description: Introducing the COVID-19 dashboard I made to compare case counts across U.S. counties.
categories: blog
image: "/assets/img/posts/covid_19_dashboard/delaware_covid_chart.png"
twitter_share: https://ctt.ac/aKnUA
---

I made a simple [COVID-19 dashboard](https://www.tenderisthebyte.com/apps/covid19dashboard) that lets you compare the confirmed case counts for multiple counties as well as viewing the raw counts and the counts per 100,000 people.  It plots the case counts over time for as many counties as you want to compare and lets you download the resulting chart.  Here is an example for Delaware's three counties:

{% include post_img_border.html path='covid_19_dashboard/delaware_covid_chart.svg' caption="Confirmed COVID-19 Cases for Delaware Counties" %}

Being a Delaware resident, I like to pretend everyone already knows everything about Delaware, but *just in case* you don't, here you go:  New Castle county is in the north and has Wilmington (our largest city) and Newark, home of the Univesity of Delaware.  Kent county is in the middle and has Dover (the state capitol), and Sussex county is in the south with Lewes and all the beaches.  It's interesting to see the differences between New Castle and Kent counties, which look pretty similar to one another, and Sussex county.  At some point, I would like to overlay some demographic or socio-economic data on this to look for any trends, but that's for a different day.


## The data

The COVID-19 case data is from the [Center for Systems Science and Engineering (CSSE) at Johns Hopkins University](https://github.com/CSSEGISandData/COVID-19).  Their data is aggregated from a ton of different sources and I encourage you to check out [their GitHub page](https://github.com/CSSEGISandData/COVID-19) for more information about the data.  If you're interested, they have [an article](https://doi.org/10.1016/S1473-3099(20)30120-1) in the Lancet talking about the data and [their dashboard](https://www.arcgis.com/apps/opsdashboard/index.html#/bda7594740fd40299423467b48e9ecf6).  Of course, their dashboard has a lot more bells and whistles than mine!

For the county level population info, I used data from the [Atlas of Rural and Small-Town America](https://www.ers.usda.gov/data-products/atlas-of-rural-and-small-town-america/) from the [USDA Economic Research Service](https://www.ers.usda.gov/).  It is a really cool and in-depth county level dataset.  In addition to the population data, you can find info about jobs, income, veterans and more.  They also have a [nice interactive map](https://www.ers.usda.gov/data-products/atlas-of-rural-and-small-town-america/go-to-the-atlas/) to view everything county-by-county.  If you want to download and remix the data yourself, it is all available in CSV and Excel format [on their site](https://www.ers.usda.gov/data-products/atlas-of-rural-and-small-town-america/download-the-data/).

One thing to note is that the county level population data is mostly from 2019 estimates.  So, while weighting the case counts by the population data gives a nice way to compare COVID-19 cases across counties, just keep in mind that the population estimates are from last year.

## The code

If you're interested in the source code for the dashboard, you can find it on my [GitHub page](https://github.com/mooreryan/Covid19Dashboard).  

It is an [Elm app](https://elm-lang.org/).  I haven't used Elm much before this project, but it was very easy to get started with.  The [documentaion](https://guide.elm-lang.org/) was awesome and the [Elm Slack channel](https://elmlang.herokuapp.com/) is full of helpful people.  I think having some experience in [Rust](https://www.rust-lang.org/) and [Clojure](https://clojure.org/) helped me feel right at home using Elm.  Elm seems a bit like a gateway to [PureScript](https://github.com/alpacaaa/elm-to-purescript-cheatsheet) or [Haskell](https://www.reddit.com/r/haskell/comments/6wbzer/elm_as_a_gateway_to_learn_haskell/), so I'm thinking of checking those out as well.

The charts are made with [Vega-Lite](https://vega.github.io/vega-lite/), a nice tool for data visualization based on [Vega](https://vega.github.io/vega/) and the [Grammar of Graphics](https://www.cs.uic.edu/~wilkinson/TheGrammarOfGraphics/GOG.html).  It's [declarative](https://en.wikipedia.org/wiki/Declarative_programming), in that you write [JSON](https://www.json.org/json-en.html) specifications and Vega-Lite compiles the spec to Vega and Vega's runtime hadles rendering the chart.  To generate the Vega-Lite specs, I used [this Elm package](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite) in conjunction with Elm [ports](https://guide.elm-lang.org/interop/ports.html).