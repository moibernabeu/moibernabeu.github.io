---
title: "Statistical inference"
author: "Moisès Bernabeu"
date: "2022"
output:
  md_document:
    variant: markdown_github
    preserve_yaml: true
---

## Inference concept

### Coin toss heads and tails probability inference

**Definition**: We define a random variable *X* as the function that
maps every element of the sample space *Ω* to a measurable space *E*. It
is *X* : *Ω* → *E*.

**Example**: When we have discrete data, like a factor or number of
rains per year, *E* are natural numbers, then *E* = ℕ and *X* : *Ω* → ℕ.
When the variable is continuous, like the height of women,
*E* = ℝ<sup>+</sup>, then *X* : *Ω* → ℝ<sup>+</sup>, finally, if the
variable can take any real values *E* = ℝ, then *X* : *Ω* → ℝ.

Let 𝒟 = (*d*<sub>1</sub>,*d*<sub>2</sub>,…,*d*<sub>*n*</sub>) be the
**random variable** of the event: having a head on a coin toss trial,
where *n* is the sample size and each *d*<sub>*i*</sub> will be an
element of the random variable. If sample space is *Ω* = {*T*, *H*}, and
the success of this trial is “having a head” (like we defined the random
variable). *d*<sub>*i*</sub> = 1 in 𝒟 will be denoting having a head,
and *d*<sub>*i*</sub> = 0 a tile. Then, from an experiment of *n* = 10,
we should get a data vector like:
𝒟 = (1,0,0,0,1,0,0,1,1,0).

The distribution of this kind of data (success and failure) is called
Bernoulli. It is a particular case of Binomial distributed data.
Binomial distribution explains the number of successes in an *n* length
sequence of trials (e.g. number of baskets scored in 10 shots). When the
number of trials is 1, the distribution is called Bernoulli, then it is
the success or failure of a single shot, coin toss, etc.

## Variables

There exist two types of quantitative variables:

1.  **Discrete variables**: are the variables that takes integer values,
    ℤ. For instance, the number of heads in 10 coin toss trials, the
    variable ranges 0 to 10, but 1.5 is an impossible outcome.
2.  **Continuous variables**: are the expressed in real numbers
    variables, ℝ. The diameter of a pine tree, for instance, it ranges
    positive reals, and 20.34 is a possible value of the variable.

Random variables, have an associated uncertainty, the mathematical
framework for this uncertainty is provided by the distribution that the
variable follows. We define distribution as the mathematical formulas
and structures that associate each possible event or a set of events to
their probabilities.

### Distribution characterisation

#### Discrete variables

Distributions are characterized by two functions, in discrete variables,
the **Probability Mass Function** (PMF, or simply the probability
function) associates a probability to an event of the sample using the
random variable. The probability function is denoted *f*(*x*) and its
definition is *f*(*x*<sub>*i*</sub>) = *P*(*X*=*x*<sub>*i*</sub>), where
*x*<sub>*i*</sub> is an element of the random variable *X* that is
linked to an element of the sample space. The other function that
defines the distribution is the **Cumulative Distribution Function**
(CDF, or distribution function), as it name says, it accumulates the
values of *P*(*X*=*x*<sub>*i*</sub>), then
$$
F(x) = P(X \\leq x_i) = \\sum\_{i = 0}^{n} P(X = x_i).
$$

<img src="/images/unnamed-chunk-1-1.png" width="80%" style="display: block; margin: auto;" />

In above figure it is seen the distribution function saturates at 1, it
is because the probability ranges 0 to 1, and the probability is
maintained until the next event of the sample space.

#### Continuous variables

In continuous variables there exist the **Probability Density Function**
(PDF, or density function). It does not give us the probability at each
point (because its probability is 0 since in a continuous variable there
exist infinite set of possible values, then the probability of a point
is 1/∞ = 0), but the relative likelihood that a test variable value is
close to this distribution, in this function, the probabilities are
areas under the PMF curve, we can talk about probabilities that the
value is between 2 values
*P*(*x*<sub>*i*</sub>\<*X*\<*x*<sub>*j*</sub>), is greater
*P*(*X*\<*x*<sub>*i*</sub>) or lower *P*(*X*\<*x*<sub>*i*</sub>) (note
we never use the equal, since it is 0, it would be caught, but it will
not change the probability). As discrete variables, it is also
characterized by the **Cumulative Distribution Function** (CDF, or
distribution function), in the continuous case
*F*(*X*=*x*<sub>*i*</sub>) = *P*(*X*\<*x*<sub>*i*</sub>) = ∫<sub>−∞</sub><sup>*x*<sub>*i*</sub></sup>*f*(*X*=*x*) *d**x*

