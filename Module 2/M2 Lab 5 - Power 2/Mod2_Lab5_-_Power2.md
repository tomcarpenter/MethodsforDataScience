Module 2, Lab 5 - Power 2 - Sample Size Planning Illustration
=============================================================

In this lab, we will practice using the `pwr` package in R for
determining the minimum necessary sample size for a two-sample *t*-test
design. I illustrate a real-life iterative project planning sequence so
you can see how power planning can be integrated into a data-science
research project.

First, we load the `pwr` package:

    library(pwr)

You wish to compare groups at two different locations in your
organization to see if either group is more satisfied with their working
conditions. You will be comparing the groups with a *t*-test, and you
care deeply about estimating the effect, even if it is small. Data will
be challenging to get, however, as you will have to get managers to ask
employees to return surveys. You need to determine the minimum necessary
sample size to get adequate power.

Usually, we start with our dream scenario. Let's ask for 90% power to
detect a very small effect size (*d* = .10). What sample size would be
required?

    pwr.t.test(d=.10, power=.9)

    ## 
    ##      Two-sample t test power calculation 
    ## 
    ##               n = 2102.445
    ##               d = 0.1
    ##       sig.level = 0.05
    ##           power = 0.9
    ##     alternative = two.sided
    ## 
    ## NOTE: n is number in *each* group

We see here that we need 2103 people per group, or 4206 people in total.
Knowing the size of the organization, you know anything that size is
likely out of the question. You might be able to get away, you think
with collecting data from 500 participants without imposing too much on
team supervisors. So, you try again, this time with a more realistic 80%
power and a mid-range-small effect of *d* = .25.

    pwr.t.test(power=.80, d=.25)

    ## 
    ##      Two-sample t test power calculation 
    ## 
    ##               n = 252.1275
    ##               d = 0.25
    ##       sig.level = 0.05
    ##           power = 0.8
    ##     alternative = two.sided
    ## 
    ## NOTE: n is number in *each* group

By pure happenstance, you get 253 per group (always round up to ensure
sample size is adequate). You take this proposal to collect data from
500 employees to your supervisor; after some discussion, you are told
that they will try to push for a large sample (on the basis of your
request), but they've decided 400 is the maximum they are likely to be
able to collect.

Now, you change your strategy. 400 is close to 500, so it's likely to be
similar. You now leave out effect size and input *n* = 200 (since it's
per group) and a request for 80% power:

    pwr.t.test(n=200, power=.80)

    ## 
    ##      Two-sample t test power calculation 
    ## 
    ##               n = 200
    ##               d = 0.2808267
    ##       sig.level = 0.05
    ##           power = 0.8
    ##     alternative = two.sided
    ## 
    ## NOTE: n is number in *each* group

You will now have 80% power to detect effects as small as *d* = .28,
which is still a mid-range small effect.

Before you tell everyone that will still work with this suggested sample
size, you run a loop to estimate power at that sample size for various
effect sizes (e.g., power would suffer if *d* were lower, but would it
be *that* terrible if *d* were, say, .10?)

    #create empty container to store  power results
    power <- NULL

    #create sequence of d values to test, from .05 to .5 in increments of .05
    d.values <- seq(from=.05, to = .5, by=.05)

    # There are 10 d values, so we will need to run a loop 15 times
    # For each d value, run the power function and save the power resul
    for (i in 1:10){
      power[i] <-  pwr.t.test(200, d=d.values[i])$power
    }

    #bind results together into comuns to make a table
    cbind(d.values, power)

    ##       d.values      power
    ##  [1,]     0.05 0.07895539
    ##  [2,]     0.10 0.16948092
    ##  [3,]     0.15 0.32174708
    ##  [4,]     0.20 0.51408164
    ##  [5,]     0.25 0.70333330
    ##  [6,]     0.30 0.84914912
    ##  [7,]     0.35 0.93718673
    ##  [8,]     0.40 0.97884032
    ##  [9,]     0.45 0.99428309
    ## [10,]     0.50 0.99876886

Looking at this chart, we see that power really starts to drop off
around *d* = .20, hitting 51%. You discuss this with your team; they
conclude they are ok with a 50% chance of declaring "no difference" if
effect is *that* small. The study is run with 400 people and an
informative result is produced.

Epilogue
========

At the very end, you hear that the original proposal had been to collect
40 responses, 20 from each site. You smile to yourself, considering how
your power analysis likely saved the project. You run a power analysis
just to see how bad the situation would be:

    pwr.t.test(n=20, power=.80)

    ## 
    ##      Two-sample t test power calculation 
    ## 
    ##               n = 20
    ##               d = 0.9091587
    ##       sig.level = 0.05
    ##           power = 0.8
    ##     alternative = two.sided
    ## 
    ## NOTE: n is number in *each* group

You see here that the smallest effect size for which you would have good
power is well into the 'large' range. What if the effects were small?
What are the odds the study would even be able to pick them up? You
consider the scenario of *d* = .25:

    pwr.t.test(n=20, d=.25)

    ## 
    ##      Two-sample t test power calculation 
    ## 
    ##               n = 20
    ##               d = 0.25
    ##       sig.level = 0.05
    ##           power = 0.1203354
    ##     alternative = two.sided
    ## 
    ## NOTE: n is number in *each* group

This study would have 12% power. Yikes. It's a good thing you performed
a power analysis.
