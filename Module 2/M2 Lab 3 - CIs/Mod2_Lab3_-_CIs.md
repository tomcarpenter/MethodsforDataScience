Module 2, Lab 3: Confidence Intervals
=====================================

This lab provides a brief extension on the previous lab, pointing out
something you may have missed in the results before. Be sure to have the
data from the previous lab open in your computer.

Moving Beyond *p* &lt; .05
==========================

In the previous lab, we tested the null hypothesis. We specifically
asked: "what percentage of the time could you get your result if the
null were true?" We called that percentage a *p*-value. The lower the
*p*-value, the less easily it is that you could get that result if the
null were true. In other words, a low *p*-value means that the data and
the null are incompatible. Since we actually did get the data, we reject
the null. This is a useful way to test the null hypothesis, but it's not
the only way.

Consider one weakness of the procedure outlined above. The null
hypothesis states that the sample result is *entirely* due to chance; in
the population the effect is completely absent (e.g., in the attitude
example in the previous lab, the null said that the average attitude was
exactly zero, or *H*<sub>0</sub> : *μ* = 0, meaning that any non-zero
result observed in the sample was entirely due to chance). When we
reject the null hypothesis, we are rejecting the premise that the
results are entirely due to chance. That's about it.

In other words, rejecting the null hypothesis tells you what the effect
is *not* (i.e., not zero, not due to chance). You aren't saying what the
effect *is.* If you want to estimate what is going on in the population,
we need to do more, to use our sample estimate as a starting point and
add some measure of uncertainty.

Confidence Intervals
====================

This is the goal of confidence intervals. A confidence interval simply
takes your sample result and puts it at the center of a ranged estimate.
We know that the sample result has error in it; it's an imperfect
estimate of the population. However, we can estimate *how much error*
and can use that to create a range around our sample estimate. I will
show you an example first, and then I will show you how it works.

Load the `attitude` data from the previous lab (re-run the code from the
previous lab if you need). We observed a sample average attitude of 2.29
(on a scale from -5 to +5):

    mean(attitude)

    ## [1] 2.293718

This was clearly in the positive range, and it was significant, meaning
that we could reject the null hypothesis that it was *zero* in the
population (i.e., sample result a fluke due to chance).

However, we can also say on the basis of our sample data with 95%
confidence that the population average is somewhere in the range from
1.93 to 2.65. This is often written: "95% CI \[1.93, 2.65\]." Look back
at the last lab: this information is given to you in lower third of the
`t.test()` output:

    t.test(attitude)

    ## 
    ##  One Sample t-test
    ## 
    ## data:  attitude
    ## t = 12.657, df = 99, p-value < 2.2e-16
    ## alternative hypothesis: true mean is not equal to 0
    ## 95 percent confidence interval:
    ##  1.934150 2.653286
    ## sample estimates:
    ## mean of x 
    ##  2.293718

Notice the code in the middle:

`95 percent confidence interval:`  
`1.934150 2.653286`

This is **much** more useful than our *p* &lt; .05 finding. Think about
it. *p* &lt; .05 tells us that we can reject the null hypothesis (which
stated *μ* = 0). The 95% confidence interval tells you that it's
*probably not zero* **and** that it's probably between 1.93 and 2.65. It
both rejects the null and tells you where the population value likely
is. It's rejecting the null and giving you additional information.

The CI also tells you how precise our result is. The two bounds on the
CI are close together, indicating I have a very good read on where the
population mean is. On a -5 to +5 scale, the CI tells me we are very
likely in the "mid-range positive" territory on our attitude scale.

Consider what would happen if the range were bigger. Imagine, for
example, that you saw this instead:

`## 95 percent confidence interval:`  
`##  0.495878 4.091558`

On our -5 to +5 attitude scale, this would be telling you that you are
95% confident that the mean of the population is somewhere between .49
(almost zero) and 4.09 (incredibly strongly positive). This tells us
something (we are still sure it's in the positive range; i.e.,
significant or not zero), but it tells you little else. It's almost
useless for decision making. Yes, people feel positive, on average.
However, in this case, your result is so imprecise that you really can't
say much beyond "it's not zero."

Therefore, I always advocate for confidence intervals. They are given to
you in almost all statistical output in any statistical or data science
tool. They let you quantify exactly how much certainty you have in your
result.

Where does this range come from? We start with the sample estimate and
add or subtract a margin of error. The margin of error, in turn,
consists of the standard error (discussed in the sampling lab)
multiplied by a scaling constant. For our mean, this looks as follows:

$$CI = \\bar{x} \\pm \\left (SE\\right )\\left (constant\\right )$$
 The purpose of the constant is to make the range big enough that you
are 95% confident it contains the true population value. Recall that the
standard error (SE) tells you how much error you expect in a typical
sample, or how much error you expect on average. However, that would not
give you a range that would engender 95% confidence. If you want the
range to be large enough **that it contains the population value 95% of
the time**, you need to add the scaling constant. (In case you've had
some statistics training: this constant, it turns out, is actually the
"critical value" for the *t*-test --not discussed in this class, but you
may have learned it in a previous statistics course).