<img src="/images/unnamed-chunk-2-1.png" width="80%" style="display: block; margin: auto;" />

## Bernoulli distribution

As described before, the coin toss experience is a Bernoulli, which
explains the probability of success and failure. We define a Bernoulli
process with the Probability Mass Function
*P*(𝒟=*d*<sub>*i*</sub>;*p*) = *p*<sup>*d*<sub>*i*</sub></sup>(1−*p*)<sup>1 − *d*<sub>*i*</sub></sup>.
Note when *d*<sub>*i*</sub> = 1 (it is a head), the term
*p*<sup>*d*<sub>*i*</sub></sup>(1−*p*)<sup>1 − *d*<sub>*i*</sub></sup> = *p*,
the probability of having a head, and, analogously, when
*d*<sub>*i*</sub> = 0,
*p*<sup>*d*<sub>*i*</sub></sup>(1−*p*)<sup>1 − *d*<sub>*i*</sub></sup> = 1 − *p* = *q*,
which is the probability of having a tail. If we are searching for a
whole population, we want to estimate the *π* parameter, which is the
*p* but for the population.

## Non-parametric estimate

The number of successes divided by the number of events, will give a non
parametric estimate of the *p*, this means that data is not modelled
using distributions having parameters.

``` r
# Coin toss with non-parametric calculation ----
# Setting real parameter
rpi <- 0.3

npest <- c()
for (i in c(5, 10, 50, 100, 500, 1000)) {
  # Setting a seed to have always the same simulation results
  set.seed(0382098)
  
  # Getting a Bernoulli (binomial with sample size = 1) sample of i elements
  # and pi = real pi. 1 = head and 0 tail
  d <- rbinom(i, 1, rpi)
  
  npest <- c(npest, sum(d) / length(d))
}

npest
```

    ## [1] 0.200 0.100 0.240 0.320 0.288 0.300

## Maximum Likelihood Inference

Likelihood methods are based on the maximum likelihood function, we
denote it *L*, and it means the probability of having given data when
the parameter gets a value, then
$$
L(\\pi) = P(\\mathcal{D} \\, \\vert \\, \\pi) = \\prod\_{i = 1}^{n} P(\\mathcal{D} = d_i \\, \\vert \\, \\pi) = \\prod\_{i = 1}^{n} \\pi^{d_i}(1 - \\pi)^{1 - d_i}
$$
is the likelihood for 𝒟 when the parameter of the Bernoulli distribution
is *π*. If we do this for all possible values of the *π* parameter
(i.e. \[0,1\]), we will have the likelihood function for all the
possible parameters, we define, then, the Maximum Likelihood Estimate
(MLE) as the parameter value which maximises the likelihood value. From
this MLE, we obtain a distribution parameter estimator, which usually is
denoted *π̂*.

Next code and figure show the experiment likelihood computations. We
started from a given real parameter (which we want to infer, `rpi`), and
we simulate a data set of size *n*, we calculate the likelihood for each
possible *π* ∈ \[0,1\]. We plotted the function and its MLE as vertical
line and its confidence interval. *π* is assumed to be asymptotically
normal for large *n*s, then, its confidence interval at *α* level will
be given by
$$
\\hat{\\pi} \\pm z\_{1 - \\alpha} \\sqrt{\\frac{\\hat{\\pi}(1-\\hat{\\pi})}{n}},
$$
this works better for samples higher with *n* ≥ 30.

