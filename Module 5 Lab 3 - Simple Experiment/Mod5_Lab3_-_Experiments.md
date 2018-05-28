Module 5, Lab 3: Between Subjects Experiments
=============================================

In this lab, we will explore the between-subjects experiment (sometimes
referred to as an "A/B" test). I apply concepts from across the entire
course, including review of statistical modeling, power analysis, effect
size, measurement, and so on ... so that you can see the entire process
at once.

In this example, a design firm has produced several different logos for
a company as part of a rebranding process, and the company executives
have selected three that they like. These were then tested on a sample
of 100 customers. Each participant completed a survey about a randomly
selected logo. You are the data analyst and were called upon to make
sense of the data.

As part of the survey, participants rated their impression of the logo
on several dimensions: "friendly", "inviting", "interesting",
"positive", "pleasant" on a 1-10 scale (1 = *does not describe logo*; 10
= *describes logo perfectly*). As these are all positive adjectives and
you suspect people will not be very discerning in their answers, you
wish to combine them into a single "positive sentiment" scale. You then
wish to compare the logos on this sentiment scale.

We will use the `ggplot2` package for data visualization, the `effsize`
package for effect sizes,the `pwr` package for power analysis, and the
`psych` package for reliability analysis.

    #### LOAD PACKAGES ####
    library(ggplot2)
    library(pwr)
    library(psych)

    ## 
    ## Attaching package: 'psych'

    ## The following objects are masked from 'package:ggplot2':
    ## 
    ##     %+%, alpha

    library(effsize)

    ## 
    ## Attaching package: 'effsize'

    ## The following object is masked from 'package:psych':
    ## 
    ##     cohen.d

Load the Data
=============

We first load the data and inspect it.

    #### LOAD DATA ####
    dat <- read.csv("logos.csv", header=T)

We quickly glance at the names of the variables:

    names(dat)

    ## [1] "id"          "friendly"    "inviting"    "interesting" "positive"   
    ## [6] "pleasant"    "sex"         "logo"

1.  We have an ID variable representing the individual participants
2.  We have variables representing the different adjectives on which
    people rated the logo
3.  We have participant sex
4.  We have the logo that was rated

It is clear this is not the whole dataset, as there would likely have
been more variables.

Let's briefly check the structure of the dataset:

    str(dat)

    ## 'data.frame':    100 obs. of  8 variables:
    ##  $ id         : int  3 98 73 89 13 19 42 21 86 38 ...
    ##  $ friendly   : int  9 7 5 4 8 8 9 9 5 10 ...
    ##  $ inviting   : int  8 6 3 5 7 7 8 9 5 9 ...
    ##  $ interesting: int  9 8 6 2 9 7 10 9 4 10 ...
    ##  $ positive   : int  10 7 6 3 8 9 8 9 6 10 ...
    ##  $ pleasant   : int  9 7 6 4 6 7 8 9 5 10 ...
    ##  $ sex        : Factor w/ 2 levels "f","m": 1 1 2 1 1 2 1 1 1 1 ...
    ##  $ logo       : Factor w/ 3 levels "Logo A","Logo B",..: 1 3 3 3 1 1 2 1 3 2 ...

Happily for us, we have the adjectives all scored as numeric variables
and the `logo` variable scored as a factor. We will need to do little
(or nothing) to wrangle the data.

We do a final quick inspection on the variables in the dataset.

    summary(dat)

    ##        id            friendly        inviting      interesting   
    ##  Min.   :  1.00   Min.   : 4.00   Min.   : 3.00   Min.   : 2.00  
    ##  1st Qu.: 25.75   1st Qu.: 7.00   1st Qu.: 7.00   1st Qu.: 7.00  
    ##  Median : 50.50   Median : 8.00   Median : 8.00   Median : 8.00  
    ##  Mean   : 50.50   Mean   : 8.05   Mean   : 7.99   Mean   : 7.93  
    ##  3rd Qu.: 75.25   3rd Qu.: 9.00   3rd Qu.: 9.00   3rd Qu.:10.00  
    ##  Max.   :100.00   Max.   :10.00   Max.   :10.00   Max.   :10.00  
    ##     positive     pleasant     sex        logo   
    ##  Min.   : 3   Min.   : 3.00   f:67   Logo A:33  
    ##  1st Qu.: 7   1st Qu.: 7.00   m:33   Logo B:32  
    ##  Median : 8   Median : 8.00          Logo C:34  
    ##  Mean   : 8   Mean   : 7.89          NA's  : 1  
    ##  3rd Qu.:10   3rd Qu.: 9.25                     
    ##  Max.   :10   Max.   :10.00

