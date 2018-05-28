Module 5, Lab 1 - Correlation
=============================

This final series of labs explores everything you need to execute
projects from start to finish based on a few different analyses.

In this lab, we will explore how to assess relationships between
variables using correlation in R.

In this example, we have a dataset, inspired by a dataset published on
kaggle (<https://www.kaggle.com/unsdsn/world-happiness>). In this
dataset, several regions of the world are compared on dimensions such as
their generosity, happiness, GDP, and so forth.

Load Packages
=============

In this lab, we will use the `ggplot2` package for data visualization,
the `corrplot` package for making visual correlation tables, and the
`psych` package for detecting skew and making correlation matrices.

    #### LOAD PACKAGES ####
    library(ggplot2)
    library(corrplot)

    ## corrplot 0.84 loaded

    library(psych)

    ## 
    ## Attaching package: 'psych'

    ## The following objects are masked from 'package:ggplot2':
    ## 
    ##     %+%, alpha

Load Data
=========

Next, let's load the data.

    #### LOAD DATA ####
    dat <- read.csv("regionalhappy.csv")

Inspecting the dataset, we see the names are a little messy:

    names(dat)

    ## [1] "Happiness.Score"               "Economy..GDP.per.Capita."     
    ## [3] "Family"                        "Health..Life.Expectancy."     
    ## [5] "Freedom"                       "Generosity"                   
    ## [7] "Trust..Government.Corruption." "Dystopia.Residual"

We can rename them easily with `names()`:

    names(dat) <- c("Happiness", "GDP", "Family", "Life.Expect", "Freedom", "Generosity", "Trust.Gov", "Dystopia")

Let's look at the first few rows of the data:

    head(dat)

    ##   Happiness      GDP   Family Life.Expect   Freedom Generosity Trust.Gov
    ## 1     7.537 1.616463 1.533524   0.7966665 0.6354226  0.3620122 0.3159638
    ## 2     7.522 1.482383 1.551122   0.7925655 0.6260067  0.3552805 0.4007701
    ## 3     7.504 1.480633 1.610574   0.8335521 0.6271626  0.4755402 0.1535266
    ## 4     7.494 1.564980 1.516912   0.8581313 0.6200706  0.2905493 0.3670073
    ## 5     7.469 1.443572 1.540247   0.8091577 0.6179509  0.2454828 0.3826115
    ## 6     7.377 1.503945 1.428939   0.8106961 0.5853845  0.4704898 0.2826618
    ##   Dystopia
    ## 1 2.277027
    ## 2 2.313707
    ## 3 2.322715
    ## 4 2.276716
    ## 5 2.430182
    ## 6 2.294804

Much better.

Bivariate Correlation
=====================

Next, let's see how variables correlate. In our research study, we want
to understand happiness. We can compute correlations between variables
with `cor()`:

    cor(dat$Happiness, dat$Life.Expect)

    ## [1] 0.7819506

We see the correlation is *r* = .57. A brief refresher: correlations
range between zero (no association between variables) and 1.0 (a
one-to-one association). They can also be positive (as one variable
increases, so does the other) or negative (as one variable increases,
the other decreases).

So, in this case, we have a large, positive link between the happiness
of a region and health / life expectancy in that region. The
statistician Jacob Cohen suggested the following guidelines:

    ##   correlation    meaning
    ## 1   0.0 - 0.1 Negligible
    ## 2   0.1 - 0.3      Small
    ## 3   0.3 - 0.5     Medium
    ## 4      0.5 +       Large

We see, then, that this association would count as "large" by Cohen's
guidelines.

We can also easily visualize this correlation with `ggplot2`:

    ggplot(data=dat, aes(x=Happiness, y=Life.Expect))+
      geom_point()+
      theme_light()

![](Mod3_Lab1_-_Correlation_files/figure-markdown_strict/unnamed-chunk-8-1.png)

### A Sample Estimate

We have discussed statistical hypothesis testing a number of times in
this course, but we haven't yet discussed it in detail with respect to
correlations. So, here is a brief refresher on the need for significance
testing, applied to correlation.

Here, are working with a sample of regions at one point in time. What if
we wanted to estimate, in a broader way, the association between
happiness and life expectancy? Assuming our data are representative of
the broader population (e.g., across times, regions, etc.; a big
assumption!), we could use this sample correlation (symbol: *r* = .78)
as an estimate of the population correlation (symbol: *ρ*). In other
words, we don't know the true correlation between happiness and life
expectancy in the population, but if we can trust this data to
adequately represent it, we can *estimate* it at .78.

The estimation piece is important. Often, people look at the sample
correlation and don't realize that it's specific to that sample. For
example, an organization might collect a survey to assess the link
between customer satisfaction and consumption. Whatever correlation
observed in the sample is only an estimate--our best guess--of the
correlation in the broader population. Were that organization to collect
another sample, they would get a different correlation. Every time, the
correlation would vary slightly, because the sample is different and
only representing (but not being) the population. This raises an
important point: sample correlations are imperfect estimates of their
population counterparts. The sample estimate has error built into it.

One important consequence is that it is possible that the correlation in
the population is actually zero (*H*<sub>0</sub> : *ρ* = 0) even when it
is not in the sample (e.g., *r* = .12). In other words, the sample
correlation could be a statistical fluke of the sample. We cannot say,
just because the sample correlation is nonzero, that the two variables
truly are correlated in the population. We will need to conduct a
statistical significance test first.

Further, we can *only* trust the sample correlation as an estimate of
the population correlation *if* the data are representative. If only a
certain kind of person selects into the survey (e.g., a certain
personality type, people who have strong feelings about a product,
etc.), then the sample correlation will estimate the correlation for
*that population only.* This essentially means that all research data is
biased toward whoever is over-represented in the sample. For this
reason, getting good information on whoever is represented in one's data
is very important for qualifying the results. In some cases, it may be
worth it to gather data from multiple different sources or methods and
cross-reference the results for very important decisions.

Significance Test
=================

If we want to test the correlation for significance, we can simply
replace `cor()` with `cor.test()`:

    cor.test(dat$Happiness, dat$Life.Expect)

    ## 
    ##  Pearson's product-moment correlation
    ## 
    ## data:  dat$Happiness and dat$Life.Expect
    ## t = 15.517, df = 153, p-value < 2.2e-16
    ## alternative hypothesis: true correlation is not equal to 0
    ## 95 percent confidence interval:
    ##  0.7120831 0.8364830
    ## sample estimates:
    ##       cor 
    ## 0.7819506

Here we see that a *t*-test was conducted.

Recall that a *t*-test compares the size of the *observed correlation*
(*r*) against the value in the null hypothesis (zero), divided by what
is typically expected by chance:

$$t=\\frac{result - null }{chance}$$

The top of the faction is key here. The more the data "disagree" with
the null, the larger the *t*-value and hence more evidence for a real
association When all is said and done, a large *t*-value tells you that
the association (top of fraction) is considerably larger than expected
by chance (bottom of fraction). Here, our *t* value is 15.52, meaning
that the correlation is 15.52 times larger than would typically be
expected by chance. Thus, chance seems an unlikely explanation for the
result.

This is converted in to a *p*-value, which R tells us is very small. In
other words, it would be exceedingly unlikely to get a result this big
if the null were true. Because it is &lt; .05, we can say that this
result would happen very rarely by chance. We reject the null hypothesis
and conclude that this correlation is not due to chance.

Using the same information, we can make a reasonable guess about what
the correlation in the population is. We see that `cor.test()` has given
us a 95% confidence interval of \[0.71, 0.84\], meaning that we are 95%
confident that the population value (*ρ*) is in that range. By "95%
confident," we mean that this range includes the population value 95% of
the time. If we act on it and trust it, we are right 95% of the time.

So, we are pretty certain that, even though we have a sample (a small
sample, too!), that there is a large correlation in the population
between happiness and life satisfaction. Even after taking the
uncertainty of our sample into account (e.g., with the *t*-test and 95%
CI), we still feel confident that there is a larger link between these
two variables.

Caveat: Normality
=================

It should be noted that correlations work best with normally distributed
(bell curve, symmetrical) data. We can briefly check the skew of the
variables. I won't use `ggplot2` here since we aren't making figures
we'd want to share:

    hist(dat$Happiness)

![](Mod3_Lab1_-_Correlation_files/figure-markdown_strict/unnamed-chunk-10-1.png)

    hist(dat$Life.Expect)

![](Mod3_Lab1_-_Correlation_files/figure-markdown_strict/unnamed-chunk-10-2.png)

We see here that both variables are decently normally distributed, but
life expectancy is possibly negative skewed (i.e., "skew left"). We can
get a metric of the skew using the `skew()` command in the `psych`
package:

    psych::skew(dat$Life.Expect)

    ## [1] -0.5668279

People disagree about what is acceptable skew, but usually a value less
than +/- 1.0 raises no alarms. Here, we can safely go about our
business.

However, if we had a bigger skew problem, we could also address the skew
by transforming the variable. (There are also backup options, such as
Spearman's correlation, but we won't explore that in this class beyond
what was covered in Module 3).

Correlations are based on variance, so anything that biases a mean
(e.g., skew) also interferes with the correlation. In general, skew
reduces correlations. For a more robust test of the correlation, you can
transform the data by performing a mathematical operation to every
score. There are many such operations we can try. In general, taking the
square root of every score reduces skew, but the catch is that the
variable must be positively skewed and no scores may be negative. For
the sake of illustration, let's try this.

First, let's "reverse" the variable. Take the maximum score, add one to
it, and subtract your score:

    dat$Life.Expect2 <- max(dat$Life.Expect) + 1 - dat$Life.Expect

This reverses the variable:

    hist(dat$Life.Expect2)

![](Mod3_Lab1_-_Correlation_files/figure-markdown_strict/unnamed-chunk-13-1.png)

Now we can perform any number of operations. The square root is the most
mild transformation. We can also take the natural log of every score (no
values may be zero!). In general, these operations reduce big numbers
more than small numbers and thus rein in the long tail:

    hist(sqrt(dat$Life.Expect2))

![](Mod3_Lab1_-_Correlation_files/figure-markdown_strict/unnamed-chunk-14-1.png)
We could save this transformed version, then re-reverse it and use that
in subsequent analyses.

    dat$Life.Expect2 <- sqrt(dat$Life.Expect2)
    dat$Life.Expect2 <- max(dat$Life.Expect2) + 1 - dat$Life.Expect2

    cor.test(dat$Happiness, dat$Life.Expect2)

    ## 
    ##  Pearson's product-moment correlation
    ## 
    ## data:  dat$Happiness and dat$Life.Expect2
    ## t = 15.747, df = 153, p-value < 2.2e-16
    ## alternative hypothesis: true correlation is not equal to 0
    ## 95 percent confidence interval:
    ##  0.7177367 0.8399223
    ## sample estimates:
    ##       cor 
    ## 0.7864035

We see here the results barely changed. In this case, that wasn't really
necessary because the variable was not that skewed to begin with. You
will find, in many cases, that it is very helpful, however. In those
cases, there are many online guides to data transformation.

Of course, if you are predicting something that has a very non-normal
distribution (e.g., categorical variables, counts of things often have
many zeros, etc.) then correlation may not be the best tool to use. A
more sophisticated data modeling technique may be warranted. However,

Correlations Among Many Variables
=================================

Often we want to examine the correlations among many variables at once.
In this case, we could look at a matrix of correlations. This can be
done by inputting a data.frame of all the variables of interest into the
`cor()` function:

You can make your own data.frame of just the variables you want to
analyze, or we could simply use `dat`. Here, I want happiness, life
expectancy, GDP, and generosity:

    cor(data.frame(dat$Happiness, dat$Life.Expect, dat$GDP, dat$Generosity))

    ##                 dat.Happiness dat.Life.Expect     dat.GDP dat.Generosity
    ## dat.Happiness       1.0000000      0.78195062  0.81246875     0.15525584
    ## dat.Life.Expect     0.7819506      1.00000000  0.84307664     0.06319149
    ## dat.GDP             0.8124688      0.84307664  1.00000000    -0.01901125
    ## dat.Generosity      0.1552558      0.06319149 -0.01901125     1.00000000

We can easily see that happiness, life expectancy, and GDP are all
highly intercorrelated, whereas generosity is seemingly less related.
Thus, we may conceive that we have a "cluster" of intercorrelated
variables around happiness.

If you wanted significance tests, you can request that with the
`corr.test()` function in the `psych` package, which returns a grid of
*p*-values. This will also give you other contents; we can request only
the *p*-values by adding `$p` at the end.

    corr.test(data.frame(dat$Happiness, dat$Life.Expect, dat$GDP, dat$Generosity))$p

    ##                 dat.Happiness dat.Life.Expect  dat.GDP dat.Generosity
    ## dat.Happiness       0.0000000       0.0000000 0.000000      0.1611975
    ## dat.Life.Expect     0.0000000       0.0000000 0.000000      0.8694396
    ## dat.GDP             0.0000000       0.0000000 0.000000      0.8694396
    ## dat.Generosity      0.0537325       0.4347198 0.814369      0.0000000

Note that only the correlations below the diagonal are correct. The ones
above the diagonal have been adjusted, but we won't deal with that here.
We do see that generosity is not significantly correlated with the other
variables.

#### Clustering

We saw before we had a cluster of several overlapping variables. We can
make some helpful visuals to view this cluster. The `heatmap()` command
highlights highly intercorrelated variables. It also produces a
dendrogram to show the clustering. I will do this for the full dataset,
less the `Life.Expect2` variable (which is redundant to the
non-transformed version). This is column 9, so I can easily drop it by
calling `dat[,-9]`:

    # save cor matrix
    cors <- cor(dat[,-9])

    #heatmap
    heatmap(cors, symm = TRUE)

![](Mod3_Lab1_-_Correlation_files/figure-markdown_strict/unnamed-chunk-18-1.png)

Here we see there is a prominent cluster of happiness, family, life
expectancy, and GDP. These are all highly inter-correlated. You can see
this from the bright spot in the histogram as well as from the "long"
stem on the dendrogram for those variables (indicates greater
clustering).

#### A Correlation Plot

Another great visual is the correlation plot. This visual helps to map
out the sample correlations but with a greater emphasis on the
individuals relationships (rather than clustering).

There are many great tutorials for using the `corrplot` package online.
In general, like `heatmap()`, it accepts your correlation grid as an
input:

    corrplot(cors)

![](Mod3_Lab1_-_Correlation_files/figure-markdown_strict/unnamed-chunk-19-1.png)

By default, it replaces the numbers in the grid with shapes
(lager/darker = stronger correlation; blue = pos, red = neg). Unlike
`heatmap()`, it does not sort the variables into clusters for us.
However, it has a lot of flexibility. Using `corplot.mixed()` you can
request numbers in one diagonal. You can also sort by the clustering
algorithm as in `heatmap()` by adding `order="hclust"`:

    corrplot.mixed(cors, order="hclust")

![](Mod3_Lab1_-_Correlation_files/figure-markdown_strict/unnamed-chunk-20-1.png)

We can also add a grid of *p*-values with `p.mat=` and tell the function
to "X" out anything not &lt; .05 by adding `sig.level = .05`:

    corrplot.mixed(cors,
                   p.mat=corr.test(dat[,-9])$p, sig.level = .05)

![](Mod3_Lab1_-_Correlation_files/figure-markdown_strict/unnamed-chunk-21-1.png)

There you have it. There are many great ways to illustrate correlations
among data.

Conclusion
==========

Using the correlation analysis, we have both learned to find clusters of
relationships among data and to estimate individual correlations and
test them for significance. If we had a specific variable we wanted to
study in greater detail, we could graduate to regression, which we will
do in the next lab.
