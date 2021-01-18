---
layout: post
title: "divnet-rs: A Rust implementation for DivNet"
author: ryan
description: DivNet is an R package for estimating diversity when taxa occur in an ecological network.  Here I talk about why you may want to use DivNet, introduce my Rust implementation of the algorithm, and compare its performance to the original R package.
categories: blog
image: "/assets/img/posts/divnet_rs_intro/timing_425_350.png"
twitter_share: https://ctt.ac/r23i1
---

## Background

One reason for doing microbiome sequencing is to learn about the microbial diversity of the ecosystems of interest. Estimating the diveristy of microbial communities is hard. Essentially every step of a sample to sequence pipeline [introduces biases](https://doi.org/10.7554/eLife.46923) into your analyses, meaning the community composition you observe is likely quite different from the true community composition. Further, [microbiome datasets are compositional](https://doi.org/10.3389/fmicb.2017.02224), and must be treated with [statistical and computational methods](https://doi.org/10.1093/gigascience/giz107) designed to handle such data.

Most communities are incredibly complex so you're going to nearly always have issues with undersampling -- there are just too many microbes to sequence them all, so you have to work with samples. Even though you cannot practically observe all the taxa in your environment, you still need to estimate the diversity of that environment. So why don't we just "plug-in" our data into one of the common diversity indices borrowed from macroecology like Shannon or Simpson and be done with it? You will actually see this a lot in the literature: plugging in the observed relative abundances (sometimes after [rarefying](https://doi.org/10.1371/journal.pcbi.1003531) the data first) from our samples into standard "plug-in" diversity formulas.

There are a couple of problems with this. Undersampling is problematic because alpha diversity metrics are [heavily biased when there are unobserved taxa](https://doi.org/10.3389/fmicb.2019.02407). The random sampling variation combined with biases introduced in the sample-to-sequence pipeline mean your observed relative abundances probably don't faithfully represent the true community you want to study. Additionally, many commonly used methods for generating confidence intervals assume that taxa are independent (i.e., if taxa A is present in a community, it doesn't provide any information about whether taxa B is there too).

### What is DivNet?

So how are you supposed to measure diversity of microbial communities then? One method that is designed to address a lot of these problems is [DivNet](https://github.com/adw96/DivNet), an R package for estimating diversity when taxa in the community occur in an ecological network (i.e., a pattern of microbial co-occurence). DivNet leverages info from multiple samples and can estimate relative abundance of taxon in communities where it was unobserved. It also gives accurate estimates of variance in the measured diversity by taking into account sample metadata/covariates.

Probably the most interesting aspect of DivNet is that it allows you to account for ecological networks where taxa positively and negatively co-occur. DivNet estimates diversity using models from [compositional data analysis](https://en.wikipedia.org/wiki/Compositional_data) that can handle co-occurance networks. This is in contrast to most common diversity estimates that are based on the [multinomial model](https://en.wikipedia.org/wiki/Multinomial_distribution) that makes assumptions about sampling that prohibit ecological networks (i.e., situations in which taxa positively and negatively co-occur). (_Note: you may know the multinomial model from your stats courses in modeling the probability of counts for dice rolls or as generalization of the [binomial distribution](https://en.wikipedia.org/wiki/Binomial_distribution)._)

You can find a lot more information about DivNet, including algorithmic details, validation, comparison to other methods of estimating diversity, and some important details to keep in mind when using DivNet on your data in the [DivNet manuscript](https://doi.org/10.1093/biostatistics/kxaa015).

### Why make divnet-rs?

In the [getting started tutorial](https://github.com/adw96/DivNet/blob/31e04e29e4f3c02ea07c7f35873ee6743b79170a/vignettes/getting-started.Rmd), there is a section called "What does DivNet do that I can't do already?" (it is worth reading if you haven't!). So I thought it would be good to answer the question, "What does `divnet-rs` do that the R implementation of DivNet can't do aleady?" The answer is simple: `divnet-rs` gives you the ability to apply the DivNet algorithm to large datasets. For those without easy access to high performance computing facilities, you will be able to run DivNet on typically sized SSU rRNA microbiome datasets on your laptop. `divnet-rs` is both faster and much more memory efficent that the R implementation. Of course, bioinformatics software is all about tradeoffs and `divnet-rs` is no different. [Comapared to the R implementation](#differences-in-the-implementations), it's harder to install, you have to write some R code specifically to get data in and out of `divnet-rs`, and not all network and boostrapping options offered by the R implementation are available in the Rust implementation. That said, I think `divnet-rs` still fulfills a useful niche by allowing researchers to apply the DivNet algorithm to datasets that are currently too large for the R implementation to handle.

## Comparing runtime and memory usage

### Set up

While developing `divnet-rs`, I spent a good amount of time profiling and optimizing the code. Rather than talk about that, I wanted to get a high level overview of how the performance of the R and Rust implementation compared on a real dataset. The data I used was the [Lee dataset](https://doi.org/10.3389/fmicb.2015.01470) that is incuded with the DivNet R package. It has 1490 [amplicon sequence variants](https://doi.org/10.1038/ismej.2017.119) (ASVs), 16 samples, and associated taxonomy and sample info.

So what did I do? First, I took the Lee data and sorted the ASV table in decreasing abundance order. Then I created new datasets from the top 10, 20, 40, 80, 160, 320, 640, and 1280 ASVs. In addition to the full 16 sample datasets, I also created test datasets with only eight samples by randomly picking samples from the ASV table, remiving any ASVs that had zero count in the remaining samples, and then took the top 10, 20, ..., 1280 ASVs just like for the 16 sample datasets. I ran everything with the default algorithm tuning in DivNet (6 expectation maximization (EM) iterations (3 burn), 500 Monte-Carlo (MC) iterations (250 burn)) and 2 replicates. I would probably use the "careful" setting (10 EM iterations and 1000 MC iterations) as well as running more replicates if I was actually analyzing data, but this was good enough for this little profiling experiment.

This isn't the most scientific profiling job ever, but it should give you a sense of how the runtime and memory scales with the number of taxa and samples for both the R and Rust versions of DivNet. For the timing, I ran each dataset three times, and I used the `time` function to get the elapsed time and the max memory used for each run. Since loading all the R dependencies takes a large proportion of the total runtime in the smaller DivNet-R runs, I got the elapsed time of just the `divnet` function using the [tictoc](https://cran.r-project.org/web/packages/tictoc/index.html) R package. I still used `time` to get the max memory for these runs though.

One other thing to mention, I ran all of these on a compute cluster. I didn't think about it until after I had already run everthing, but I compiled both `divnet-rs` and `OpenBLAS` on a different node than the one that I used to actually run the tests. The compute cluster that I used has a bunch of different types of nodes, so the compiled output of both may not be ideal for the node I actually ran the timings on (e.g., different [SIMD instructions](https://en.wikipedia.org/wiki/SIMD), different CPU architectures, etc.). While the timing experiments were running, there were other jobs on the same node running at the same time, so that is another thing that may have influenced the results.

For the R tests, I used R v3.6.2 linked against [OpenBLAS](https://www.openblas.net/) v0.3.7 and DivNet v0.3.6. I set DivNet to use only 1 core (`ncores = 1`) because in all my tests (and on multiple different machines), DivNet is actually slower when using more than one core. For `divnet-rs` I used v0.1.1 linked against OpenBLAS v0.3.13. I also forced OpenBLAS to use only 1 core (`OPENBLAS_NUM_THREADS=1`) as that is how the R was using OpenBLAS. (_As an aside, if you don't have [R linking against an optimized BLAS implementation](https://csantill.github.io/RPerformanceWBLAS/), you should. It will give you a big perfomance increase._)

Just keep all this stuff in mind while taking a look at these results.

### Results

Here are the runtime and memory profiling results:

{% include post_img_border.html path='divnet_rs_intro/timing_425_350.svg' caption="DivNet timing and memory requirements" %}

Let's break down a couple of things. The Rust version is faster and more memory efficient, but that's not surprising -- a Rust program should be faster than an R program, and I spent a good amount of time profiling and optimizing the code. In this test, the Rust version is about 20 times faster than the R version.

The other interesting thing to measure is max memory usage. For the largest dataset that I tested (16 samples, 1280 taxa), the Rust version used ~300 MB of RAM as compared to the ~6000 MB used by the R version. When implementing DivNet in Rust, I spent a good amount of time and effort optimizing the run time, and much less worrying about the memory, so it was nice to see it being relatively frugal with the memory.

As you might expect, the 16 sample datasets took longer and used more memory than the 8 sample datasets, but not twice as much time and memory. There was a weird thing thing in the 1280 taxa test set in the Rust implementation. The 8 sample set actually took a bit more time (but still used less memory) than the 16 sample set. I thought this was strange so I actually ran the 16x1280 and 8x1280 datasets many more times to see if there was some weird random variation in the timings, or if I made some mistake in the testing and mislabeled the datasets or something, but each run gave me relatively the same result as you see here. I'm not honestly sure why this is, but like I mention above, these benchmarks aren't prefect and could be improved.

## Differences in the implementations

Before wrapping up, I want to take a little time to highlight some of the more important differences in the R and Rust implementations of DivNet.

### Estimating the network

While the original DivNet R code has multiple options for the `network` parameter, the only network option in `divnet-rs` is "diagonal". To explain why this is, here is an excerpt from a [GitHub issue](https://github.com/adw96/DivNet/issues/32) where [Amy Willis is talking](https://github.com/adw96/DivNet/issues/32#issuecomment-521727997) about using DivNet on large datasets:

> I would recommend network="diagonal" for a dataset of this size. This means you're allowing overdispersion (compared to a plugin aka multinomial model) but not a network structure. This isn't just about computational expense -- it's about the reliability of the network estimates. Essentially estimating network structure on 20k variables (taxa) with 50 samples with any kind of reliability is going to be very challenging, and I don't think that it's worth doing here. In our simulations we basically found that overdispersion contributes the bulk of the variance to diversity estimation (i.e. overdispersion is more important than network structure), so I don't think you are going to lose too much anyway.

Another benefit of the diagonal network is that it is fast: it's a simple, vectorizable mathematical operation, as compared to the [default method](https://github.com/adw96/DivNet/blob/31e04e29e4f3c02ea07c7f35873ee6743b79170a/R/MCmat.R#L83), which will need to do either a Cholesky decomposition or a generalized matrix inversion, or to the ["stars"](https://github.com/adw96/DivNet/blob/31e04e29e4f3c02ea07c7f35873ee6743b79170a/R/MCmat.R#L114) method, which does a whole lot more operations.

`divnet-rs` isn't a replacement for DivNet. It's focus is on allowing the core algorithm to be applied to datasets that are too large for the R implementation to handle, and so, only the diagonal network is available in `divnet-rs`. If your data is small enough that the R implentation can handle it, then I recommend using the original!

### Bootstrapping

Another difference from the original is that only the parametric bootstrap is available -- you can't do the nonparametric bootstrap. The parametric bootstrap is the default in the R implementation, and, if you check out the [DivNet manuscript](https://doi.org/10.1093/biostatistics/kxaa015), you'll see that the parametric and nonparametric bootstraps perform similarly.

### Setting the random seed

`divnet-rs` currently does not allow you to set the seed for the random number generator, which will have an impact on reproducibility across runs. While the DivNet R implementation does allow you to set the random seed prior to the run (for example, just use `set.seed(5623472)` before running the `divnet` function), there is a [caveat about setting the random seed when running DivNet on multiple cores](https://github.com/adw96/DivNet/blob/31e04e29e4f3c02ea07c7f35873ee6743b79170a/vignettes/getting-started.Rmd#L64) that you should be aware of. In practice, if you are getting more variability across runs than desired, you can up the EM iterations, the MC iterations, and the replicates, and it [should take care of things](https://github.com/adw96/DivNet/blob/31e04e29e4f3c02ea07c7f35873ee6743b79170a/vignettes/getting-started.Rmd#L64).

## Wrap-up

In this post, I introduced `divnet-rs`, a Rust implementation of the [DivNet R package](https://github.com/adw96/DivNet). It is both faster and more memory efficent than the original, allowing you to run much larger data sets even on your laptop, but it has fewer features and isn't as straightforward to use. Like any bioinformatics software, there are always tradeoffs, so I encourage you to pick the right tool for the right job: if you have small enough datasets, stick with the R implementation, but if R keeps crashing on you or DivNet is just too slow for whatever reason, give [divnet-rs](https://github.com/mooreryan/divnet-rs) try.
