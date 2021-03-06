---
title       : Bootstrapping
subtitle    : 
author      : Jeffrey Leek
job         : Johns Hopkins Bloomberg School of Public Health
logo        : bloomberg_shield.png
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow   # 
url:
  lib: ../../libraries
  assets: ../../assets
widgets     : [mathjax]            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
---






## Pro tip

Science is run by humans. Science is also not one thing, but a lot of things. Keeping this in mind can save you a lot of headache. 


---

## Papers of the day

[R Markdown: Integrating A Reproducible Analysis Tool into Introductory Statistics](http://arxiv.org/abs/1402.1894)

[PloS Data Sharing Policy](http://blogs.plos.org/everyone/2014/02/24/plos-new-data-policy-public-access-data/)

* [9 questions about the new PLoS clarification](http://scientopia.org/blogs/neuropolarbear/2014/02/28/9-questions-plos-clarification/)
* [PLoS Clarification confuses me more](http://rxnm.wordpress.com/2014/03/03/plos-clarification-confuses-me-more/)
* [I own my data until I don't](http://smallpondscience.com/2014/03/03/i-own-my-data-until-i-dont/)


---

## The bootstrap - the 30,000 foot view

Bootstrapping is a computational procedure for:

* Calculating standard errors
* Forming confidence intervals
* Performing hypothesis tests
* Improving predictors (called bagging)

---

## A bit of history

The basic idea behind bootstrapping is to treat the sample of data you observe (or some appropriate function of the data) as
if it was the super-population you were sampling from. Then you sample from the observed set of values to try to approximate
the sampling variation in the whole population. 

The idea of the bootstrap was originally proposed by [Efron in 1979](http://projecteuclid.org/DPubS?service=UI&version=1.0&verb=Display&handle=euclid.aos/1176344552) 

Related ideas are very old by the standards of statistics (Quenouille, 1956 Notes on bias in estimation. Tukey, 1958 Bias and confidence in not-quite large samples) 


---

## The Frequentist "Central Dogma"

<img class="center" src="../../assets/img/frequentist.jpg" height=400>



---

## The Bootstrap "Central Dogma"

<img class="center" src="../../assets/img/bootstrap.jpg" height=400>

---

## The plug in principal and ECDF 

The __plug-in__ principle states that if we have a parameter $\theta = t(F)$, then we estimate the parameter
by applying the same functional to an estimate of the distribution function $\hat{\theta} = t(\hat{F}_n)$. Although
other estimates can also be used\vsp
The default $\hat{F}_n = {\mathbb F}_n$ is the empirical distribution $$ {\mathbb F}_n(y) = \frac{1}{n} \sum_{i=1}^n 1(Y_i \leq y)$$
A sample $Y_i^'$ from ${\mathbb F}_n$ has the property that $Y_i^' = Y_j$ with probability $1/n$ for $1 \leq j \leq n$\vsp
Why ${\mathbb F}_n$?

* Glivenko-Cantelli ($||{\mathbb F}_n-F||_\infty  = \sup_{y\in \mathbb{R}}|{\mathbb F}_n(y) - F(y)| \rightarrow_{a.s.} 0$)
* ${\mathbb F}_n$ is a maximum likelihood estimate (see e.g. http://www.cs.huji.ac.il/~shashua/papers/class3-ML-MaxEnt.pdf)
* It is reasonable 

---

## An example of the plug in principle

Suppose $Y_i > 0$ are _i.i.d._ from $F$. We might be interested in:
$$ \theta = {\rm E}_F \log(Y)$$
- the log of the geometric mean of F. A plug in estimate of $\theta$ is:
$$\hat{\theta} = {\rm E}_{{\mathbb F}_n} \log(Y^')$$
which is actually available in closed form:
$$\hat{\theta} = \frac{1}{n} \sum_{i=1}^n \log(Y_i)$$
We never specified $F$ parametrically, hence this is a model-agnostic estimate and robustness can be expected, in large samples. (Also note $e^{\hat{\theta}}$ is the sample geometric
mean)

---

## Another example

Let $Y$ be a binary variable and suppose
$$\theta(F) = Pr(Y = 1) = {\rm e}_F[1(Y_i = 1)]$$
We can find the estimate of $\theta(F)$ using the plug-in principle:
$$\hat{\theta} = {\rm e}_{{\mathbb F}_n}[1(Y_i^' = 1)] = Ar{Y}$$
Suppose we wanted an estimate for the variance $${\rm Var}(\hat{\theta}) = {\rm Var}(Ar{Y}) = {\rm Var}(Y_i)/n$$

We could use the plug in estimator
$${\rm Var}_{{\mathbb F}_n}(Y_i)/n = {\rm e}_{{\mathbb F}_n}[(Y_i^' - Ar{Y})^2]/n = \frac{Ar{Y}(1-Ar{Y})}{n}$$


---

## The algebra for ${\rm Var}_{{\mathbb F}_n}(Y_i)$

$${\rm Var}_{{\mathbb F}_n}(Y_i)/n = {\rm E}_{{\mathbb F}_n}[(Y_i^' - Ar{Y})^2]/n$$
$$= \sum_{j=1}^n \frac{1}{n}(Y_j - Ar{Y})^2$$
$$=\frac{1}{n}\left[(1-Ar{Y})^2 \sum_{j=1}^n Y_j + Ar{Y}^2 \sum_{j=1}^n(1- Y_j)\right]$$
$$=\frac{1}{n} \left[\sum_{j=1}^n Y_j - 2Ar{Y} \sum_{j=1}^n Y_j + nAr{Y}^2\right]$$
$$= Ar{Y}^2 - Ar{Y} = Ar{Y}(1-Ar{Y})$$

When evaluating ${\rm E}_{{\mathbb F}_n}$, $Ar{Y}$ is a ``parameter'' and treated as fixed. 

---

## Bootstrap - usually no closed form

Usually, no closed form evaluation will exist (we sort of "got lucky" in the previous examples). How did we get lucky? The plug-in estimate ended up being an expectation of a single random variable $Y_i^'$.

* What if we were unlucky and the plug in estimate was a function of all of the values $Y_1^{'},\ldots,Y_n^{'}$?

* For example, suppose we wanted to estimate the variance of the sample median $\hat{\theta} = {\mathbb F}_n^{-1}(1/2)$?

* In this case, the variance ${\rm Var}_{{\mathbb F}_n}(\hat{\theta})$ is an expectation of a function of $Y_1^',\ldots,Y_n^'$.

---

## Too many bootstrap samples

It isn't clear there is a pretty formula for the variance of the median, but if we let $X_j$ denote the number of times $Y_j$ occurs in a bootstrap sample, then: $(X_1,\ldots,X_n) \sim Mult(n; 1/n,\ldots, 1/n)$ so:
$$ \sum_{Y^'\in \mathcal{S}} \left\{{\mathbb F}_n^{'-1}(1/2) - {\rm E}_{{\mathbb F}_n}[{\mathbb F}_n^{-1}(1/2)]\right\}^2 \frac{n!}{\prod_{i=1}^n x_i!} (1/n)^n$$
where ${\mathbb F}_n^{'-1}(1/2)$ is the sample median for each bootstrap sample and $\mathcal{S}$ is the set of all unique
bootstrap samples from $Y_1,\ldots,Y_n$. 

* There are ${2n -1}\choose{n}$ unique bootstrap samples.
* For $n = 10$ there are 92,378 unique values. For $n=25$ there are 63.2 trillion or so. 


---

## Monte carlo bootstrap

Most of the time, you'll use the bootstrap for parameters where a closed form doesn't necessarily exist. 
Instead we use a __Monte Carlo__ approach. 
For the variance of the sample median you would:

* Select $B$ independent bootstrap samples $Y^{'b}$ from ${\mathbb F}_n$. 
* Recalculate the statistic for each sample $\hat{\theta}^{'b} = {\mathbb F}_n^{'b-1}(1/2)$
* Approximate $\sum_{Y^'\in \mathcal{S}} \left\{{\mathbb F}_n^{'-1}(1/2) - {\rm E}_{{\mathbb F}_n}[{\mathbb F}_n^{-1}(1/2)]\right\}^2 \frac{n!}{\prod_{i=1}^n x_i!} (1/n)^n$
by $$\frac{1}{B} \sum_{b=1}^B \left\{{\mathbb F}_n^{'b-1}(1/2) - Ar{{\mathbb F}}_n^{'-1}(1/2)\right\}^2$$ where $Ar{{\mathbb F}}_n^{'-1}(1/2) = \frac{1}{B}\sum_{i=1}^b {\mathbb F}_n^{'b-1}(1/2)$

As $b \rightarrow \infty$ you get closer and closer to the exact or ``ideal bootstrap''. 


---

## Bootstrap - Monte Carlo standard error

The general form for calculating bootstrap standard errors is similar:

* Select $B$ independent bootstrap samples $Y^{'b}$ from ${\mathbb F}_n$. 
* Recalculate the statistic for each sample $\hat{\theta}^{'b}$
* Approximate ${\rm E}_{{\mathbb F}_n}[(\hat{\theta'} - \hat{\theta})^2]$
by $$\frac{1}{B} \sum_{b=1}^B (\hat{\theta}^{'b} - Ar{\hat{\theta}}^{'})$$
where $Ar{\hat{\theta}}^' = \frac{1}{B}\sum_{i=1}^b \hat{\theta}^{'b}$


--- 

## Bootstrap - Monte Carlo coverage estimates

For a confidence interval $a(Y), b(Y)$ based on a sample of size n, we may want to estimate coverage $$E_F 1_{[a < \theta < b]} = Pr(a < \theta < b | F)$$. The bootstrap estimate is
$$E_{{\mathbb F}_n} 1_{[a < \hat{\theta} < b]} = Pr(a < \hat{\theta} < b | {\mathbb F}_n)$$
The Monte-Carlo version of this calculation is: 

* Select $B$ independent bootstrap samples $Y^{'b}$ from ${\mathbb F}_n$
* Approximate the coverage by:
$$ \hat{{\rm Coverage}}_{BOOT} = \frac{1}{B} \sum_{b=1}^B 1_{[{\rm E}ll(Y^{'b}) < \hat{\theta} < u(Y^{'b})]}$$


Note this means you need two steps of calculation

---

## Bootstrap Monte Carlo coverage estimates

Think about these in very frequentist terms:

* Calculate 2.5\%, 97.5\% quantiles from many $\hat{\theta}_n(Y^')$. This bootstrap
percentile interval does contain $\hat{\theta}_n$ in 95\% of the replicates under ${\mathbb F}_n$ and hence for large $n$ should contain the true $\theta$ in $\approx 95\%$ replicates under $F$. 
* Calculate bootstrap mean, variance of $\hat{\theta}_n$. For large $n$ asymptotic
normality means ${\rm E}_{{\mathbb F}_n}\hat{\theta}_n \pm 1.96 \times \sqrt{{\rm Var}_{{\mathbb F}_n}}(\hat{\theta})_n$ will contain $\theta_n({\mathbb F}_n)$ in
95\% of replicates from ${\mathbb F}_n$; hence approximately 95\% under F.
* Moments are more stable than quantiles; smaller $B$ may be okay. 
* Draw a histogram of your $\hat{\theta}(Y^')$ to informally check if asymptotic normality under ${\mathbb F}_n$ is reasonable. (If it's not, the quantile method is not guaranteed either!)

---

## Bootstrap: important caveat

We resampled $Y$ __and__ $X$; whole rows of the dataset

* This is effectively treating $Y$ and $X$ as random quantities, which may not
be how the experiment was set up. 
* Random $X$ tends to lead to wider confidence intervals; inference is (typically)
__conservative__, if $X$ was actually fixed
* Remember the empirical average in the default sandwich also treats $X$ as random,
in the approximation for $\hat{A}$, $\hat{B}$; default bootstrapping (a.k.a. resampling cases) is in general very similar
* In practice other choices often matter more (e.g. which $\hat{F}$). 

---

## Bootstrap and sandwich (adapted from N. Breslow)

The score contribution for the $i^{th}$ subject for $\beta_j$ is $U_{ij} = \frac{\partial {\rm E}ll_i}{\partial \beta_j}$ so
$\sum_{i} \frac{\partial {\rm E}ll_i}{\partial \beta} = U^T_\beta \cdot 1$ where $1$ is an $n \times 1$ vector of ones. Note
that $\hat{\beta}$ satisfies $U^T_\beta \cdot 1 = 0$. 


Let $\frac{\partial^2 {\rm E}ll }{\partial \beta \partial \beta^T} = -A$
The score evaluated at $\beta = \hat{\beta}$ in the bootstrap sample is:
$$U^T_{\hat{\beta}} \cdot W \neq 0$$ 
where $W$ is a vector of random weights. 

The one step approximation to $\hat{\beta}_W$ the ML estimate in the bootstrap sample, starting from $\hat{\beta}$ is given by
$$(\hat{\beta}_W^' - \hat{\beta}) \approx (\hat{\beta}^'_{one} - \hat{\beta}) \approx \hat{A}^{-1} U^T_{\hat{\beta}} W$$

Now ${\rm E}[W] = 1$ and ${\rm Var}(W) = I - \frac{1}{n} 1 \cdot 1^T$

---

## Bootstrap and sandwich (continued)

Thus the approximate bootstrap mean is:
$$ {\rm E}_W(\hat{\beta}^'_{one} - \hat{\beta}) \approx \hat{A}^{-1} U^T_{\beta} {\rm E}(W) = 0$$
and the approximate variance is:

$${\rm E}_W(\hat{\beta}^'_{one} - \hat{\beta})(\hat{\beta}^'_{one} - \hat{\beta})^T$$

$\approx  \hat{A}^{-1} U_{\hat{\beta}}^T {\rm Var}{W} U_{\hat{\beta}} \hat{A}^{-1}$$

$$= \hat{A}^{-1} \hat{B} \hat{A}^{-1}$$

where $\hat{B} = \sum_{i=1}^n \hat{U}_i \hat{U}_i^T$


---

## Bootstrap: parametric version

* Suppose we believe the assumed model $F(\theta)$

* Usually the likelihood-based approaches give adequate intervals for parameters. We may
be interested in analytically difficult functions of $F$, e.g. medians, ranks, .. for example, taking derivatives may be awkward. 

* The parametric bootstrap is a useful approach; we take many replicates $Y^'$ under $\hat{F}_n = F(\hat{\theta})$

* Then look at the long-run, frequentist behavior of $g(Y^')$. Typically, $g()$ is some form of estimator. 

---

## Regression models: resampling residuals

* We discussed resampling cases, the default bootstrap method.
* In linear regression we can consider resampling residuals. 
* Consider the model $$ Y_i = \mu(x_i,\beta) + {\rm E}psilon_i$$ where the residuals ${\rm E}psilon_i$ are such that ${\rm E}[{\rm E}psilon_i] = 0$ for $i = 1,\ldots,n$ and are independent.
* Assume $F$is the distribution of $Y$ only. We assume $x_i$ is fixed, for our data and all replications. 
* The bootstrap datasets are formed as $$Y_i^{'b} = \mu(x_i,\hat{\beta}) + {\rm E}psilon_i^{'b}$$
where a number of options are available for sampling ${\rm E}psilon_i^{'(b)}$, $b=1,\ldots,B$, $i=1,\ldots,n$.  

---

## Parametric versus non-parametric

* The non-parametric version samples ${\rm E}psilon_i^{'(b)}$ with replacement from:
$$e_i = y_i - \mu(x_i, \hat{\beta}) - \left(\frac{1}{n} \sum_{j=1}^n y_j - f(x_j, \hat{\beta})\right)$$
* The second term centers the $e_i$, (should be minor, if ${\rm E}[{\rm E}psilon_i] = 0$). 

* If we assumed that ${\rm E}psilon_i \sim_{ind} N(0,\sigma^2)$ then a parametric resampling residuals method samples from 
$${\rm E}psilon_i^' \sim_{ind} N(0, {\rm Var}\{e_1,e_2,\ldots,e_n\})$$

* Both methods respect the "design" of the $x_i$
* Both methods break any relationship between $x_i$ and ${\rm E}psilon_i$, so they assume
homoskedasticity. This is a big assumption, often violated by real data. It is violated by assumption of you believe $Y_i \sim Pois(\lambda_i)$. 

---

## Bootstrap notes

* Bootstrap methods do not work for all functions of interest. For example the maximum $Y_{(n)}$ if you want to estimate the range of $F$. Regularity conditions apply - roughly, asymptotic normality of $\hat{\theta}_n$ must hold.
* There are no small sample guarantees. Informally, if your data look nothing like $F$, you will get misleading results. 
* The bootstrap is very good (i.e. competitive) at producing confidence intervals where the delta method is unhelpful but you must still give a scientificjustification for your estimates (quantiles of F? number of modes in F?)
* Bootstrapping requires as much thinking as working results out theoretically for complex cases. 

---

## More bootstrap notes

* Theoretical validation of bootstrap methods is non-trivial; checking by simulating the bootstrap is quite slow
* Choosing $B$ is not totally trivial
  * Try $B$ small for early analysis
  * Set $B$ huge for publication.
*  Note this is a Monte Carlo method, we need to generate random numbers. Sandwich approach may be more appealing when possible on practical and rhetorical grounds. 
* But the real advantage of the bootstrap is for complicated parameters (see for example [the Felsenstein paper](http://statweb.stanford.edu/~nzhang/Stat366/Felsenstein85.pdf)) where straightforward calculation of the sandwich/model based standard errors are not possible. 

---

## The jackknife

* The jackknife, invented by Tukey and Quenouille is closely related to the bootstrap. The jackknife is a leave one out procedure. Let $$Y_{(i)} = \{Y_1,Y_2,\ldots, Y_{i-1},Y_{i+1},\ldots Y_n\}$$

* Similarly let $\hat{\theta}_{(i)}$ be the estimate of a parameter based on $Y_{(i)}$. Then the jackknife standard error estimate is: $$\hat{{\rm Var}}_{jack} = \frac{n-1}{n} (\sum \hat{\theta}_i - \bar{\hat{\theta}})^2$$

* The reason for the $n-1$ term is that the jackknife deviations are very close to each other (since we only leave out one observation), so they need to be inflated. 

* The jackknife can be thought of as a linear approximation to the bootstrap (see Efron 1979 or Efron and Tibshirani). For linear statistics:
$$\hat{\theta} = \mu + \frac{1}{n} \sum_{i=1}^n \alpha(x_i)$$
they will agree up to a constant. 

---

## Non-linear statistics and the jackknife

* For non-linear statistics, particularly non-smooth statistics, this approximation can lead to problems. For example the sample median is not smooth. Consider the set of data $$\{10,27,31,40,46,50,52,104,146\}$$
* The median of the values is 46. If we start increasing the 4th largest value ($x=40$). The median doesn't change at all until $x$ becomes larger than 46, then the median is equal to $x$ until $x$ exceeds 50. 
* So the median is not smooth (differentiable) function of $x$. 
* This lack of smoothness causes the jackknife estimate of the standard error to be inconsistent for the median. For the mouse data, the jackknife values for the median are:
$$\{48,48,48,48,45,43,43,43\}$$

* There are only 3 values, a consequence of the lack of smoothness of the median and the fact that the jackknife data sets only differ by 1 observation. The estimated jackknife variance is 44.62. A bootstrap estimate of the standard error based on $B=100$ bootstrap samples is 91.72, considerably larger. 

---

## Vector outcomes

* For univariate $Y$, nonparametric inference by sandwich methods is very close (in spirit and numerically) to use of the bootstrap; both use "plug-in" estimates of $F$, where the estimate is based on the data. 
* This result, and validity of bootstrap methods, also holds for analysis of vector outcomes; by studying an empirical, data-based estimate of $F$, we learn about e.g. ${\rm Var}_F[\hat{\beta}_n]$.

* Two competing strategies for estimating $F$ are:
  * Resample clusters with replacement, and within them sample observations $\{Y_{\cdot j},X_{\cdot j}\}$ without replacement (S1)
  * Resample clusters with replacement, and within each cluster sample observations $\{Y_{\cdot j},X_{\cdot j}\}$ with replacement. (S2)


---

## Comparing S1 and S2 

Assume

$$Y_{ij} = b_i + Z_{ij}, 1 \leq i \leq n, 1 \leq j \leq n_i$$
where $n_i$ is constant for all i, and all $b_i$ and $Z_{ij}$ are independent with: ${\rm E}[b_i] = 0$, ${\rm E}[Z_{ij}] = 0$,  ${\rm Var}[b_i] = \sigma^2_b$,  ${\rm Var}[Z_{ij}] = \sigma^2_Z$
It follows that:

$$ {\rm Var}[Y_{ij}] = \sigma_b^2 + \sigma^2_Z$$
$${\rm Cov}[Y_{ij},Y_{ij'}] = {\rm Cov}[b_i + Z_{ij}, b_i + Z_{ij'}]$$
$$= {\rm Var}[b_i] + {\rm Cov}[Z_{ij},Z_{ij'}] = \sigma^2_b$$

Note the intraclass correlation is $\frac{\sigma^2_b}{\sigma^2_b + \sigma^2_Z}$; it can be interpreted as the correlation between two outcomes in the same cluster, or the proportion of total variance that is between clusters.

---

## Comparing S1 and S2 (cont)

Denoting the bootstrap distribution by $^'$, and denoting the random cluster number by $I^'$, under either S1 or S2:


$${\rm E}_{F^'}[Y_{I'j} | I^' = i] = n_i^{-1} \sum_l Y_{il} = \bar{Y}_i$$
$${\rm E}_{F^'}[Y_{I'j}^2 | I^' = i] = n_i^{-2} \sum_l Y_{il}^2$$

and consequently:

$${\rm E}_{F^'}[Y_{ij}] = n_i^{-1} n^{-1} \sum_{ij} Y_{ij}$$
$${\rm E}_{F^'}[Y_{ij}^2] = n^{-1} \sum_{i} (\bar{Y}_i - \bar{Y})^2 + n^{-1} \sum_i n_i^{-1} (Y_{ij} - \bar{Y}_i)^2$$ 


---

## Comparing S1 and S2 (cont)

The expectation of the resampled outcomes is unbaised for ${\rm E}_{F}[Y_{\cdot j}]$. Slightly more work gives:
$${\rm E}\left[{\rm Var}^'[Y^'_{\cdot j}]\right] = \frac{n-1}{n}\sigma^2_b + \frac{nn_i -1}{nn_i}\sigma^2_Z$$

---

## Off diagonal of covariance

For the cross-terms with $j\neq k$ we get:
$${\rm E}_{F^'}[Y_{I^'j}Y_{I^'k} | I^' = i] = \frac{1}{n_i(n_i-1)}\sum_{l \neq m} Y_{il}Y_{im}$$
$${\rm E}_{F^'}[Y_{I^'j}Y_{I^'k} | I^' = i] = \frac{1}{n_i^2}\sum_{l \neq m} Y_{il}Y_{im}$$
for models (S1) and (S2).Given a particular data set the bootstrap(s) give
$${\rm Cov}_{F^'}[Y_{ij}Y_{ik}] = n^{-1} \sum_i(\bar{Y}_i - \bar{Y})^2 - \frac{1}{nn_i(n_i -1)}\sum_{ij} (Y_{ij} - \bar{Y}_i)^2$$
$${\rm Cov}_{F^'}[Y_{ij}Y_{ik}]=\frac{1}{n} (\bar{Y}_i - \bar{Y})^2$$

---

## Off diagonal of covariance (cont)

for models (S1) and (S2) and this leads to:
$$ {\rm E}[{\rm Cov}_{F^'}[Y_{ij}Y_{ik}]]  = \frac{n-1}{n}\sigma^2_b - \frac{1}{nn_i} \sigma^2_Z$$
$$  {\rm E}[{\rm Cov}_{F^'}[Y_{ij}Y_{ik}]]   = \frac{n-1}{n}\sigma^2_b - \frac{n-1}{nn_i} \sigma^2_Z$$

Recall that this covariance is just $\sigma^2_b$ in the true F; which formula above
is the closest to that value? 

---

## Notes on vector outcomes


* Resampling whole clusters leads to better reconstruction of the first two moments of $F$
* Given that we didn't specify a within-cluster correlation structure, naively using a bootstrap that treats observations as i.i.d. within clusters seems unwise. 
*  With observations $\{Y_i, X_{i1}, X_{i2},X_{i3}\}$ you'd resample whole observations. Now we have e.g.  $\{Y_i, Y_{i2}, X_{i1}, X_{i2}\}$ ...  so do the same.
* If $n_i$ is very large or $\sigma^2$ is tiny, resampling within clusters won't hurt
* The default nonparametric bootstrap resamples whole clusters. In R you will have to write
this yourself. 