We see here that the ranges on the adjective variables are all in range
(1-10), which we should expect (but should still check). The `logo` was
randomly assigned to participants, so this is approximately evenly
distributed. Annoyingly, we have a missing value on the `logo` variable.
Of note,`sex` weights toward females, which we assume is representative
of the data source. Hopefully this is representative of likely customers
(if not, this would be a key limitation of this study).

Score the Scale
===============

We had the goal of creating a "positive sentiment" variable for these
data. We can quickly eyeball a correlation matrix to ensure these
measures correlate highly with each other. The adjectives represent
columns 2-6 of `dat`, so the correlations among the adjectives can be
shown with `cor(dat[,2:6])`. Remember that the brackets after `dat`
contain a comma: the blank *before* the comma means "all rows," the
"2:6" after the comma means "columns 2-6". Note that I wrap this within
`round()` to enhance readability, rounding to two decimals:

    round(
      cor(dat[,2:6])
    ,2)

    ##             friendly inviting interesting positive pleasant
    ## friendly        1.00     0.86        0.87     0.82     0.85
    ## inviting        0.86     1.00        0.66     0.70     0.76
    ## interesting     0.87     0.66        1.00     0.77     0.73
    ## positive        0.82     0.70        0.77     1.00     0.69
    ## pleasant        0.85     0.76        0.73     0.69     1.00

We see these ratings are all moderately-strongly correlated. Later in
your training, you might learn to run a principal components analysis or
a factor analysis to ensure they appear to represent "one underlying
factor" (i.e., sentiment). For now, we can be satisfied with showing
acceptable correlations among the ratings and by showing acceptable
scale reliability.

Next, we proceed to score it as a sale. We use `rowMeans()` to get the
average across the five adjectives. Note that `rowMeans()` only accepts
a data frame of variables to score, so we must create a data frame. We
could do this by manually creating one with `data.frame()` or by using
`subset()` and selecting the variable names we want. Either works. I
illustrate both ways. In either case, we save the result to `dat` as the
new variable, `dat$sentiment`.

    #data.frame method
    dat$sentiment <- rowMeans(
      data.frame(dat$friendly, dat$inviting, dat$interesting, dat$positive, dat$pleasant)
    )

    #subset method
    dat$sentiment <- rowMeans(
      subset(dat, select=c("friendly", "inviting", "interesting", "positive", "pleasant"))
    )

Both of these do the same thing (the `subset()` command returns a data
frame equivalent to the one in the first example).

Let's check the measure.

    mean(dat$sentiment)

    ## [1] 7.972

    sd(dat$sentiment)

    ## [1] 1.54345

    ggplot(data=dat, aes(x=sentiment))+
      geom_histogram(color="black", fill="light blue", bins=15)+
      theme_light()+
      scale_x_continuous(name="Sentiment")+
      ggtitle("Histogram of Sentiment")

![](Mod5_Lab3_-_Experiments_files/figure-markdown_strict/unnamed-chunk-8-1.png)

We see that scores tend to weight toward the positive end of the scale.

Is this a reliable and valid measure of consumer sentiment? Before we
make business decisions with it, we should assess that.

We first assess reliability by computing Cronbach's alpha using
`alpha()` from the `psych` package. This is done using the same data
frame as we used for `rowMeans()` above. Because several packages have
an `alpha()` function, we will specify that we want the one from the
`psych` package with `psych::alpha()`. The `alpha` command returns a
long list of things, but we are only interested in
`psych::alpha()$total`:

    psych::alpha(
      data.frame(dat$friendly, dat$inviting, dat$interesting, dat$positive, dat$pleasant)
    )$total

    ##  raw_alpha std.alpha   G6(smc) average_r      S/N         ase  mean
    ##  0.9413098 0.9437555 0.9459402 0.7704266 16.77953 0.009503987 7.972
    ##       sd
    ##  1.54345

