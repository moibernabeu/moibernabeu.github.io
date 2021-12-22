---
layout: post
title: Phylogenetic inference and regression for genomic metrics
permalink: /tutorials/metrics_tutorial
---

## Aligning and trimming sequences

First of all, we have to get the interest sequences. For this example we will work using L3 ribosomal protein, which is highly conserved under the tree of life. First we align the data using the program we like, I have chosen MAFFT, which is currently one of the most important alignment softwares.

~~~ bash
mafft-linsi data/sequences/sequence.fa > data/sequences/aln.fa
~~~

From this, we obtain an alignment

~~~ R
library(alignfigR)
library(plot.matrix)

aln <- t(as.data.frame(read_alignment('data/sequences/aln.fa')))
par(mar =c(3, 6, 3, 3))
plot(as.matrix(aln), col = c(1:22), las = 2, xlab = '', main = 'alignment',
     ylab = '', cex.axis=0.7)
~~~

This, as can be seen in the figure, has some regions in black, which means the lack of data, a gap in the alignment. To solve this, we can trim these positions by several algorithms, I like to use `trimal` with `-gappyout` option.

~~~ bash
trimal -in data/sequences/aln.fa -out data/sequences/trim.fa -gappyout
~~~

~~~ R
trim <- t(as.data.frame(read_alignment('data/sequences/trim.fa')))
par(mar =c(3, 6, 3, 3))
plot(as.matrix(trim), col = c(1:22), las = 2, xlab = '',
     main = 'trimmed alignment', ylab = '', cex.axis=0.7)
~~~

~~~ R
dev.off()
~~~

The proportion of unknown information is lower, then the phylogenetic bias due to this lack is removed. With the trimmed alignment we can do the phylogeny.

## Inferring the tree topology

There are two important methodologies to infer the topology of phylogenetic trees.

* **Maximum Likelihood**: it is based on the maximum likelihood estimator, which is the parameter value which maximizes the likelihood function. It is inferred for the branch lengths and for the topology. For this method we will use IQ-TREE.
* **Bayesian Inference**: it is based on the Bayes' Theorem, which allows to incorporate previous information. This provides a posterior distribution of the branches lengths and the tree topology. For this method we will use PhyloBayes.

### Maximum Likelihood
~~~ bash
mkdir data/tree
mkdir data/tree/rpl3
iqtree2 -s data/sequences/trim.fa -m TEST -B 4000 --prefix data/tree/iqtree/rpl3
~~~

`-s` means the input data (i.e.: trimmed alignment of the interest gene), `-m TEST` performs an optimum model selection for the inference considering the alignment, `-B` option is the ultra-fast bootstrap with 4000 replicates, finally, `--prefix` is the command to say where to save the final documents. (More information in http://www.iqtree.org/doc/iqtree-doc.pdf).

~~~ R
library(treeio)
library(ggtree)

mltree <- read.newick('data/tree/iqtree/rpl3.contree', 'support')
ggtree(mltree) +
  geom_tiplab() +
  xlim(c(0, 1.4)) +
  geom_nodelab(geom = 'label', aes(label = support))
~~~

We can also re-root the tree with our outgroup, which has to be carefully selected. We choose the `contree` file, that means the consensus tree of all topologies in the bootstrap calculation.

~~~ R
library(ape)
library(ggtree)

mltree <- read.newick('data/tree/iqtree/rpl3.contree', node.label = 'support')

rmltree <- mltree
rmltree@phylo <- root.phylo(mltree@phylo, outgroup = 'NZ_CP023278',
                            resolve.root = TRUE)

mlp <- ggtree(rmltree) +
  geom_tiplab() +
  xlim(c(0, 1.4)) +
  geom_nodelab(geom = 'label', aes(label = support))

mlp
~~~

### Bayesian inference

Bayesian inference is quite different from ML. First, we have to run 2 or more chains with the same parameters to check if the inference is well done. PhyloBayes reads the alignments in phylip format, they can be converted using `readal`. After that, we will check the convergence between chains and infer the final tree with the posterior samples.

~~~ bash
readal -in data/trim.fa -out data/trim.phy -phylip

mpirun -n 6 pb_mpi -cat -wag -dgam 4 -d data/trim.phy data/chains/ch1/ch1
mpirun -n 6 pb_mpi -cat -wag -dgam 4 -d data/trim.phy data/chains/ch2/ch2
~~~

Above commands execute the Monte Carlo Markov Chains with our alignment. These should be tested, first they could be plotted.

~~~ R
postch1 <- read.table('data/tree/phylobayes/ch1/ch1.trace', header = 1)
postch2 <- read.table('data/tree/phylobayes/ch2/ch2.trace', header = 1)

par(mfrow = c(1, 3))
plot(postch1$length, type = 'l', col = 'steelblue', ylab = '')
lines(postch2$length, col = 'darkorange3')
abline(v = 50, lty = 4, col = 'black')
title('Tree length')
plot(postch1$topo, type = 'l', col = 'steelblue', ylab = '')
lines(postch2$topo, col = 'darkorange3')
abline(v = 50, lty = 4, col = 'black')
title('Topology')
plot(postch1$loglik, type = 'l', col = 'steelblue', ylab = '')
lines(postch2$loglik, col = 'darkorange3')
abline(v = 50, lty = 4, col = 'black')
title('log(likelihood)')
~~~

As we see, both chains start to converge as of, approximately, the 50th iteration. This burning time has to be removed since it distorts the topology of the trees, for this we will use the next code. The `-x` argument provides, first, in which iteration you want to start and, second, how often you want to get data, then `-x <burning> <thinning>`.

~~~ bash
bpcomp -x 50 1 ch1/ch1 ch2/ch2
~~~

This code returns the next syntax:

~~~
ch1/ch1.treelist : 1259 trees
ch2/ch2.treelist : 1251 trees

maxdiff     : 0.225564
meandiff    : 0.014339

bipartition list in : bpcomp.bplist
consensus in        : bpcomp.con.tre
~~~

From this is important to see the `maxdiff` number, as the software authors recommend (https://github.com/bayesiancook/pbmpi/blob/master/pb_mpiManual1.8.pdf):

* `maxdiff` < 0.1:  good run.
* `maxdiff` < 0.3:  acceptable:  gives a good qualitative picture of the posterior consensus.
* 0.3 < `maxdiff` < 1:  the sample is not yet sufficiently large, and the chains have not converged, but this is on the right track.
* if `maxdiff` = 1 even after 10,000 points, this indicates that at least one of the runs is stuck in a local maximum.

Finally we can get the Bayesian consensus tree and see its topology, we will root it to compare its topology with the ML tree or another reference tree.

~~~ R
bitree <- read.newick('data/tree/phylobayes/bpcomp.con.tre', node.label = 'support')

rbitree <- bitree
rbitree@phylo <- root.phylo(bitree@phylo, outgroup = 'NZ_CP023278',
                            resolve.root = TRUE)

bip <- ggtree(rbitree) +
  geom_tiplab() +
  xlim(c(0, 1.5)) +
  geom_nodelab(geom = 'label', aes(label = support))

bip
~~~

We can plot them jointly to compare:

~~~ R
library(ggpubr)
ggarrange(mlp, bip, labels = c('ML', 'BI'))
~~~
