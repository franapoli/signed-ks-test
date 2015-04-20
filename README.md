# signed-ks-test

Modification to R ks.test to obtain signed KS statistic, following the Gene Set Enrichment Analysis method (special case of p=0).

# Example
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
> ks.test.2(1:10, 11:100)$ES
[1] 1
> ks.test.2(11:100, 1:10)$ES
[1] -1
> 
```

## Leading edge
The "edge" variable contains the limiting index for the "leading edge" subset.

```
> ks.test.2(c(1:10, 40), c(11:39, 41:100))$edge
[1] 10
```

## Notes

The source code contains a quick and dirty patch to cope with the low-level function "C_psmirnov2x", which has changed name across R versions (it is now "C_pSmirnov2x"). Code is replicated to account for the two version, should be cleaned up.