The raw alpha here is .94, meaning that the scale is an estimated 94%
reliable (i.e., 94% of variance is "true" variance in sentiment; 6% of
the variance is random junk). This is great.

The next question is tougher. Are we **validly measuring sentiment and
not something else**, such as a tendency to respond positively on
surveys? In this case, we don't know. The adjectives seem representative
of the positive sentiment we would want to know about (i.e., content
validity) and appear good on their face (i.e., face validity). However,
we do not have evidence that this sent of adjectives teaks with other
indicators of sentiment in this study. Thus, we are accepting on faith
that these are a valid measure of sentiment. This is important to keep
in mind when interpreting results. It would be ideal if we had a set of
"tried and true" adjectives from prior studies that we could use that we
*know* correlate in ways that show their validity. For now, we just have
to trust the measure based on our impression.

Vizualize the Data + Run Descriptives
=====================================

The next step is to visualize the relationships in the data we want to
test. We also should examine the descriptive statistics to test our
question. In this case, we have randomly assigned participants to logos,
so everything about the participants (age, sex, consumption history,
etc.) will also be randomly distributed across the three logos. The
*only* systematic differences across the three groups was the logo.
Thus, any systematic differences in perceived friendliness can be
attributed to the logo. This is the beauty of an experimental design. We
can easily see the effect of the logo by comparing the groups.

In the between-subjects design, we compare means across groups. Thus, we
can visualize our data by looking at the distribution of scores across
groups. Using `ggplot2` we can use a boxplot.

    ggplot(data=dat, aes(x=logo, y=sentiment, fill=logo))+
      geom_boxplot(alpha=.20, color="black")+
      geom_jitter(alpha=.5, color="black", fill="grey90", width=.20)+
      theme_light()+
      scale_y_continuous(name="Sentiment")+
      scale_x_discrete(name="Logo")+
      scale_fill_discrete(name="Logo")

![](Mod5_Lab3_-_Experiments_files/figure-markdown_strict/unnamed-chunk-10-1.png)
Annoyingly, we see that we forgot to remove the case with the missing
value on the logo, so `ggplot2` created a separate column for it. We can
easily fix this by telling ggplot to *include only rows* of `dat` that
are "not missing on the logo variable" (i.e., `!is.na(dat$logo)`) as
follows: `dat[!is.na(dat$logo), ]`:

    ggplot(data=dat[!is.na(dat$logo), ], aes(x=logo, y=sentiment, fill=logo))+
      geom_boxplot(alpha=.20, color="black")+
      geom_jitter(alpha=.5, color="black", fill="grey90", width=.20)+
      theme_light()+
      scale_y_continuous(name="Sentiment")+
      scale_x_discrete(name="Logo")+
      scale_fill_discrete(name="Logo")

![](Mod5_Lab3_-_Experiments_files/figure-markdown_strict/unnamed-chunk-11-1.png)
We can see in our sample that Logo A and B appear to result in the
highest positive sentiment, followed by Logo C. It's not clear whether
Logo A or B is better.

We can quickly request more detailed statistics using `tapply()`. Recall
for `tapply()` accepts four arguments: the first is the variable to
analyze, the second is a factor (or list of factors) across which we
want to run the analysis, the third is the function we want to use in
the analysis, and the rest are arguments to pass along to our function.
In this case, we want means of `sentiment` across levels of `logo`, with
missing values ignored:

    # Means
    tapply(dat$friendly, dat$logo, mean, na.rm=T)

    ##   Logo A   Logo B   Logo C 
    ## 8.696970 8.562500 6.970588

    # SDs
    tapply(dat$friendly, dat$logo, sd, na.rm=T)

    ##   Logo A   Logo B   Logo C 
    ## 1.158794 1.522678 1.242805

We could also make a table and wrap each `tapply()` call in a `round()`
command to make a pleasant table of results:

    #Make Mean and SD Table
    tab.1 <- rbind(
      round(tapply(dat$sentiment, dat$logo, mean, na.rm=T), 2),
      round(tapply(dat$sentiment, dat$logo, sd, na.rm=T), 2)
    )

    tab.1 <- t(tab.1)
    colnames(tab.1) <- c("M", "SD")
    tab.1

    ##           M   SD
    ## Logo A 8.58 1.27
    ## Logo B 8.44 1.51
    ## Logo C 7.00 1.33

