Test Theory: Confirmatory Factor Analyses
================
Philipp Masur
2021-11

-   [Introduction](#introduction)
    -   [Classical Test Theory](#classical-test-theory)
    -   [Assumptions](#assumptions)
    -   [Reflective measurement model](#reflective-measurement-model)
-   [Preparation](#preparation)
    -   [Getting some data](#getting-some-data)
    -   [Recoding](#recoding)
    -   [Psychometric Properties](#psychometric-properties)
    -   [Multivariate normal distribution
        check](#multivariate-normal-distribution-check)
-   [Confirmatory factor analysis](#confirmatory-factor-analysis)
    -   [Estimating the model](#estimating-the-model)
    -   [Investigating possibilites to increase
        fit](#investigating-possibilites-to-increase-fit)
-   [Reliability](#reliability)
-   [Why bother?](#why-bother)
-   [Where to go next?](#where-to-go-next)

# Introduction

## Classical Test Theory

In the social sciences, we are often interested in somewhat ‘abstract’
concepts (e.g., emotions, attitudes, literacy, personality,…). These
concept cannot be measured *directly*, but have to be assessed
*indirectly* using observable indicators (e.g., items in a
questionnaire). We therefore create several items that are meant to
provide information about the underlying trait.

Test theory explains the relationships between the “latent variable”
(e.g., a personality trait such as “extraversion”) and the responses to
several items (e.g., “I make friends easily”, “I know how to captivate
people”…). It defines the statistical relation between a measurement and
the actual characteristic of interest.

## Assumptions

A basic assumption of classical test theory is thus that the trait
explains response patterns in items. To investigate this relationship
further, we need to differentiate the following concepts:

<br>

*Table 1: Important concepts in classical test theory*

| Name                | Definition                                                                                                                                                                 |
|---------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Latent variables    | Not directly observable concepts - later also called ‘factors’ - that we are interested in estimating (e.g., emotions, attitudes, personality, literacy, concerns…)        |
| Manifest indicators | Measureable aspects that should be influenced by the latent variable (e.g., items in a questionnaire, but we can also think of other indicators)                           |
| True score          | The share of the variance in the measurement of a manifest indicator that is directly linked to the latent variable; what we want to estimate to the best of our abilities |
| Measurement error   | Share of the measurement variance that is not linked to the latent variable (includes item-specific variance, systematic errors, and random errors)                        |

<br>

In classical test theory, we decompose the variance of each measurement
of an manifest indicator: Every observable measurement *Y* (e.g., an
item) is composed of variance explained by the latent variable (true
score: *τ*<sub>*a*</sub>) ), variance explained by the specific manifest
indicator (e.g., the item: *τ*<sub>*b*</sub>)), and the measurement
error (*ϵ*):

*Y*<sub>*i*</sub> = *τ*<sub>*a*, *i*</sub> + *τ*<sub>*b*, *i*</sub> + *ϵ*<sub>*i*</sub>

The measurement error is thereby independent of the true score and
varies randomly across persons and measurement occasions:

*C**o**r*(*τ*<sub>*i*</sub>, *ϵ*<sub>*i*</sub>) = 0

Measurement errors can be divided into two components: *random error*
and *systematic error.* Random errors are errors in measurement that
lead to measurable values being inconsistent when repeated measurements
of a constant attribute or quantity are taken. Systematic errors are
errors that are not determined by chance but are introduced by an
inaccuracy (involving either the observation or measurement process).
Systematic errors can be imperfect calibration of the instrument or
interference of the environment.

## Reflective measurement model

When we aim to measure abstract concept (e.g., neuroticism), we estimate
a **reflective** measurement model. Such models, in which a latent
characteristics is estimated based on multiple manifest indicators, can
be represented with the following path model (panel 1):

<br>

![](https://ars.els-cdn.com/content/image/1-s2.0-S0148296308000118-gr1.jpg)

*Image source: Diamantopoulos et al., 2008*

<br>

We specify a latent variable (*η*) that explains people’s responses in
several items (*x*<sub>*i*</sub>) by taking the items-specific error
(*e**p**s**i**l**o**n*<sub>*i*</sub>; composed of item-specific variance
and the measurement error) into account. In most cases, not all items
are equally well explained by the latent factor. For some, the true
score will be lower than for others. This is denoted by the factor
loadings of each item (*λ*<sub>*i*</sub>).

**Note:** There are also formative concepts in which the combination of
individual indicators make up the formative factor (e.g., the value of a
car is determined by its age, condition, size, make, etc.)

In this tutorial, we will estimate a reflective measurement model using
a confirmatory factor analysis (CFA) in R. A CFA can be best estimated
using the package `lavaan`, which I will introduce later. As we will
engage in some data wrangling beforehand, we will load the package
collection `tidyverse`. Because we will also assess items individually,
we also load the package `psych`, which provides nice tools to assess
psychometric properties of variables.

``` r
library(tidyverse)
library(psych)
```

# Preparation

## Getting some data

For this tutorial, we will assess a classic reflective measurement model
of psychology: The Big Five Personality Model as assessed in the
International Personality Item Pool (<https://ipip.ori.org>).
Conveniently, it is included in the `psych` package and we can load it
by simply calling `bfi`. Let’s quickly open the respective documentation
to assess the item formulations.

``` r
d <- bfi %>% as_tibble
head(d)
?bfi
```

As we can see, the scale consists of 25 items. Based on the Big Five
Model of Personality, we can assume that these items reflect five
distinct dimensions:

-   Agreeableness (e.g., “I inquire about others’ well-being”)
-   Conscientiousness (e.g., “I continue until everything is perfect”)
-   Extraversion (e.g., “I don’t talk a lot.”)
-   Neuroticism (e.g., “I get irritated easily.”)
-   Openness (e.g., “I am full of ideas.”)

All items are measure on 6-point scale from 1 = Very Inaccurate to 6 =
Very Accurate.

If we look at the item formulations, we can see that 8 items are reverse
coded. We should hence inverse them before continuing with out analyses.
We can use a simply `mutate()` command to do this quickly.

## Recoding

``` r
bfi_items <- d %>%
  select(A1:O5) %>%
  mutate(A1 = (A1-7)*-1,  # recoding (inversed)
         C4 = (C4-7)*-1,
         C5 = (C5-7)*-1,
         E1 = (E1-7)*-1,
         E2 = (E2-7)*-1,
         O1 = (O1-7)*-1,
         O3 = (O3-7)*-1,
         O4 = (O4-7)*-1)
```

## Psychometric Properties

A first step in any confirmatory factor analyses should consist of
assessing all items’ psychometric properties and whether they are
normally distributed. The former, we can do by using the `describe()`
function from the `psych` package. This function will provide the mean,
standard deviation, min and max as well as estimates for the skewness
and kurtosis.

``` r
# Descriptive Analysis
bfi_items %>%
  describe
```

Based on the columns skewness and kurtosis, all items seems to be
reasonably normally distributed. But let’s check this also visually. We

``` r
# Checking normal distributions
bfi_items %>%
  pivot_longer(A1:O5, names_to = "key", values_to = "value") %>%
  ggplot(aes(x = value)) +
  geom_histogram(bins = 6, 
                 fill = "lightblue",
                 color = "white") +
  facet_wrap(~key) +
  theme_minimal() +
  labs(x = "Values (1 = Do not agree at all; 6 = Fully agree)",
       y = "Number of responses")
```

We can see that most values are rather normally distributed, but e.g.,
item A4 or O4 are heavily skewed. This may not be a problem, but this
information might be useful later, if we try to improve the model.

## Multivariate normal distribution check

For any confirmatory factor analyses, we should additionally check
whether the assumption of multivariate normal distribution is met. We
can do so with the the function `mardia()` from the `psych` package
which computes Mardia’s test of multivariate skewness and kurtosis.

``` r
bfi_items %>%
  mardia(plot = FALSE) # We should use MLR instead of ML
```

Both test are significant, suggesting that this assumption is violated.
Again, this is not problematic per se, but we should use a robust
estimator when fitting the confirmatory model.

# Confirmatory factor analysis

## Estimating the model

Now we can estimate the assumed model (i.e., assuming the five
dimensions). For this, we will use the package `lavaan` (for more
information and code example, see this documention:
<https://lavaan.ugent.be/>), which provides a convenient syntax for
fitting structure equation models in general (measurement models are
technically structural equation models!).

The lavaan syntax is straightforward. First, we define each latent
factor in one string. In this example, the model syntax will contain
five ‘latent variable definitions’. Each latent variable formula has the
following format:

`latent variable =~ indicator1 + indicator2 + ... + indicator_n`

We call these expressions latent variable definitions because they
define how the latent variables are ‘manifested by’ a set of observed
(or manifest) variables, often called ‘indicators’. Note that the
special `=~` operator in the middle consists of a sign (`=`) character
and a tilde (`~`) character next to each other.

The reason why this model syntax is so short, is that behind the scenes,
the `cfa()` function, which wraps around this model in the next step,
will take care of several things. First, by default, the factor loading
of the first indicator of a latent variable is fixed to 1, thereby
fixing the scale of the latent variable. Second, residual variances are
added automatically. And third, all latent variables are correlated by
default.

We can generally choose between different estimators (see this
[page](https://lavaan.ugent.be/tutorial/est.html) for more information
on different estimators), but as our Mardia test was significant, we use
a robust version of the maximum likelihood estimation (MLR). The first
argument is the user-specified model. The second argument is the
estimator and the third argument the dataset that contains the observed
variables.

``` r
library(lavaan)

# Defining model
cfa_model <- "
   Agreeableness =~     A1 + A2 + A3 + A4 + A5
   Conscientiousness =~ C1 + C2 + C3 + C4 + C5
   Extraversion =~      E1 + E2 + E3 + E4 + E5
   Neuroticism =~       N1 + N2 + N3 + N4 + N5
   Openness =~          O1 + O2 + O3 + O4 + O5
"

# Estimate the model
fit.cfa <- cfa(cfa_model, 
               estimator = "MLR", 
               data = bfi_items)
```

In a first step, we should assess how well our model fits the data.

``` r
fitMeasures(fit.cfa, c("chisq.scaled", "df.scaled", "pvalue.scaled", 
                       "cfi.robust", "tli.robust", "rmsea.robust"))
```

In our example, the model did not fit the data too well as the
*χ*<sup>2</sup> value is comparatively high and significant and CFI and
TLI are below common thresholds (e.g., &lt; .95).

Once the model has been fitted, we can also use the `summary()`
function, which provides a nice summary of the fitted model:

``` r
summary(fit.cfa, fit = T, std = T)
```

The output contains three parts:

-   The header: Information about lavaan, the optimization method, the
    number of free parameters and number of observations used in the
    analysis (in this case n = 2436)
-   The fit section: Includes various fit indices to assess model fit
-   Parameter Estimates: The last secion contains all parameters that
    were fitted (including the factor loadings, variances, thresholds…)

In our example, the factor loadings range from .233 to .825. Thus, some
items (those with factor loadings below e.g., &lt; .5) have a
comparatively low true score, i.e., the assumed latent dimensions does
not explain the responses to this items very well.

## Investigating possibilites to increase fit

We thus have to conclude that our original model does not fit the data
well and that there is still some misfit that may stem from bad items or
a faulty assumption about the dimensionality.

We can use the function `modindices()` to better understand reasons for
misfit. We directly arrange the output according to the mi column and
round the result to 2 decimals.

``` r
# Extracting modifiction indices
modindices(fit.cfa) %>%
  arrange(-mi) %>%
  mutate_if(is.numeric, round, 2)
```

The first line shows e.g., that the items N1 and N2 share a lot of
common variance and should be subsumed under their own latent factor.
The second line further tells us that the item N4 (an item that should
measure neuroticism) also laods onto the factor extraversion. It thus
has unwanted cross-loadings. Looking at these modification indices as
well as the factor loadings AND assessing the items formulations, we may
come to the conclusion that we could signicantly improve model fit by
removing some items. So we specify a second model where some “bad” items
are removed.

``` r
# Removing some items
cfa_model2 <- "
   Agreeableness =~          A2 + A3 + A4 + A5
   Conscientiousness =~ C1 + C2 + C3 + C4 + C5
   Extraversion =~      E1 + E2 + E3 + E4 + E5
   Neuroticism =~       N1 +      N3 + N4 + N5
   Openness =~          O1 + O2 + O3 +      O5
"
fit.cfa2 <- cfa(cfa_model2, 
               estimator = "MLR", 
               data = bfi_items)

# Output
summary(fit.cfa2, fit = T, std = T)
```

The model fit slightly increased (yet still does not reach satisfactory
levels) and all factor loadings are now acceptable.

Overall, this tells us that the theoretical assumption that this item
pools reflects five personality characteristics is somewhat right, but
there is still problems in the data. Usually, we would try to further
optimize the model or even evaluate a new item pool at this point. For
this tutorial, we will now move on to look at the reliability - another
aspect that we should look at when evaluating a scale.

# Reliability

The additive variance decomposition that we used to estimate our latent
measurement model is also the basis for one of the most important
estimates in classical test theory: the reliability. It can be defined
as the true score share of the measured variance:

*R**e**l*(*Y*<sub>*i*</sub>)=

*V**a**r*(*τ*<sub>*i*</sub>)/*V**a**r*(*Y*<sub>*i*</sub>)=

*V**a**r*(*τ*<sub>*i*</sub>)/(*V**a**r*(*τ*<sub>*i*</sub>) + *V**a**r*(*ϵ*<sub>*i*</sub>))

If we have a multi-item measure, the covariance between two items of the
same scale represents the true score:

*V**a**r*(*τ*) = *C**o**v*(*Y*<sub>1</sub>, *Y*<sub>2</sub>)

In practice, several different scores have been proposed to estimate a
scale’s reliability. Perhaps the most common one is Cronbach’s *α*,
which however assume *τ*-equivalency (i.e., that all factor loadings are
the same), which is mostly not the case! Alternatively, we can have a
look at McDonald’s *ω* which is also known as “composite” reliability.
The package `semTools` provides convenient functions that extend the
functionality of `lavaan`. We can use the simple function
`reliability()` to compute all relevant reliability estimates.

``` r
semTools::reliability(fit.cfa2)
```

The resulting table includes Cronbach’s *α* (alpha), Bollen’s *ω*
(omega), Bentler’s *ω* (omega2), and McDonald’s *ω* (omega3) which is
also know as hierarchical omega. If the model is good, all of these
scores will not differ much. If the model contains items with low factor
loadings, the omega values will differ considerably from the alpha. All
of them range from 0 to 1 with values closer to 1 representing good
reliability. Although common thresholds (e.g., .7 = acceptable, .8 =
good, and .9 = excellent), what can be considered a good reliability
really depends on what you are measuring and the consequences of
potential measurement error in your model.

# Why bother?

We have seen that we can use confirmatory factor analyses to assess a
scales’ dimensionality, factor validity, and reliability. This is
important in its own right as it will tell us how appropriate it is to
combine several items into a mean or sum indice. However, there is more
to accurately measuring latent variables:

> “Generally, ignoring measurement error leads to inconsistent
> estimators and to inaccurate assessments of the relation between the
> underlying latent variables.” (Bollen, 1989)

There is always measurement error that we should not ignore.
Relationships between variables with measurement errors (e.g., mean
indices) will always be biased (usually downward). As we are often
interested in rather small relationships (e.g., r = .10), we should try
to reduce the measurement error. For example, instead of correlating
mean scores, we can correlate the estimated true scores in a structural
equation model (SEM). A SEM is simply two measurement models linked by a
regressions or correlation (see Figure 2):

![](https://upload.wikimedia.org/wikipedia/commons/thumb/b/b1/Example_Structural_equation_model.svg/1200px-Example_Structural_equation_model.svg.png)

Below, we first compute the mean indices for all dimensions. We then
investigate the relationship between Agreeableness and Extraction based
on the mean scores as well as based on the structural equation model
that we estimated earlier (basically the inter-factor correlation in the
CFA model). As one can see, the correlation between the latent variables
is higher than the correlation between the mean scores.

``` r
# Creating Sum Scores
bfi_items <- bfi_items %>%
  mutate(A_i = (A2 + A3 + A4 + A5)/4,
         C_i = (C1 + C2 + C3 + C4 + C5)/5,
         E_i = (E1 + E2 + E3 + E4 + E5)/5,
         O_i = (O1 + O2 + O3 + O5)/4,
         N_i = (N1 + N3 + N4 + N5)/4)

# Correlation between Agreeableness and Extraversion based on the mean scores
corr1 <- cor(bfi_items$A_i, bfi_items$E_i, use = "complete.obs")

# Correlation between true scores extracted from the CFA
corr2 <- parameterEstimates(fit.cfa2, standardized = T) %>% 
  filter(lhs == "Agreeableness", rhs == "Extraversion") %>% 
  select(std.all) %>% 
  as.numeric

# Comparison
bind_rows(sumscore = corr1, 
          truescore = corr2)
```

# Where to go next?

Test theory and measurement theory is a wide field. But to get started
check out the following books and articles:

-   Beaujean, A. A. (2014). Latent Variable Modeling Using R: A
    Step-by-Step Guide. Routledge.

-   Bollen, K. A. (1989). Structural Equations with Latent Variables.
    John Wiley and Sons, Inc.

-   Diamantopoulos, A., Riefler, P., & Roth, K. P. (2008). Advancing
    formative measurement models. Journal of Business Research, 61(12),
    1203–1218. <https://doi.org/10.1016/j.jbusres.2008.01.009>

-   DiStefano C, Zhu, M. & Mîndrila, D. (2009). Understanding and Using
    Factor Scores: Considerations for the Applied Researcher. Practical
    Assessment, Research & Evaluation, 14(20), 1-11.

-   Maydeu-Olivares, A. & McArdle, J. J. (2005). Contemporary
    Psychometrics. Lawrence Erlbaum Associates.