``` r
# Coin toss with ML ----
# Setting real parameter
rpi <- 0.3

par(mfrow = c(2, 3))
for (i in c(5, 10, 50, 100, 500, 1000)) {
  # Setting a seed to have always the same simulation results
  set.seed(0382098)
  
  # Getting a Bernoulli (binomial with sample size = 1) sample of i elements
  # and rpi = real pi. 1 = head and 0 = tail
  d <- rbinom(i, 1, rpi)
  
  # Generating the estimate pi values and computing likelihood as the product
  # of the probability mass function for our data (d sample) evaluated at pi
  # parameter
  pi <- 0:1000/1000
  like <- c()
  for (i in 1:length(pi)) {
    # Computing likelihood
    like <- c(like, prod(dbinom(d, 1, pi[i])))  # 1 - pi[i] will plot q MLE
  }
  
  # Getting the Maximum Likelihood estimate
  MLE <- pi[which.max(like)]
  
  # Getting confidence interval
  epi <- sum(d) / length(d)
  CI95 <- epi + 1 * qnorm(c(0.025, 0.975)) * sqrt(epi * (1 - epi) / length(d))
  
  # Plotting the likelihood function for the paramiter with d data and its MLE
  plot(pi, like, type = 'l', main = paste('n =', length(d)),
       xlab = TeX('$\\pi = P(D = 1)$'), ylab = 'Likelihood')
  abline(v = MLE, lty = 4, col = 'steelblue')
  abline(v = CI95, lty = 4, col = 'darkorange3')
  legend('topright', inset = 0.02,
         legend = c(paste('MLE =', round(MLE, 2)), 'CI 95 %'),
         col = c('steelblue', 'darkorange'), lty = 4)
}
```

<img src="/images/unnamed-chunk-4-1.png" width="100%" style="display: block; margin: auto;" />

While the sample size grows, the likelihood uncertainty on the parameter
space is reduced (dispersion of the likelihood for greater *n* samples
is lower) and it confidence interval is lower. This means that higher
amount of data will give less uncertain parameter. Remember that the
confidence interval is not a probability, but the number of times a
parameter would be in the interval if we repeat the experiment 100
times.

## Bayesian Inference