Logo A and B look close together and ahead of Logo C. Of course, that is
just in our sample, so it is unclear whether we can infer there are real
differences in the population. For that, we need our inferential tests.

Inferential Test for Two Groups
===============================

In some experiments, you will have two groups to compare. For example,
in the Module 3 lab on causal claims, we explored such an experiment. In
others, you will have three or more groups. Although our overarching
example is three groups, let's review this and pretend for a while that
you were *only* comparing logos A and B to illustrate the two-group
example.

Whenever you compare (means of) two groups, you conduct an independent
samples *t*-test. You can also use regression (run a regression
predicting your outcome from your two-group predictor), but the *t*-test
is commonly done and is illustrated here.

Recall that Logo A did better *in our sample*, but is this a *real*
difference in the population or just an artifact of random chance (due
to reliance on random assignment to groups)?

Recall that the null hypothesis always says that the **effect is absent
in the population** and that the sample result is an artifact of random
chance. In symbols, this means that the difference between the group
averages is exactly zero in the population.

*H*<sub>0</sub> : *μ*<sub>*A*</sub> − *μ*<sub>*B*</sub> = 0
 Remember that *μ* refers to the population average, so this is saying
that the population difference is exactly zero. Any difference observed
in our sample is therefore due to random chance.

We run our *t*-test to consider this possibility.

Recall that a *t*-test compares the size of the *observed difference*
($\\bar{x}\_{1}-\\bar{x}\_{2}$) against the value in the null hypothesis
(zero), divided by what is typically expected by chance:

$$t=\\frac{result - null }{chance}$$
 The top of the faction is key here. The more the data "disagree" with
the null, the larger the *t*-value and hence more evidence for a logo
effect. However, we also know some differences occur due to random
chance, so we divide by a measure of that that to take it into account.
When all is said and done, therefore, a large *t*-value tells you that
the effect is considerably larger than expected by chance. That would be
evidence for a real logo effect.

How can we run our test? The default in R is to run the "Welch" version
of the test. This version of the test does *not* make any assumptions
about the variances of the two groups.

$$t'=\\frac{result - null }{chance}=\\frac{(\\bar{x}\_{1}-\\bar{x}\_{2}) - 0 }{\\sqrt{\\frac{\\hat{\\sigma}\_1^2}{n\_{1}}+\\frac{\\hat{\\sigma}\_2^2}{n\_{2}}}}$$

There's no need to worry about the equation; it just implements what we
said above. Notice that the bottom includes the sample variances of the
two groups ($\\hat{\\sigma}\_1^2$ and $\\hat{\\sigma}\_2^2$ .... i.e.,
standard deviations squared). These are kept separate in the Welch
*t*-test, meaning that it's OK if they are different from each other.

How is this done via R?

First, we need a data frame that restricts `logo` to the two groups of
interest. In this case, I make `dat2` that contains only rows for which
`dat$logos` is Logo A or (symbol for "or" is `|`) Logo B (for the
purposes of example):

    dat2 <- dat[(dat$logo == "Logo A" | dat$logo=="Logo B"), ]

The R code for the *t* test looks similar to the code for a regression.
Either of the following would work:

    t.test(dat2$sentiment ~ dat2$logo)

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  dat2$sentiment by dat2$logo
    ## t = 0.3809, df = 60.547, p-value = 0.7046
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -0.5611026  0.8251178
    ## sample estimates:
    ## mean in group Logo A mean in group Logo B 
    ##             8.575758             8.443750

    t.test(sentiment ~ logo, data=dat2)

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  sentiment by logo
    ## t = 0.3809, df = 60.547, p-value = 0.7046
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -0.5611026  0.8251178
    ## sample estimates:
    ## mean in group Logo A mean in group Logo B 
    ##             8.575758             8.443750

I didn't show you this one yet, but a third option is to ditch the
`sentiment ~ logo` notation and simply input two vectors of sentiments
representing the two groups:

    t.test(dat2$sentiment[dat2$logo=="Logo A"], dat2$sentiment[dat2$logo=="Logo B"])

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  dat2$sentiment[dat2$logo == "Logo A"] and dat2$sentiment[dat2$logo == "Logo B"]
    ## t = 0.3809, df = 60.547, p-value = 0.7046
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -0.5611026  0.8251178
    ## sample estimates:
    ## mean of x mean of y 
    ##  8.575758  8.443750

