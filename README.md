# signed-ks-test

Modification to R ks.test to:

* Obtain signed KS statistic, following the Gene Set Enrichment
Analysis method (special case of p=0). See
http://www.broadinstitute.org/gsea/index.jsp.

* Give the possibility to set the maximum combined size of the two
samples for exact computation of the p-values in the two-samples
test. In the original ks.test, p-values for samples whose sizes
product is > 10,000 are approximated. In case approximation is used,
ks.test.2 throws a warning. In ks.test a warning is always thrown when
p-values are not computed exactly, **but** when the cause is samples
size.

# Examples
## With original ks.test:
```
> ks.test(1:10, 11:100)$statistic
D 
1 
> ks.test(11:100, 1:10)$statistic
D 
1
```

## With ks.test.2
The additional variable "ES" contains the signed stiatistic.
```
> source("signed-ks-test.R")
> ks.test.2(1:10, 11:100)$ES
[1] 1
> ks.test.2(11:100, 1:10)$ES
[1] -1
```

## Leading edge
The "edge" variable contains the limiting index for the "leading edge" subset.

```
> ks.test.2(c(1:10, 40), c(11:39, 41:100))$edge
[1] 10
```

## p-value Approximation

Normally, ks.test will warn you if it uses approximated p-values:
```
> ks.test(1:10, 5:15)

	Two-sample Kolmogorov-Smirnov test

data:  1:10 and 5:15
D = 0.4545, p-value = 0.2293
alternative hypothesis: two-sided

Warning message:
In ks.test(1:10, 5:15) :
  unable to compute exact p-values in the presence of ties
```
In the following, however, since the size of the two samples multiplied is
larger than 10,000, ks.test is also using approximation (this is documented). **But it won't tell
you**.

```
> ks.test(c(100, 1000), (1:10000)[-c(100, 1000)])

	Two-sample Kolmogorov-Smirnov test

data:  c(100, 1000) and (1:10000)[-c(100, 1000)]
D = 0.9002, p-value = 0.07827
alternative hypothesis: two-sided
``` 
On the other hand, ks.test.2 does warn in the same case:
```
> ks.test.2(c(100, 1000), (1:10000)[-c(100, 1000)])

	Two-sample Kolmogorov-Smirnov test

data:  c(100, 1000) and (1:10000)[-c(100, 1000)]
D = 0.9002, p-value = 0.07827
alternative hypothesis: two-sided

Warning message:
In ks.test.2(c(100, 1000), (1:10000)[-c(100, 1000)]) :
  P-value not computed exactly because of combined sample size
```
Plus, it can overcome the approximation:
```
> ks.test.2(c(100, 1000), (1:10000)[-c(100, 1000)], maxCombSize=10^7)

	Two-sample Kolmogorov-Smirnov test

data:  c(100, 1000) and (1:10000)[-c(100, 1000)]
D = 0.9002, p-value = 0.01998
alternative hypothesis: two-sided
```
Note that in this example, at a significance threshold of 0.05, you would reject the
null hypothesis when using the exact p-value, but not when using the
approximated p-value.

## Gene Set Enrichment Analysis (GSEA)

ks.test.2 can be used for rank-based GSEA. Suppose you have a gene
expression profiles P including 22000 genes sorted from the most
upregulated to the most downregulated. Suppose you also have a gene
set S containing 5 genes, and such genes are ranked 1000th, 21000th,
21100th, 21200th and 22000th in P. The following code compute the GSEA
of S against the profile P.

```
> S <- c(1000, 21500, 21800, 21900, 22000)
> k <- ks.test.2(S, (1:22000)[-S], maxCombSize=10^10)
> k$ES; k$p
[1] -0.777404
[1] 0.001097664
```

The set is significantly downregulated. In particular this is the GSEA
method with parameter p=0. The p-value is exact (no permutations),
although correlations are not taken into account. Works fine in
practice and allows to compute thousands of GSEAs in seconds
(Napolitano et al 2016, Bioinformatics).


# Notes

The source code contains a quick patch to cope with the low-level
function "C_psmirnov2x", which has changed name across R versions (it
is now "C_pSmirnov2x") and "C_pkstwo" (which is now "C_pKS2"). The
code tries one (trycatch) and falls back on the other one.
