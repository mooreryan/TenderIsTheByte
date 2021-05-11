---
layout: post
title: "Virome Bytes:  Prophages in Lactobacillus"
author: ryan
description: In today's edition of Virome Bytes, I talk about a couple of new papers looking at prophages in Lactobacillus!
categories: blog
image: "/assets/img/posts/lactobacillus_prophages/morphotype_3_phages_672_low.jpg"
twitter_share: https://ctt.ac/0bc4o
---

{:.gray}
*I wanted to start a little column where I would take some new(ish) virome papers that I like, and talk a bit about them.  Given that this is the first entry, I think it's kind of funny that these two papers are not actually virome papers!  While these studies aren't looking at viromes per se, I see them as sort of "virome-adjacent"--papers that I think will still be of interest to virome researchers who work mainly with environmental or microbiome data sets.*

Lysongenic phages are pretty interesting.  When they infect their host, rather than lysing the cell directly, they integrate into the genome.  While snug inside the host genome, the prophage isn't just along for the ride.  Sometimes they can be a burden to their host.  For example, there is a [metabolic consequence for maintaining the extra genetic content](https://doi.org/10.1016/0734-9750(95)00004-A) (sometimes [up to 16% of the total genome](https://dx.doi.org/10.1128%2FMMBR.67.2.238-276.2003)!!), and eventually prophages can activate and kill there host.  On the other hand, prophages can provide [superfection immuntiy](https://doi.org/10.1046/j.1365-2958.2002.02763.x) (protection against similar phages) to their host, and can confer [other advantages](https://doi.org/10.1007/s12275-014-4083-3), such as beneficial virulence factors.  However, there are only a couple of studies that look at the effect of prophages on host ecology actually conducted in the environment (see see [here](https://doi.org/10.1073/pnas.1206136109) and [here](https://doi.org/10.1371/journal.pgen.1005861) for two examples).

## Prophages in *Lactobacillus reuteri*