In all three cases, you get the same result.

The *t*-value of 0.38 tells us that the difference is 0.38 times the
size of what one would typically expect by chance. This gives us our
*p*-value. We see here that the *p*-value is .70 so by definition, we
could get a result this big in our sample 70% of the time when the null
hypothesis that there is *NO* difference is true. This is not
statistically significant (*p* must be below .05 or 5%). We must accept
the null hypothesis that this could plausibly be due to random chance. I
don't have enough evidence to declare a real difference between Logo A
and Logo B.

#### Another Option: Student's *t*

Note that there is another version of the *t*-test, Student's *t*, which
is the same but it uses a form of averaging of the two sample variances:

$$t=\\frac{result - null }{chance}=\\frac{(\\bar{x}\_{1}-\\bar{x}\_{2}) - 0 }{\\sqrt{\\hat{\\sigma}\_p^2 (\\frac{1}{n\_{1}}+\\frac{1}{n\_{2}}))}}$$
 In this case, the $\\hat{\\sigma}\_p^2$ is a weighted average of the
two sample variances. If you cared for the equation for that (which you
probably don't), it looks like this:

$$\\hat{\\sigma}\_p^2 = \\frac{\\left ( n\_1 - 1 \\right )\\hat{\\sigma}\_1^2+\\left ( n\_2-1 \\right )\\hat{\\sigma}\_2^2}{\\left ( n\_1 - 1 \\right )+\\left ( n\_2-1 \\right )}$$
 Essentially, this is a sophisticated form of averaging
$\\hat{\\sigma}\_1^2$ and $\\hat{\\sigma}\_2^2$ so that the one from the
larger group has more influence over the final answer.

Long story short: this may not be the best test, as it *requires* the
variances (*σ*<sub>1</sub><sup>2</sup> and *σ*<sub>2</sub><sup>2</sup> )
of the two groups to be equal at the population level. Since we can
never know anything about the population, we can never know if this
assumption is met. We can test for it, but that has its own
complications. Further, recent advice (Delacre, Lakens, & Leys, 2017)
indicates that the Welch's *t* (done above) is better under real-world
conditions. So, we will skip this version. If you wanted to run it, you
would simply add the equal-variance assumptions:

    t.test(dat2$sentiment ~ dat2$logo, var.equal=TRUE)

    ## 
    ##  Two Sample t-test
    ## 
    ## data:  dat2$sentiment by dat2$logo
    ## t = 0.38192, df = 63, p-value = 0.7038
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -0.5587100  0.8227252
    ## sample estimates:
    ## mean in group Logo A mean in group Logo B 
    ##             8.575758             8.443750

The only other thing to know about the difference between the two tests
is that Welch's *t*-test is often slightly less powerful (we act as
though it were based on a smaller sample than it actually was). The
rationale for that goes beyond the scope of this tutorial, but
essentially we are adding some uncertainty into our result because we
are using two sample-based numbers ($\\hat{\\sigma}\_1^2$ and
$\\hat{\\sigma}\_2^2$) in the bottom of our equation, both of which are
prone to error. It's worth it, however, as you don't make as many
assumptions about the data. Still, in cases of very small samples (or
very uneven sized groups), Student's *t* test is an acceptable backup.

#### Conclusions

We need to tell our team that Logo A and Logo B are statistically tied:
we have no evidence for any real difference between them at the
population level. But what does that mean?

Before we move on: remember that **any** time a result is not
significant, it means that either

1.  The null hypothesis is true (i.e., the two groups are **exactly**
    equal in the population)
2.  The effect is smaller than we have power to detect

In fact, given our sample size, it's possible that there may still be an
logo effect...and we simply missed it. Given our sample size, what can
we actually *say* about the difference? This is given for us in the
*t*-test output with a 95% Confidence Interval.

