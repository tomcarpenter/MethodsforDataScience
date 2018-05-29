Module 4, Lab 2 - Validity
==========================

In this lab, we build on the previous lab to validate our measure of
consumer sentiment. In the previous lab, we found that we could make a
reliable, one-factor measure that we think represents consumer
sentiment. However, we did not show that the measure was actually
measuring sentiment. In this lab, we discuss the concept of measurement
validity--ensuring that the measure actually captures what it claims to.

There is no sure-fire way to ensure that a measure is valid. However,
there are some things we can consider.

1.  Face validity: does the measure look valid, at face level? This is
    subjective, but it is important. For example, in the previous lab,
    we captured people's feelings about a taco brand with reported
    rating on four adjectives: "inviting," "friendly," "awesome," and
    "pleasant." We asked people to rate how well each adjective
    describes the brand on a 1-10 scale. We can think through whether
    these seem like they would be capturing sentiment. Clearly, a person
    who rates a brand highly on "awesome" feels positive toward the
    brand, right? Well, we can think through this a bit more. What other
    reasons might a person respond positively on that question? Might
    they just be an enthusiastic person? Perhaps someone who wants to
    make a good impression or feels social pressure from the survey to
    give a positive response? Next, we might consider whether those
    issues would be shared among all the adjectives. Indeed, it seems
    possible. There are still some other issues we might consider. For
    example, do people really know and have the ability to report their
    attitudes and feelings toward a brand accurately? Might something
    else such as an analysis of their natural language be better? In
    short, this measure *looks* face valid, but can also conceive of
    some potential threats to its validity. We then would have to ask
    whether those issues are big enough to warrant not using it.

2.  Content validity: does the measure have the appropriate breadth?
    With this set of adjectives, we might consider other words that
    should be included, or we might consider removing some that don't
    belong. This is also subjective. Indeed, in the last lab we were
    considering discarding "quirky" before the data analysis came in.

3.  Criterion validity: does the variable correlate in ways that a good
    measure should? This is the "data-driven" option, but it is not
    without issues as well. For example, it presumes that you have
    picked a good set of outcomes to correlate it with and that those
    are also measured validly. Still, assuming you are collecting data,
    it doesn't hurt to collect a little bit more or to check this in the
    data that you have collected.

In this lab, I briefly demonstrate **criteiron validity** by checking
correlations among measures.

Load Data
=========

In this lab, I use a slightly different measure of sentiment but a
similar research design. A research team has a new sentiment measure,
and they wish to know if it is criterion valid. To validate it, they
have assembled data on the number of positive words used to describe a
product (word count, or `WC`), the rating of a product using a standard
rating system (`rating`), and the anticipated likelihood of purchase
(`purchase`). The data are in the github folder for this lab:

    #### LOAD DATA ####
    dat <- read.csv("validity.csv")

We can briefly check the data. Notice there is an id variable called `X`
as well.

    str(dat)

    ## 'data.frame':    250 obs. of  5 variables:
    ##  $ X       : int  1 2 3 4 5 6 7 8 9 10 ...
    ##  $ sent    : int  5 10 8 6 9 10 4 12 6 4 ...
    ##  $ WC      : int  3 1 2 4 2 3 4 3 2 2 ...
    ##  $ rating  : int  5 6 7 6 7 5 4 8 5 3 ...
    ##  $ purchase: int  2 4 3 3 4 4 3 4 3 1 ...

    head(dat)

    ##   X sent WC rating purchase
    ## 1 1    5  3      5        2
    ## 2 2   10  1      6        4
    ## 3 3    8  2      7        3
    ## 4 4    6  4      6        3
    ## 5 5    9  2      7        4
    ## 6 6   10  3      5        4

Is the sentiment measure criterion valid? We can test this by simply
assessing correlations.

    #drop the ID variable
    dat <- dat[,-1]

    #correlations
    round(cor(dat), 2)

    ##          sent   WC rating purchase
    ## sent     1.00 0.30   0.69     0.55
    ## WC       0.30 1.00   0.15     0.05
    ## rating   0.69 0.15   1.00     0.32
    ## purchase 0.55 0.05   0.32     1.00

We see here that the sentiment variable correlates at .30 with word
count, .69 with a product rating, and .55 with purchase likelihood. We
also see that those correlations are larger than they are between the
other measures (e.g., the `rating` variable is not correlating more
strongly with the other variables).

We can also use `corr.test()` in the `psych` package for *p*-values.

    round( psych::corr.test(dat)$p, 3)

    ##          sent    WC rating purchase
    ## sent        0 0.000  0.000    0.000
    ## WC          0 0.000  0.043    0.408
    ## rating      0 0.021  0.000    0.000
    ## purchase    0 0.408  0.000    0.000

The bottom triangle (below the diagonal) represents uncorrected
p-values. We see that everything for sentiment is &lt; .05 (coming up at
zero).

Although this is a short exercise, it is critically important. Not only
can you quickly see if a measure is valid, but you might also find a
better measure. For example, if the product rating was correlating with
purchase likelihood *better* than the sentiment measure, I would
question the use of the sentiment measure for making decisions.