With this in mind, Jee-Hwan Oh and colleagues published a really cool study in [Applied and Environmental Microbiology](https://aem.asm.org) called [Prophages in Lactobacillus reuteri are associated with fitness trade-offs but can increase competitiveness in the gut ecosystem](https://doi.org/10.1128/AEM.01922-19) (Nov. 2019).  In the study, the authors looked at prophages present in a set of phylogenetically diverse *Lactobacillus reuteri* strains.  *L. reuteri* is a Gram-positive bacteria [found in the guts of vertebrates](https://doi.org/10.1093/femsre/fux030) and is often used as a [model for studying gut symbionts and their hosts](https://doi.org/10.1073/pnas.1000099107).  One strain, *L. reuteri* 6475 (a human isolate) has some nice genome editing tools [including a CRISPR-Cas genome editing system](https://doi.org/10.1093/nar/gku623) that make it a good strain to work with.  This particular strain also has [two biologically active prophage genomes that are induced in the GI tract](https://doi.org/10.1016/j.chom.2018.11.016).

{% include post_img_border.html path='lactobacillus_prophages/l_reuteri_phages.png' caption='Electron micrographs of two prophages from this study.' %}

The main points of this paper go something like this:

* They checked a bunch of *L. reuteri* genomes for prophages with [PHASTER](https://phaster.ca/) (a computational tool to predict prophage regions in sequenced genomes).
* They took a bunch of strains of *L. reuteri*, hit them with [Mitomycin C](https://dx.doi.org/10.3389%2Ffmicb.2017.01343), and then checked for prophage induction.
* They built and extensively tested a sweet experimental system around *L. reuteri* 6475, including a derivative lacking both phages, a sensitive host, single phage deletions, and more.
* They used this system to test some hypotheses about the affect of prophages on host fitness in the gut environment.

They used this system to show that prophages reduce the fitness of *L. reuteri* during gastrointestinal transit, and did some fancy work to show that phages were actually produced during GI transit in an [SOS](https://doi.org/10.1016/s0300-9084(85)80077-8)-dependent manner.  Even though the propahges were found to reduce host fitness in the GI tract, the authors found that biologically active prophages are widely distributed among phylogenetically diverse strains of of *L. reuteri*.

If you stop to think about it, that's a little weird.  Why are all these *L. reuteri* strains, across multiple lineages, maintaining prophages over evolutionary time scale if the prophages are reducing their host's fitness?  The authors figured that production of phages by lysogenized *L. reuteri* strains may provide a competitive advantage by killing competitor strains.  So they used their experimental system and did a whole bunch of competition experiments in germ-free mice.  To make a long story short, the *L. reuteri* strains with the prophages beat out the strains without them.  In other words, the prophage carrying *L. reuteri* strains had a competive advantage over the others *in vivo*, presumably because the non-lysogenized strains were susceptible to the phages produced by the lysogenized strains.

This competiteve advantage is one potential explanation for the widespread distribution of prophages found in gut bacteria.  While the lysogenized *L. reuteri* strains outcompeted non-lysogenized strains susceptible to their phages, there was still a decrease in overall fitness of the lysogenized strains.  Such fitness trade-offs (as mentioned in the title!) may help to explain the co-existance of closely related lysogenized and non-lysogenized strains.

Overall, a really cool study with some interesting results that I definitely encourge you to check out!

{:.gray}
*Citation:  Oh, J.-H. et al. Prophages in Lactobacillus reuteri are associated with fitness trade-offs but can increase competitiveness in the gut ecosystem. Appl. Environ. Microbiol. AEM.01922-19 (2019). [doi: 10.1128/AEM.01922-19](https://doi.org/10.1128/AEM.01922-19).*

## Prophages in *Lactobacillus brevis*

Another recent study also looked at prophages of Lactobacillus:  [Biodiversity and Classification of Phages Infecting Lactobacillus brevis](https://doi.org/10.3389/fmicb.2019.02396) (publishd by Feyereisen and collegues in [Fronteirs in Microbiology](https://www.frontiersin.org/journals/310), Oct. 2019).  Whereas the previous study focused on phages of *Lactobacillus reuteri*, this one focused on phages of *Lactobacillus brevis*.  *L. brevis* is particularly well known for its ability to [grow and survive in beer](https://doi.org/10.1002/j.2050-0416.2006.tb00247.x), leading to spoilage.

{% include post_img_border.html path='lactobacillus_prophages/beer.jpg' caption='No L. reuteri here!  (Image by Republica from Pixabay)' %}

As we know, prophage induction will generally cause cell lysis and death, so learning more about prophages in an important beer-spoilage bacteria should be beneficial to the beer making industry.  According to the authors, prophages could act as beneficial antimicrobial agents in beer fermentation, providing an alternative current practices.  Additionally, few *L. brevis* phages have been characterized, so a study looking at the biodiversity of these phages is a welcome addition to the literature.

In the this study, the authors collected nineteen publically available *L. brevis* genome sequences from which they identified putative prophage sequences using a combination of [PHASTER](https://phaster.ca/) and manual annotation.  Five of the strains were available for prophage induction trials.  They also looked at the relatedness of the *L. brevis* phages and proposed a classification scheme for them based on virion morphology and genome sequence data (cool!).

{% include post_img_border.html path='lactobacillus_prophages/morphotype_3_phages.jpeg' caption='Electron micrographs of morphotype 3 phages from this study.' %}

Prophages were common in the tested genomes, with one of the genomes having four predicted prophage regions!  Some of these were found to encode superinfection proteins, which could indicte that these prophages are boosting host fitness by confering resistance to infection by other phages.

The authors set up a classification scheme for *L. brevis* phages based on morphology, phylogeny, and genomic diversity, and provide a lot of interesting discussion about the defining charateristics of the five groups they defined.  Overall, the *L. brevis* phages showed substantial diversity.  Many of the identified prophages belong to the *Myoviridae*.  This is unusual for prophages of Lactic acid bacteria like *L. brevis*, as generally members of the *Siphoviridae* are more typically seen (see [Brüssow and Desiere, 2001](https://doi.org/10.1146/annurev.micro.55.1.283), [Casey et al., 2015](https://doi.org/10.1128/aem.03413-14), and [Kelleher et al., 2018](https://doi.org/10.1016/j.ijfoodmicro.2018.02.024)).

This is a nice paper to study if you are interested in looking at prophages in your favorite genomes or systems.  They provide a pretty good model to follow for identifying, characterizing, and classifying phages and prophages from particular organisms.  Check it out!

{:.gray}
*Citation:  Feyereisen, M. et al. Biodiversity and Classification of Phages Infecting Lactobacillus brevis. Frontiers in Microbiology 10, 2396 (2019). [doi: 10.3389/fmicb.2019.02396](https://doi.org/10.3389/fmicb.2019.02396).*