Notice in the middle of the *t*-test output that a 95% CI is given to
you: \[-0.55 to 0.82\]. This means that we are 95% confident that the
true difference between the groups is somewhere between a half a point
*lower* for Logo A and nearly a full point *higher* for Logo A. Thus, it
is obvious this is non-significant (since a difference of *zero* is
right in the middle of that range of plausible values).

It is up to our subject matter expertise to interpret this. On a 1-10
satisfaction scale, we are confident that the they are within about .50
to .80 points of each other. That's not a very big difference in my
subjective opinion. Thus, we can reasonably state that there is not a
large difference between our groups, but it is possible there is still a
small difference (specifically: between a half a point *lower* for Logo
A and nearly a full point *higher* for Logo A).

What if we wanted a more precise answer with a tighter range? Well, we
would need a bigger sample size. A bigger sample has less error and
therefore a more precise CI. If our higher ups wanted a more precise
answer, we would need to re-run the survey with a larger sample.

#### If Units Are Meaningless

In this case, a 1-10 satisfaction scale has somewhat meaningful units.
In some cases, that is not the case. In such cases, you can use the
Cohen's *d* effect size statistic.

Recall from an earlier module that this quantifies the size of the
difference using a standard unit. Technically, that unit is standard
deviations:
$d = \\frac{difference}{SD} = \\frac{\\bar{x}\_{1}-\\bar{x}\_{2}}{\\sigma}$.
If you don't care about formulas, you simply need to know that it
rephrases the size of the difference using a common unit with a known
scale to it:

    data.frame(d=c("0 - 0.2",
                   "0.2 - 0.5",
                   "0.5 - 0.8",
                   "0.80 + "),
               meaning=c("Negligible", "Small", "Medium", "Large"))

    ##           d    meaning
    ## 1   0 - 0.2 Negligible
    ## 2 0.2 - 0.5      Small
    ## 3 0.5 - 0.8     Medium
    ## 4   0.80 +       Large

To test this in R, we use the `effsize` package, which contains a
`cohen.d()` function. Because both the `effsize` and `psych` packages
contain functions with that name, we specify here that it comes from the
`effsize` package:

    effsize::cohen.d(dat2$sentiment ~ dat2$logo)

    ## Warning in cohen.d.default(d, f, ...): Factor with multiple levles, using
    ## only the two actually present in data

    ## 
    ## Cohen's d
    ## 
    ## d estimate: 0.09475291 (negligible)
    ## 95 percent confidence interval:
    ##        inf        sup 
    ## -0.4013111  0.5908169

We see that the difference between them in the sample was classified as
"negligible." Further, this gives you the same 95% CI from earlier, but
in Cohen's *d* units instead of "points" on the satisfaction scale.
Neat! I won't use this here, given that we can interpret points on a
1-10 satisfaction scale pretty easily. Still, this is a good tool to
have.

#### Final Report

Let's see how we might summarize our final conclusions. Note that this
would be written differently depending on who the audience is (i.e.,
board members, other data scientists). I present the most frank
(non-hyped-up) version of the analysis here.

First, I would begin with a visualization of the data:

    ggplot(data=dat2[!is.na(dat2$logo), ], aes(x=logo, y=sentiment, fill=logo))+
      geom_boxplot(alpha=.20, color="black")+
      geom_jitter(alpha=.5, color="black", fill="grey90", width=.20)+
      theme_light()+
      scale_y_continuous(name="Sentiment")+
      scale_x_discrete(name="Logo")+
      scale_fill_discrete(name="Logo")+
      ggtitle("Sentiment: Logo A vs. Logo B")

![](Mod5_Lab3_-_Experiments_files/figure-markdown_strict/unnamed-chunk-20-1.png)

Second, I present the results. Generally, I try to state what we
observed first, quantifying the size of the observed results. Second, I
try to state something about whether the effect is significant, using
common language where possible. Third, I try to present some measure of
uncertainty (e.g., our CI), interpreting in plain language as much as
possible. Here is a paragraph possibility:

> The average sentiment for Logo A was slightly higher (*M* = 8.57, *SD*
> = 1.27) than the average sentiment for Logo B (*M* = 8.44, *SD* =
> 1.51). This was a small difference...only 0.13 points on the 1-10
> scale. Given our sample size, this was well within what would be
> expected by random chance (i.e., not statistically significant; *t* =
> 0.38, *p* = 0.70). Thus, they were statistically tied.

