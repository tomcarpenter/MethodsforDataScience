Module 5, Lab 2 - Regression
============================

In this lab, we continue from where we were in the previous lab. In case
it's been a little while, I will re-load the data and get you up and
running.

Regression is used when you seek to *understand* or *predict* a given
outcome variable in greater depth. One problem with correlation, as we
saw in the previous lab, is that many of our variables overlap. In
regression, we can isolate the unique or non-overlapping relationships
among our variables. This is very powerful, as it allows us to
understand what our variables are doing in a new way.

In this example, we have a dataset, inspired by a dataset published on
kaggle (<https://www.kaggle.com/unsdsn/world-happiness>). In this
dataset, several regions of the world are compared on dimensions such as
their generosity, happiness, GDP, and so forth.

In this lab, we will use the `ggplot2` package for data visualization,
the `lm.beta` package to give standardized regression coefficients, the
`corrplot` package for making visual correlation tables, and the `psych`
package for detecting skew and making correlation matrices.

    #### LOAD PACKAGES ####
    library(ggplot2)
    library(lm.beta)
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

Regression Concepts
===================

Recall that the purpose of regression is to predict one variable. In
this case, let's predict `Happiness`. In regression, this is done by
generating an equation that predicts the outcome from one or more
predictors. The equation looks like this for a single predictor:

*Y*′=*b*<sub>0</sub> + *b*<sub>1</sub>*x*<sub>1</sub>
 In this regression equation, we can guess each person's Y score (*Y*′)
by starting with a constant (i.e., intercept, *b*<sub>0</sub>) and
adding to that their score on a predictor variable (*x*<sub>1</sub>),
multiplied by its respective slope (*b*<sub>1</sub>).

We need not only use one predictor:

*Y*′=*b*<sub>0</sub> + *b*<sub>1</sub>*x*<sub>1</sub> + *b*<sub>2</sub>*x*<sub>2</sub> + *b*<sub>3</sub>*x*<sub>3</sub> + *b*<sub>4</sub>*x*<sub>4</sub>...
 In regression, you can have as many predictors as you want, and each
one you add (assuming it's useful), should improve the accuracy of the
prediction. As you can imagine, this is immensely useful in many data
science applications, as it literally provides the ability to predict
the future.

For example, imagine we wanted to predict `Happiness` from a series of
some of our variables:

*H**a**p**p**y*′=*b*<sub>0</sub> + *b*<sub>1</sub>*G**D**P* + *b*<sub>2</sub>*F**a**m**i**l**y* + *b*<sub>3</sub>*L**i**f**e**E**x**p**e**c**t* + *b*<sub>4</sub>*F**r**e**e**d**o**m* + *b*<sub>5</sub>*G**e**n**e**r**o**s**i**t**y*

In this regression equation, we can guess each person's `Happiness`
score by starting with the constant (i.e., intercept, *b*<sub>0</sub>)
and adding to that their score on each of the other variables (GDP,
Family, Life.Expect, Freedom, Generosity), multiplied by its respective
slope
(*b*<sub>1</sub>, *b*<sub>2</sub>, *b*<sub>3</sub>, *b*<sub>4</sub>, *b*<sub>5</sub>).

In addition to helping to predict an outcome, regression is also useful
for *understanding* the variables. By definition, the slope is the
expected increase in the outcome (e.g., expected gain in happiness) for
every one unit of that predictor. For example, if the slope for `GDP`
was 2.18, then by definition I expect to gain an additional 2.18 units
of happiness for every one unit of GDP. The larger the slope, the more
"potent" the variable. A large slope means you get a *lot* of the
outcome (e.g., a lot of happiness) out for every one unit of the
predictor. Conversely, if the slope were zero, the predictor literally
would not matter; adding more of that predictor would have no impact on
the (prediction of the) outcome. So, the slope can be though of as the
importance of that variable for your outcome.

In a regression analysis, all that is needed is the set of *b*s.

Let's try this with a basic regression with one predictor. Let's predict
`Happiness` from `GDP`.

A Simple Regression
===================

To fit a regression, all we do is ask R to calculate the slope(s) and
intercept. This is done with the code:
`lm(outcome ~ predictor, data=dat)`. Note that you do not need to list
them as `dat$outcome` although you can. There are good reasons not to,
however. It's better to have all the variables in one data frame and do
it this way.

    lm(Happiness ~ GDP, data=dat)

    ## 
    ## Call:
    ## lm(formula = Happiness ~ GDP, data = dat)
    ## 
    ## Coefficients:
    ## (Intercept)          GDP  
    ##       3.203        2.184

Thus, our regression equation is as follows:

> Happiness' = 3.2 + 2.18(GDP).

That is, we can take anyone's `GDP` score and plug it into this equation
to predict their happiness. To get a sense as to what a "high" happiness
scores is, let's inspect its distribution:

    range(dat$Happiness)

    ## [1] 2.693 7.537

    hist(dat$Happiness)

![](Mod3_Lab2_-_Regression_files/figure-markdown_strict/unnamed-chunk-8-1.png)
Happiness scores tend to range from ~ 2-8.

Let's look at the distribution of `GDP` scores:

    hist(dat$GDP)

![](Mod3_Lab2_-_Regression_files/figure-markdown_strict/unnamed-chunk-9-1.png)

GDP scores tend to range from 0-2 on this scale.

What if I had a relatively low `GDP` score of say, .5? My predicted
Happiness score would be ...

> Happiness' = 3.2 + 2.18 \* 0.5

Try it yourself. What do you get as the predicted loneliness score?

> Loneliness' = 4.29.

Side note: there is an easy way to get R to do this for you. You simply
use `predict()`, and input the name of your model and a `data.frame()`
giving scores on the predictors:

    mod <-lm(Happiness ~ GDP, data=dat)

    predict(mod, data.frame(GDP=0.5))

    ##        1 
    ## 4.295305

This is handy. We can also give it several GDP scores (0.5, 0.9, 1.7)

    predict(mod, data.frame(GDP=c(0.5, 0.9, 1.7)))

    ##        1        2        3 
    ## 4.295305 5.168979 6.916327

This tool is often used in the real world when you have trained a model
using old data and want to then use it to make guesses for new data.
It's easy to save or download a data frame with information on many new
people and place it into `predict()`. Once the model is trained, you now
have a valuable tool for making predictors for any new person for whom
you have data.

We can also easily visualize this relationship. Because it's simply an
intercept and slope, the predictors form a straight line:

    ggplot(aes(x=GDP, y=Happiness), data=dat)+
      geom_jitter()+
      geom_smooth(method=lm)+
      theme_light()+
      scale_x_continuous(name="GDP")+
      scale_y_continuous(name="Happiness")

![](Mod3_Lab2_-_Regression_files/figure-markdown_strict/unnamed-chunk-12-1.png)

This represents the predictions we generated earlier (e.g., for a GDP of
0.5, the line is at height of 0.5). The grey band represents a 95% CI
around the line. This represents the uncertainty in the steepness of our
slope. We are 95% confident that the "true" slope in the population is
somewhere in that grey band (you can imagine the blue line swiveling a
bit within that grey band). We can easily see the relationship is
"significant" (i.e., not zero, not flat) since the most shallow line we
could fit into that grey band is still going upward. In other words, we
are confident that the slope *is* going upwards (i.e., is not zero). If
we had a null hypothesis that the population slope (often symbolized
*β*) was zero (e.g., *H*<sub>0</sub> : *β* = 0), we could reject that.
This is more informative than simply giving us a *p*-value for the
slope; in this case, we know the slope is not zero *and* that it is
still pretty steep, even at is most shallow plausible value. Of course,
we can get a *p*-value for that slope and more, which we will see in
just a moment.

Inspecting the Regression
=========================

We should check our regression in more detail, see if it satisfies
assumptions, and see if it's a good fit to the data. To do all this, we
first save the regression model.

    mod <- lm(Happiness ~ GDP, data=dat)

To get information about your regression model, we simply ask for the
`summary()` of `mod`.

    summary(mod)

    ## 
    ## Call:
    ## lm(formula = Happiness ~ GDP, data = dat)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -1.88807 -0.45200 -0.05328  0.49425  1.89833 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)   3.2032     0.1356   23.62   <2e-16 ***
    ## GDP           2.1842     0.1267   17.24   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.6617 on 153 degrees of freedom
    ## Multiple R-squared:  0.6601, Adjusted R-squared:  0.6579 
    ## F-statistic: 297.1 on 1 and 153 DF,  p-value: < 2.2e-16

First, to be clear, there are *two* parameters in the regression model:
the intercept and the slope. Each exists in the population and we are
estimating them in our sample.

This information is found in the middle, where it says 'coefficients'.
There are two rows, one for the intercept and one for the slope. We see.
For our data, we estimate the intercept is 3.2032127. However, this is
just an estimate of the "real intercept" in the population. There is a
standard error, *t*-test, and *p*-value (in R, called `Pr(>|t|)`) shown.
As we see, the intercept is significant, meaning that we are confident
it is "not zero." Of course, we don't really care about the intercept,
but the information is there.

More interesting is the slope for `GDP`. We see that the slope is
estimated at 2.1841849. Again, that is just an estimate of the "real"
slope in the population. However, we see there is a *t*-test and
*p*-value and again, we see it is significant. (Same information given
to us by the 95% CI "grey band" in the graph above). This *is* useful
information. This tells you that you can be confident that the slope is
"not zero"...in other words, `GDP` is 'predicting' `Happiness` The slope
can also be interpreted directly. For each unit of GDP, we expect
Happiness to change by 2.1841849 units.

Maybe you want a numeric 95% CI for your regression coefficients? This
is also easy:

    confint(mod)

    ##                2.5 %   97.5 %
    ## (Intercept) 2.935283 3.471143
    ## GDP         1.933859 2.434511

(Those values represent 95% CIs... from '2.5% to 97.5%' is a range of
95%).

There are many other pieces of information one can get from the model.
For example, we can find the predicted values with `fitted(mod)` or
`mod$fitted`. We can extract the slopes with `coef(mod)` or
`mod$coefficients`. The number of things we can easily do is too great
to go into detail here, but as a general rule, if there is something you
want to know about this regression model, we can probably get it easily.
The names of all the things you can get can be given with `names()`.
Note that there is different information for a regression if you ask for
the `names()` of the model versus the `names()` of the `summary()` of
the model:

    names(mod)

    ##  [1] "coefficients"  "residuals"     "effects"       "rank"         
    ##  [5] "fitted.values" "assign"        "qr"            "df.residual"  
    ##  [9] "xlevels"       "call"          "terms"         "model"

    names(summary(mod))

    ##  [1] "call"          "terms"         "residuals"     "coefficients" 
    ##  [5] "aliased"       "sigma"         "df"            "r.squared"    
    ##  [9] "adj.r.squared" "fstatistic"    "cov.unscaled"

Now, we are predicting each region's happiness. How accurate are those
predictions? To answer that, we need to discuss residuals.

We see in our graph that not every point is on the line. In other words,
our trend line under-predicts for some and over-predicts for others. The
degree of "mis-prediction" for each observation is called a residual. We
can plot them easily in `ggplot2` by adding a `geom_segment()` and
asking for lines to be drawn between observed scores(`dat$GDP`) and
predicted scores(`mod$fitted`):

    ggplot(aes(x=GDP, y=Happiness), data=dat)+
      geom_jitter()+
      geom_smooth(method=lm)+
      geom_segment(aes(xend = dat$GDP, yend = mod$fitted), alpha=.3)+
      theme_light()+
      scale_x_continuous(name="GDP")+
      scale_y_continuous(name="Happiness")

![](Mod3_Lab2_-_Regression_files/figure-markdown_strict/unnamed-chunk-17-1.png)

On average, these residuals look to be about 1.0 points or so in length.

However, we can compute the "residual standard error" which is basically
the "average residual." Some books call this the "standard error of the
estimate." It is essentially the same idea as the standard deviation:
assessing the average degree to which points vary *aroudn the line* (as
opposed to varying around the mean, in standard deviation). In fact, it
uses the same equation as the standard deviation, but modified so scores
vary around the predicted value (y' ... the line) rather than the mean.
Also, we use "n-2" on the bottom for reasons I won't get into:

$$ RSE = \\sqrt{\\frac{\\sum \\left ( y-y' \\right )^{2}}{n-2}} $$

We see in our our output, "Residual standard error: 0.6617", meaning our
`Happiness` predictions are "off" (on average) by 0.66 points. Given
that the scale is basically 2-7 (ish), this seems pretty good. Of
course, we are *only* using `GDP` to predict scores right now. We could
add other predictors and attempt to make this value smaller (i.e.,
become more accurate in our predictions).

Let's continue exploring our summary output. Residuals are important.
More information about our residuals can be seen up top: the smallest
residual, 1Q (the 25th percentile), the median (the 50th percentile), 3Q
(the 75th percentile), and max. This will come in much handier later.
The median residual is close to zero...that is very important because it
means we are not systematically over-predicting or under-predicting
(phew!). Also importantly, one assumption of a *valid* regression model
is that the regressions are *normally distributed*. Of course, in a
normal distribution, the data are symmetrical, so the lower values (1Q)
should mirror the top values (3Q). We could also simply histogram them:

    hist(mod$residuals)

![](Mod3_Lab2_-_Regression_files/figure-markdown_strict/unnamed-chunk-18-1.png)

These residuals are beautifully normally distributed. This is a good
sign for our model. This means that we tend to under-predict (negative
values) just as much as we over-predict (positive values). This looks
like a healthy and valid regression model.

Finally, we see that the "Multiple R squared" is .66. I won't go into
the math behind that, but it essentially represents the proportion of
the variance in the outcome (`Happiness`) that is explained by the
predictor(s). In other words, we have explained 66% of all the variance
in `Happiness` with just `GDP`. That further reinforces our conclusion
that GDP is a very strong predictor of happiness. Related to the
Multiple R squared is a bit more information:
`F-statistic: 297.1 on 1 and 153 DF,  p-value: < 2.2e-16` ... this is
simply a significance test for the *R*<sup>2</sup> value. We see here
that our model *is* explaining significant variation in happiness. On
level, this is redundant to the test of significance for the slope, but
it because very helpful when there are multiple predictors as a quick
snapshot of the significance of the model.

It's worth noting that the *R*<sup>2</sup> is, basically, the opposite
of the residuals. The residuals represent *unexplained* variation in the
happiness (Y) scores. That is, one region's happiness score may 7, yet
we predict 6 (a residual of -1). This represents some portion of the
score we have not yet explained. On the other hand, the *R*<sup>2</sup>
represents the proportion of variation in `Happiness` that **is
explained.** The better our model, the higher the *R*<sup>2</sup> and
the smaller the residuals. If we could make our residuals zero (i.e.,
perfectly guess every score), our *R*<sup>2</sup> would be 100% (1.00).

I bring this up only because I think it's useful to understand what
these numbers mean. When our goal is *understanding* our variables, then
this information is very important. We are saying we can understand 66%
of what there is to know about happiness with GDP alone.

Multiple Regression
===================

We can improve our predictions by adding more predictors. There are
three reasons to do this.

1.  Our model predicts better given more information to predict with
2.  GDP may be caused by (and therefore serve as a proxy for) other
    variables that are more directly relevant
3.  GDP probably doesn't make people happy directly; it is likely
    causing other things like healthcare that are more directly relevant

Reasons 2 and 3 are very similar, but they differ in an important way.
Reason 2 suggests that something may be driving *both* GDP *and*
happiness. We discussed this possibility in the labs for association /
causal claims. In that case, GDP is acting as a proxy for whatever that
prior cause was...leading us to possibly misunderstand the role of GDP.
We must statistically control for any prior causes if we want a good
view of what effect GDP has on happiness (and even then, we can't know
whether GDP is the cause or the effect).

Reason 3, on the other hand, suggests that GDP may do things that, in
turn, help happiness. For example, GDP may lead to better health, and
that may aid happiness. In this case, we can still still control for
those variables. Doing so will give "credit" for that relationship to
the more proximal variable (i.e., it will appear that health, not GDP
predicts happiness). In such a case, we should try to remember that GDP
may still be ultimately responsible.

Note that there is no way to tell the difference, statistically between
reason 2 and 3. Thus, the data analysis needs subject matter expertise
to draw good conclusions for observational (i.e., correlation /
regression based) data.

#### Controlling Prior Causes

Reason 2 suggests that GDP may be caused by other things and that *those
things*, not GDP are causing happiness. In such a case, failure to
statistically adjust or control for them makes our regression is
misleading, attributing to GDP what should rightly be attributed to
those prior causes. For example, imagine that GDP was caused, in part,
by freedom. In that case, we can't rightly examine the role of GDP
without first *controlling* for freedom. Failure to do so will
misrepresent what GDP is doing, likely inflating its importance.

Importantly, however, you can easily control for a third (or fourth,
fifth, or any number) of variables. To do so, you simply add them as
other predictors in the regression. One very helpful feature of
regression is that every variable is examined *controlling for / holding
constnant all the others*, so regression lets you see the *unique*
influence of each predictor. In other words, it lets you see what the
slope for each predictor would be, had participants had the same score
on all the other predictors. **Importnat: this only works if you thought
ahead and measured every variable you need to control for**. This is one
reason why I advocate for collecting data and not simply relying on
existing data repositories. You may very well need to control for a
variable that you don't have! However, if we think ahead, we can attempt
to measure any variable we might want to control for.

Let's try it, controlling for freedom:

    mod2 <- lm(Happiness ~ GDP + Freedom, data=dat)
    summary(mod2)

    ## 
    ## Call:
    ## lm(formula = Happiness ~ GDP + Freedom, data = dat)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -2.0725 -0.3726  0.0772  0.3475  1.3592 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)   2.5461     0.1502  16.955  < 2e-16 ***
    ## GDP           1.8736     0.1187  15.788  < 2e-16 ***
    ## Freedom       2.3557     0.3329   7.076 5.09e-11 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.5757 on 152 degrees of freedom
    ## Multiple R-squared:  0.7443, Adjusted R-squared:  0.741 
    ## F-statistic: 221.3 on 2 and 152 DF,  p-value: < 2.2e-16

First, our residual standard error (average misprediction) has decreased
some and our *R*<sup>2</sup> has increased. That tells us we have
"added" to our ability to predict happiness. As a side note, you can
test whether that increase is significant with 'anova(mod, mod2)\`:

    anova(mod, mod2)

    ## Analysis of Variance Table
    ## 
    ## Model 1: Happiness ~ GDP
    ## Model 2: Happiness ~ GDP + Freedom
    ##   Res.Df    RSS Df Sum of Sq     F    Pr(>F)    
    ## 1    153 66.983                                 
    ## 2    152 50.386  1    16.597 50.07 5.095e-11 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

This runs an *F* test with a significant result (*p*-value is `Pr(>F)`,
i.e., less than .05). This tells us that adding health/life expectancy
*significantly* (i.e., greater than chance) improved our prediction of
happiness. The *R*<sup>2</sup> value went up by 0.0864 (i.e., 8.6%, mod:
0.6579, mod2: 0.7443) so we have explained an additional 8.6% of
happiness.

Our new regression equation is:

> Happiness' = 2.55 + 1.87(GDP) + 2.36(Freedom).

The real action is in the slopes of the model, however. First, we can
see that the slope for GDP is still significant as is now the slope for
freedom (*p* values in far right column of the summary output). That
tells us we can be confident that, to some degree, they each predict
happiness *with the other controlled*. In other words, it is not the
case that the role of GDP is actually all due to freedom or vice versa.
Each has some *unique* or independent relationship with happiness.

However, we also do see that the slope for GDP went down. It was
originally 2.1841849 and is now 1.8735929, a difference of 0.3105921. If
we take that as a fraction of the original slope,
$\\frac{0.3105921}{2.184185} = .14$, we see that our slope has gone down
by 14%. In other words, 14% of what we had previously attributed to GDP
we can now actually attribute to freedom. This shows just how important
it is to control for things: we were seriously overestimating the
importance of GDP.

This illustrates the importance of having the correct statistical
controls in your study. We could easily imagine relationships that could
be reduced to zero when the necessary variables are controlled. If we
want to measure, as best we can, the impact of a predictor on the
outcome it is a **requirement** that we control for any possible hidden
drivers of your predictor that might *actually* be responsible for your
outcome. Thus, in this case, we might think of any *other* variables
that might lead to *both* higher GDP and greater national happiness.
Freedom is one, but we cannot draw a sound conclusions without seriously
thinking through and controlling for any others.

This brings up a *central* reason why organizations should conduct
research and not merely rely on existing data. In existing data sets,
you are tempted to draw conclusions on the basis of the data you have,
not the data you don't have. Yet, we see here, the data you don't have
could be necessary as a statistical control. Without that, how can we
really know that it is GDP, and not some other prior cause of GDP that
is driving up happiness? It is very important to seriously consider what
variables might be causing *both* your predictor and your outcome; if
these are not statistically controlled, then false conclusions are
possible.

#### Standardized Slopes

At this point, you might also be interested in which variable is the
stronger predictor. Unfortunately, the slopes are entirely dependent on
the units measured.

However, we can standardize them, putting them on the same scale so they
can be directly compared. This is done by taking each slope and
multiplying it by the ratio of the standard deviations:

$$b \\frac{SD\_x}{SD\_y}$$

Functionally, this changes nothing. **It only changes the units.** For
example, the slope for freedom was originally 2.3557. This means that
happiness is expected to increase 2.3557 *units* per *unit* of freedom.
If we multiply by standard deviations, however, the units become
standard deviations:

    1.8736 * sd(dat$GDP) / sd(dat$Happiness)

    ## [1] 0.696938

We immediately see that GDP has a much larger estimated effect than
freedom. At least in our sample data, the wealth of a region is more
important for its happiness than its freedom (the difference between
slopes can also be tested for significance, ideally with something
called bootstrapping, but we will not explore that now).

If you don't wish to calculate standardized slopes manually, the
`lm.beta` package has a function called `lm.beta()`. Instead of
requesting `summary(model)`, you request \`summary(lm.beta(model)) and a
column of standardized slopes is given to you.

    summary(lm.beta(mod2))

    ## 
    ## Call:
    ## lm(formula = Happiness ~ GDP + Freedom, data = dat)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -2.0725 -0.3726  0.0772  0.3475  1.3592 
    ## 
    ## Coefficients:
    ##             Estimate Standardized Std. Error t value Pr(>|t|)    
    ## (Intercept)   2.5461       0.0000     0.1502  16.955  < 2e-16 ***
    ## GDP           1.8736       0.6969     0.1187  15.788  < 2e-16 ***
    ## Freedom       2.3557       0.3124     0.3329   7.076 5.09e-11 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.5757 on 152 degrees of freedom
    ## Multiple R-squared:  0.7443, Adjusted R-squared:  0.741 
    ## F-statistic: 221.3 on 2 and 152 DF,  p-value: < 2.2e-16

The standardized slopes are added just to the right of the original
slopes.

We now have the tools we need to begin to understand our data. Using
multiple regression, I can examine the relationships between any
variables, controlling for any number of others. If I want to be able to
assess what variables are the most important predictors, standardized
slopes offer an easy solution (they are often comparable to correlations
as well, providing a rough mental guide for interpreting them).

#### Assumptions of Regression

There are a few important assumptions and caveats you should know. Seven
key assumptions are:

Our assumptions are:

1.  Linear model is appropriate
2.  Homoskedasticity
3.  Observations are independent
4.  Our residuals must be normally distributed
5.  Our residuals should have a mean of zero
6.  The model is correctly specified
7.  Multicollinearity is not a problem

**Assumption 1**, is a linear model appropriate? Well, do we have good
reason to suspect a non-linear relationship between GDP and happiness?
In this case, I have no theoretical or evidence based reason to think
otherwise. However, obviously, a linear model (i.e., a "straight line"
regression) would not make sense if the data were not a straight line.

Note that there are a few cases where this *IS* violated.

1.  If your y-variable has only two levels (e.g., "purchased product,
    did not purchase product"), then a linear model is not appropriate
    (we use something called *logistic regression*). There are similar
    extensions for variables with several categories.

2.  Another common case is if your y-variable is a "count" of something
    (e.g., how many pairs of shoes do people have?). In that case,
    negative numbers are impossible, and any slopes will need to "bend"
    as they approach the x-axis so they don't cross it. There are
    "count" models for that. We won't learn them, but you could if you
    needed to.

3.  Math allows for any shape of line; for similar reasons, we can make
    a regression line into any shape (not covered here). A word of
    caution: the more complex a model, the more likely it is to be a
    "false positive" finding. Beware making models too complex on the
    basis of your data. Usually, simpler models are more correct unless
    you have good reason to make it non-linear.

**Assumption 2**, homoskedasicity. Briefly, this means that the
residuals are the same all the way up and down the line. This is
violated if residuals are really small at one end (line fits great) and
really big at the other (line fits awful). Without going into detail,
the *p*-values for the slopes use the residuals in the equations. If the
residuals vary systematically throughout the graph, that's a problem.
Look at the residuals plot above, however, and we see they seem roughly
uniform up and down the line. We're fine.

**Assumption 3**, our observations are independent. That is, our data
points didn't influence each other. This would be a problem, for
example, if our data included the same regions over a multi-year period.
The 2016 happiness score and the 2017 happiness score for a region will
be similar because it's the same region. This influences the residuals,
and in turn, the *p*-values in a problematic way. *Violating this
assumption tends to make your findings **more** significant* so it's
good to account for any kind of "nesting" substructure.

Having multiple data points on the same observation is one common way
you can violate this assumption, but other kinds of nesting might as
well. For example, data may be nested within stores in a way we might
want to account for. In such a case, a *linear mixed model* is often
better and simply extends the regression somewhat to account for the
nesting.

**Assumption 4 + 5,** our residuals must be normally distributed and
with a mean of zero. We can briefly check this with a histogram:

    hist(mod2$residuals)

![](Mod3_Lab2_-_Regression_files/figure-markdown_strict/unnamed-chunk-23-1.png)

It's roughly normal. We can check the mean:

    #mean is almost zero
    mean(mod$residuals)

    ## [1] -4.727191e-17

It has some skew, but it's not terrible.

**The normality of residuals tends to be violated if the outcome
(y-variable) is skewed**. There are other kinds of regressions for
highly skewed outcome variables; there are also transformations that can
be done to the outcome variable to make it more normal. We explored such
transformations in the previous lab, but such a transformation is not
needed here.

**Assumption 6** states that we have included all the causes of our
outcome in the model. Although impossible in practice, we can reasonably
think through what variables we need to control for when running our
regression.

Now you see something important: to do regression analysis well, you
need to understand the subject area. You need to know what to control
for *before* the data are collected.

**Assumption 7** states that multicollinearity is not a problem.

Briefly, when you control for a variable (e.g., looking at GDP and
happiness, controlling for freedom), you can cause a problem called
multicollinearity.

Consider the following situation: you want to assess the role of annual
salary on employee engagement, controlling for monthly salary.
Hopefully, you think that's a crazy idea because the predictor and
control variables are redundant. It's completely nonsensical. What would
it even mean to examine the role of your annual salary, holding your
monthly salary constant? There would be "nothing left" to annual salary
after we statistically hold monthly salary constant.

Similarly, there are a lot of highly overlapping variables in research
contexts. Imagine a survey asking customers how 'angry' they are and how
'upset' they are after receiving poor service. Angry people are upset.
Upset people feel angry. They're very similar. Conceptually, if you
tried to examine anger with 'upset' controlled, you are going to remove
a lot of the meaning in the anger variable, and vice versa. Will there
be enough "left" to make a reasonable regression?

When predictors overlap (correlate) too much among each other, they
become useless. In general, it's good to avoid having high correlations
among predictors, *r* = .80, *r* = .90, etc.). The slopes can act oddly
in such cases, even reversing directions.
