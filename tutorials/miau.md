---
layout: post
title: Miau
permalink: /tutorials/miau/
---

## Data import

~~~ R
dat <- read.csv('tonemas_02.csv', stringsAsFactors = TRUE)
head(dat)
~~~


## Data plot

~~~ R
par(mfrow = c(2, 2), mar = c(4, 4, 2, 1))
for (i in 1:(dim(dat)[2] - 1)) {
  boxplot(dat$dif ~ dat[, i], main = names(dat)[i], xlab = '')
}
par(mfrow = c(1, 1))
~~~

## ANOVA

### `pcurv`

#### Applicability conditions
~~~ R
table(dat$pcurv)
~~~

CLT is not applicable.

~~~ R
shapiro.test(dat$dif[which(dat$pcurv == 'ta')]) # Normal
shapiro.test(dat$dif[which(dat$pcurv == 'ts')]) # Not normal
# shapiro.test(dat$dif[which(dat$pcurv == 'td')]) # Only 2 observations

library(car)
leveneTest(dat$dif ~ dat$pcurv) # There is variance homogeneity
~~~

ANOVA test for this data cannot be applied due to the not normal distribution of
difference in `ts` and `td` (which has only 2 observations) groups. We have to
do a Kruskal-Wallis test by groups.

#### Tests

~~~ R
kruskal.test(dat$dif, dat$pcurv)
pairwise.wilcox.test(dat$dif, dat$pcurv)
~~~

### `curvf0`

#### Applicability conditions
~~~ R
table(dat$curvf0)
~~~

CLT is applicable to `ts` which is the unique category without normal distribution.

~~~ R
shapiro.test(dat$dif[which(dat$curvf0 == 'ta')]) # Normal
shapiro.test(dat$dif[which(dat$curvf0 == 'ts')]) # Not normal, CLT
shapiro.test(dat$dif[which(dat$curvf0 == 'td')]) # Normal

library(car)
leveneTest(dat$dif ~ dat$curvf0) # There is variance homogeneity
~~~

Normality and variance homogeneity are accomplished and then we will apply ANOVA.

#### Tests

~~~ R
anr <- aov(dat$dif ~ dat$curvf0)
summary(anr)
~~~

There are differences between groups. Post-hoc pairwise tests should be conducted
to reveal significantly distinct groups.

~~~ R
pairwise.t.test(dat$dif, dat$curvf0, p.adjust.method = 'bonferroni')
~~~

`ta` is different to `td` and `ts` but `td` is not different to `ts`.

### `taorig`

#### Applicability conditions
~~~ R
table(dat$taorig)
~~~

CLT is applicable to `td` which is the unique category without normal distribution.

~~~ R
shapiro.test(dat$dif[which(dat$taorig == 'ta')]) # Normal
shapiro.test(dat$dif[which(dat$taorig == 'ts')]) # Normal
shapiro.test(dat$dif[which(dat$taorig == 'td')]) # Not normal, CLT

library(car)
leveneTest(dat$dif ~ dat$taorig) # There is variance homogeneity
~~~

Normality and variance homogeneity are accomplished and then we will apply ANOVA.

#### Tests

~~~
anr <- aov(dat$dif ~ dat$taorig)
summary(anr)
~~~

There are differences between groups. Post-hoc pairwise tests should be conducted
to reveal significantly distinct groups.

~~~
pairwise.t.test(dat$dif, dat$taorig, p.adjust.method = 'bonferroni')
~~~


## Groups comparisons
All groups are different between them.

~~~
by(dat$dif, dat$pcurv, summary)
by(dat$dif, dat$curvf0, summary)
by(dat$dif, dat$tacord, summary)
by(dat$dif, dat$taorig, summary)
~~~

~~~
table(dat$pcurv, dat$curvf0)
table(dat$pcurv, dat$tacord)
table(dat$pcurv, dat$taorig)
table(dat$curvf0, dat$tacord)
table(dat$curvf0, dat$taorig)
table(dat$tacord, dat$taorig)
~~~