> It is possible that a small sentiment difference may still exist
> between the two logos. We are 95% confident that the difference is
> somewhere between 0.80 points higher for Logo A and 0.55 pionts higher
> for Logo B. So, if they do differ, they are likely very close to one
> another; however, we woudl need a larger sampel to say something more
> precise than that.

#### Addendum: Power Considerations

We are done with the analysis. However, it's worth briefly chatting
about power. Did our study have much ability to detect a difference
between Logo A and Logo B in the first place? Was it set up to fail?

Recall that power depends on both sample size and effect size. Large
samples can detect small effects. Smaller samples cannot detect small
effects.

We simply feed information about our study (our sample size and request
for 80% power) to the `pwr.t2n.test()` command from the `pwr` package.
It returns to us the smallest effect we could reliably detect.

    # First, ask R for our sample sizes
    table(dat2$logo)

    ## 
    ## Logo A Logo B Logo C 
    ##     33     32      0

    # Feed information inot pwr.t2n.test()
    pwr.t2n.test(n1=33, n2=32, power=.80)

    ## 
    ##      t test power calculation 
    ## 
    ##              n1 = 33
    ##              n2 = 32
    ##               d = 0.7058882
    ##       sig.level = 0.05
    ##           power = 0.8
    ##     alternative = two.sided

Aha. We could **ONLY** reliably detect effects as small as *d* = .71 ...
a "moderate-large" effect. So, this reinforces our conclusion above...
we would have detected a moderate-large difference between Logo A and B,
but if there was a smaller difference, we were prone to miss it.

Why bring this up? Because I want to remind you that a non-significant
result *does not mean the effect was exactl zero*. It would be a mistake
to say "there was NO difference between Logo A and Logo B." Instead, as
we stated above, we should say "there was not a large difference between
Logo A and Logo B." This reinforces that, but you wouldn't need to
report this.

However, this *would* be worth doing prior to running the study as we
plan our sample size. What if we wanted to be able to detect a smaller
difference, say *d* = .20 with our 80% power? In this case, enter the
*d* value and leave out the *n* values from the power command. I use the
`pwr.t.test()` variant which makes the group sizes equal:

    pwr.t.test(d=.20, power=.80)

    ## 
    ##      Two-sample t test power calculation 
    ## 
    ##               n = 393.4057
    ##               d = 0.2
    ##       sig.level = 0.05
    ##           power = 0.8
    ##     alternative = two.sided
    ## 
    ## NOTE: n is number in *each* group

Here we see that we would need nearly 393 people *per logo* to detect
small differences. If we were planning the study from the start, we
might iteratively explore several possible sample sizes to find the
optimal balance between sample size (i.e., cost of the study) and
ability to detect an effect.

Example 1: Inferential Test for Three Groups
============================================

The previous example considered only two groups: Logo A and Logo B.
However, we had three logos in our study.

When three or more groups are present, we switch from a *t*-test to
*ANOVA* (ANalysis Of VAriance).

The ANOVA is a two stage analysis. In the first stage, we test whether
we can the logo mattered at *all* (i.e., null hypothesis says there were
*no* differences:
*H*<sub>0</sub> : *μ*<sub>*A*</sub> = *μ*<sub>*B*</sub> = *μ*<sub>*C*</sub>).
If that comes up significant, we can systematically compare the logos.

In this example, I assume you are familiar with ANOVA. An overview of
the math behind ANOVA goes beyond the scope of this lab. Essentially,
ANOVA computes the *F* statistic, which yields a *p*-value for the null
hypothesis that all the groups are equal. If we can reject that null
hypothesis, it means the logo is doing something. I illustrate this
next.

In R, ANOVA is relatively easy to test. I illustrate the base R version
in this lab. In base R, we use the `aov()` command to run the model,
which functions the same as the `t.test()` command. We first save the
result and then request a `summary()` of the model. **Note that this
will give incorrect results if `dat$logo` is not a factor**.

    model1 <- aov(dat$sentiment ~ dat$logo)
    summary(model1)

    ##             Df Sum Sq Mean Sq F value   Pr(>F)    
    ## dat$logo     2  51.23  25.617   13.61 6.25e-06 ***
    ## Residuals   96 180.68   1.882                     
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 1 observation deleted due to missingness