The main goal of Bayesian inference is the same than frequentist
inference (ML), to learn about a population from a subset of it (the
sample). The difference lies in the conception of the probability, not
at mathematical level, but how to interpret it. In fraquentist
statistics, the probability is undesrtood as the limit of the relative
frequency (*P*(*A*) = lim<sub>*n* → *m*</sub>*f*(*A*)/*n*, where *m* is
the population size). Nevertheless, the Bayesian concept of probability
is linked to the uncertainty of an event.

Bayes’ theorem provides the possibility to calculate conditional
probabilities. Let *A* and *B* be random events, the probability of *A*
given (having occurred) *B* is
$$
P(A \\, \\vert \\, B) = \\frac{P(B \\, \\vert \\, A) P(A)}{P(B)} \\propto P(B \\, \\vert \\, A) P(A).
$$

Using this theorem we can structure a learning from data process. It is
the Bayesian inference process, and it has these components:

1.  **Quantity of interest**: the object of our interest, in this case
    the parameter *π*, which is the probability of having a head in a
    coin toss.
2.  **Prior distribution**: previous known information about the
    quantity of interest, we could say we actually know the probability
    of a head or a tail is both 0.5, this allows us to design a prior
    distribution with more density in 0.5. *P*(*π*)
3.  **Prior predictive distribution**: the prior probability that the
    quantity of interest is greater, lower or is inside an interval
    before doing the experiment, only with the prior distribution.
4.  **Likelihood function**: merges the data with our interest quantity.
    *L*(*π*) = *P*(𝒟 \| *π*).
5.  **Posterior distribution**: Baye’s rule updated probability for the
    interest quantity. This update is made from the data through the
    likelihood function, it is denoted by
    *P*(*π* \| 𝒟) ∝ *P*(𝒟 \| *π*)*P*(*π*).
6.  **Posterior predictive distribution**: the posterior probability of
    having a value greater, lower or inside an interval for the interest
    quantity.

``` r
alpha <- c(0.5, 0.5, 1, 1, 2, 5)
beta <- c(0.5, 1, 1, 1.5, 2, 2)

par(mfrow = c(2, 3))
for (j in 1:length(alpha)) {
  i <- 10
  # Setting a seed to have always the same simulation results
  set.seed(0382098)
  
  # Getting a Bernoulli (binomial with sample size = 1) sample of i elements
  # and rpi = real pi. 1 = head and 0 = tail
  d <- rbinom(i, 1, rpi)
  
  # Generating the estimate pi values and computing likelihood as the product
  # of the probability mass function for our data (d sample) evaluated at pi
  # parameter
  pi <- 0:1000/1000
  
  # Generating a prior which will be used to the Bayesian inference
  prior <- dbeta(seq(0, 1, length.out = length(pi)), alpha[j], beta[j])
  
  like <- c()
  for (i in 1:length(pi)) {
    # Computing likelihood
    like <- c(like, prod(dbinom(d, 1, pi[i])))
  }
  
  # Removing infinity values (they are because of the existence of asymptotes
  # in 0 or 1)
  infprior <- which(prior == Inf)
  if (length(infprior) > 0) {
    prior <- prior[-infprior]
    like <- like[-infprior]
    pi <- pi[-infprior]
  }
  
  # Computing weighted likelihood
  whlike <- prior * like
  
  # Computing normalising bayesian constant by the total probability theorem
  nmctnt <- sum(prior * like)
  
  post <- whlike / nmctnt
  
  # Plotting the likelihood function for the paramiter with d data and its MLE
  if (mean(prior) != 1) {
    plr <- range(c(like, post, prior))
    plot(pi, prior, type = 'l', col = 'darkorange3', lty = 2, lwd = 1.5,
         ylab = 'Relative density', xlab = TeX('$\\pi$'),
         yaxt = 'n', ylim = plr)
    lines(pi, like * max(plr) / max(like), col = 'steelblue', lty = 4, lwd = 1.5)
    lines(pi, post * max(plr) / max(post), type = 'l', col = 'brown3', lwd = 2)
    legend('topright', inset = 0.02, lty = c(2, 4, 1), lwd = 2,
           col = c('darkorange3', 'steelblue', 'brown3'),
           legend = c('Prior', 'Likelihood', 'Posterior'))
    title(paste0('Be(', alpha[j], ', ', beta[j], '), n = 10'))
  } else {
    plr <- c(0, 2)
    plot(pi, prior, type = 'l', col = 'darkorange3', lty = 2, lwd = 1.5,
         ylab = 'Relative density', xlab = TeX('$\\pi$'),
         yaxt = 'n', ylim = plr)
    lines(pi, like * 2 / max(like), col = 'steelblue', lty = 4, lwd = 1.5)
    lines(pi, post * 2 / max(post), type = 'l', col = 'brown3', lwd = 2)
    legend('topright', inset = 0.02, lty = c(2, 4, 1), lwd = 2,
           col = c('darkorange3', 'steelblue', 'brown3'),
           legend = c('Prior', 'Likelihood', 'Posterior'))
    title(paste0('Be(', alpha[j], ', ', beta[j], '), n = 10'))
  }
}
```

<img src="/images/unnamed-chunk-5-1.png" width="100%" style="display: block; margin: auto;" />

``` r
par(mfrow = c(2, 3))
for (i in c(5, 10, 50, 100, 500, 1000)) {
  # Setting a seed to have always the same simulation results
  set.seed(0382098)
  
  # Getting a Bernoulli (binomial with sample size = 1) sample of i elements
  # and rpi = real pi. 1 = head and 0 = tail
  d <- rbinom(i, 1, rpi)
  
  # Generating the estimate pi values and computing likelihood as the product
  # of the probability mass function for our data (d sample) evaluated at pi
  # parameter
  pi <- 0:1000/1000
  
  # Generating a prior which will be used to the Bayesian inference
  prior <- dbeta(seq(0, 1, length.out = length(pi)), 2, 2)
  
  like <- c()
  for (i in 1:length(pi)) {
    # Computing likelihood
    like <- c(like, prod(dbinom(d, 1, pi[i])))
  }
  
  # Removing infinity values (they are because of the existence of asymptotes
  # in 0 or 1)
  infprior <- which(prior == Inf)
  if (length(infprior) > 0) {
    prior <- prior[-infprior]
    like <- like[-infprior]
    pi <- pi[-infprior]
  }
  
  # Computing weighted likelihood
  whlike <- prior * like
  
  # Computing normalising bayesian constant by the total probability theorem
  nmctnt <- sum(prior * like)
  
  post <- whlike / nmctnt
  
  # Plotting the likelihood function for the paramiter with d data and its MLE
  plr <- c(0, 2)
  plot(pi, prior, type = 'l', col = 'darkorange3', lty = 2, lwd = 1.5,
       ylab = 'Relative density', xlab = TeX('$\\pi$'),
       yaxt = 'n', ylim = plr)
  lines(pi, like * 2 / max(like), col = 'steelblue', lty = 4, lwd = 1.5)
  lines(pi, post * 2 / max(post), type = 'l', col = 'brown3', lwd = 2)
  legend('topright', inset = 0.02, lty = c(2, 4, 1), lwd = 2,
         col = c('darkorange3', 'steelblue', 'brown3'),
         legend = c('Prior', 'Likelihood', 'Posterior'))
  title(paste0('Be(2, 2), n = ', length(d)))
}
```

<img src="/images/unnamed-chunk-6-1.png" width="100%" style="display: block; margin: auto;" />
