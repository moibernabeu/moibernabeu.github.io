---
layout: post
title: "`ggcollapse`, an R package to collapse nodes using ggtree"
author: "Moisès Bernabeu"
categories: resources
tags: [resources,R,visualisation]
image: ggcollapse.png
---


This packages has been programmed to incorporate a function that
collapses clades obtaining symmetric triangles and collapsed nodes with
the same size across the tree. However, we also developed some functions
that help to annotate and find specific nodes in the tree, such as all
the monophyletic clades of tips belonging to a category or all the
sister clades to a specific node of the tree.

## Installing `ggcollapse`

To install `ggcollapse`, you just needs to execute the following
command:

```r
if (!require('devtools', quietly = TRUE)) {
    install.packages('devtools')
}

devtools::install_github('moibernabeu/ggcollapse')
```

## Data for `ggcollapse`

Although the functions are versatile, the main idea of them is to use
three data types, a `phylo` object for the tree, a `data.frame` for the
data, and named vectors for the nodes (`numeric`) to collapse and the
colours of the collapsed clades (`character`).

### Tree and their tip labels
In this tutorial we will simulate our data, first, we simulate a tree:

```r
library(treeio)

set.seed(2024-11-04)
tree <- rtree(20)
```

To see the whole potential of the package, we will assign species to
each tip, the tip label format chosen for this tutorial will be
`tip_SPECIES`, therefore,

```r
tree$tip.label[3:20] <- paste(tree$tip.label[3:20], LETTERS[2:19], sep = '_')
# Adding a species duplication in the tree
tree$tip.label[1:2] <- paste(tree$tip.label[1:2], 'A', sep = '_')
```

The final tree, with the number of the internal nodes, would be:

```r
library(ggtree)

ggtree(tree) +
  geom_tiplab() +
  geom_nodelab(aes(label = node), geom = 'label') +
  xlim(0, 5.5)
```

### Function to get the species from the tip label
To extract the species from each tip, we will design a function, which
will be added to the `ggcollapse` functions later on:

```r
library(stringr)
get_sp <- function(tip.label) {
  return(str_split(tip.label, '_', n = 2, simplify = TRUE)[, 2])
}

tree$tip.label[1]
get_sp(tree$tip.label[1])
```

Other options, and advantages, of using a species-getting function are the
search of the species in a table relating the tip value with the species, a
vector, etc.

### Data linked to the tree
Imagine that the species of your tree are distributed into two groups. To
incorporate this information to the tree we can take two strategies:

1. generate a `data.frame` where each tip has the information, in this case,
the species-getting function must return the header, or

2. generate a `data.frame` relating the species with its information, in this
case, the species-generating function must return the species.

In this tutorial, we will use the second strategy, to generate de `data.frame`:

```r
tree_data <- data.frame(species = LETTERS[1:19],
                        group = c(rep('G1', 7), rep('G2', 3), rep('G3', 9)))
head(tree_data, n = 4)
```

## `ggcollapse` functions

### Annotating the tree with the data using `annotate_tree`
To incorporate the tree data (`tree_data`) into the tree, we can use the 
`annotate_tree` function. This function associates the tip labels with the
information of the tree data table by using a linking function, in this case,
the species-getting function. We added the argument `data_sp_column` to force
the user to incorporate the value (as character or numeric) of the column where
the species code is. The resulting tree is a `treedata S4` object.

```r
library(ggcollapse)
annot_tree <- annotate_tree(tree = tree, tree_data = tree_data, get_sp = get_sp,
                            data_sp_column = 'species')
```

Now, we can plot data across the tree, we can plot the group they belong using
points on the tips:

```r
ggtree(annot_tree) +
  geom_tiplab(offset = 0.035) +
  geom_tippoint(aes(colour = group)) +
  xlim(0, 5.5)
```