This first step tests the null hypothesis that *all the group means are
equall* ... in other words that the logo doesn't matter **at all**. In
this case, the *p* value is 6.25e-06 (aka 0.00000625). This is highly
statistically significant (*p* &lt; .05). By definition, this means that
we could only get a logo effect this big 0.000625% of the time by random
chance alone. So we are confident the logo is doing something.

Why did we bother with the test described above? Consider what would
happen had we done several *t*-tests. There were three possible
*t*-tests we could have conducted, each of which has the possibility of
giving us a false positive. If we were to run *many* tests, we would
likely stumble across a false-positive eventually. Thus, we appreciate
the ability to conduct a **single** overall test to tell us our logo is
doing something.

The next step is to figure out where the differences are. Recall we made
a table of results earlier with `tapply()`.

To avoid the issue described above, we use a special test (there are
many; I cover Tukey's HSD here) that takes into account the number of
comparisons we want to make. We wish to make 3 comparisons:

1.  A vs. B
2.  A vs. C
3.  B vs. C

Tukey's test knows how many comparisons we want to make. It inflates our
*p*-values on each test, making it harder to detect a difference for any
one comparison. This *reduces* our likelihood of making a false positive
on any one comparison, counteracting the problem of running multiple
tests. Neat!

It is easy to run. Simply feed your ANOVA model into: `TukeyHSD()`:

    TukeyHSD(model1)

    ##   Tukey multiple comparisons of means
    ##     95% family-wise confidence level
    ## 
    ## Fit: aov(formula = dat$sentiment ~ dat$logo)
    ## 
    ## $`dat$logo`
    ##                     diff        lwr        upr     p adj
    ## Logo B-Logo A -0.1320076 -0.9422799  0.6782647 0.9205043
    ## Logo C-Logo A -1.5757576 -2.3738391 -0.7776761 0.0000257
    ## Logo C-Logo B -1.4437500 -2.2481347 -0.6393653 0.0001337

Of note here:

1.  Group A vs. B = not significant (*p* = .92)
2.  Group A vs. C = highly significant (*p* = 0.0000257)
3.  Group B vs C = highly significant (*p* = 0.0001337)

The output above also gave us difference scores + 95% CIs. Very handy:

1.  Logo B was 0.13 points lower than Logo A. We are 95% confident the
    true difference is between -0.94 and 0.68. This includes zero, so we
    cannot declare a difference between Logo B and Logo A.

2.  Logo C was 1.58 points lower than Logo A. We are 95% confident the
    true difference is between -2.37 and -0.78.

3.  Logo C was 1.44 points lower than Logo A. We are 95% confident the
    true difference is between -2.25 and -0.64.

As in the *t*-test example above, we see the confidence intervals are
helpful. If we wanted more precise CIs, we need a larger sample.

As a reminder, if we needed Means and SDs, we could easily use
`tapply()`:

    #Make Mean and SD Table
    tab.1 <- rbind(
      round(tapply(dat$sentiment, dat$logo, mean, na.rm=T), 2),
      round(tapply(dat$sentiment, dat$logo, sd, na.rm=T), 2)
    )

    tab.1 <- t(tab.1)
    colnames(tab.1) <- c("M", "SD")
    tab.1

    ##           M   SD
    ## Logo A 8.58 1.27
    ## Logo B 8.44 1.51
    ## Logo C 7.00 1.33

### Final Report

A report might look as follows:

> The average sentiment on a 1-10 scale is as follows:

    tab.1

    ##           M   SD
    ## Logo A 8.58 1.27
    ## Logo B 8.44 1.51
    ## Logo C 7.00 1.33

> An ANOVA revealed a signifiant effect of logo on sentiment (*F* =
> 13.61, *p* &lt; .001). As shown in the table above, Logos A and B were
> similar in sentiment, while Logo C had reduced sentiment. The pattern
> of results was such that Logos A and B were statistically tied, and
> both performed better than Logo C.

Given the number of results, one could then report the CIs or *p*-values
above in a table or in paragraph form. It is possible to go into the
same level of detail as for the *t*-test, but people usually don't, as
it becomes exponentially more complicated with all the comparisons going
on.
