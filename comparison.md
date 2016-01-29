# Comparing the Sensitivity and Linearity of the QE and QTOF

## Sensitivity
Comparing the sensitivity of these instruments is complicated due to the QE's varying sensitivity in proportion to the ion accumulation time (inj. time or IT).  As ion flux increases beyond the capacity of the trap, some flux is discarded.  This decreases LOD in that region of the chromatogram.  

In contrast, sensitivity of the QTOF is constant throughout the run.

The goal here is to determine what minimum inj. time is necessary for the QE to equal the sensitivity of the QTOF.  With this information we will see if we can partition the mass range into regions which decrease total flux such that sensitivity is preserved.  The limit to the number of partitions is the cycle time and therefore peakwidth.

We varied the accumulation time of AcCoA in negative ion mode from 1ms to ~200ms (filling the trap with only AcCoA).  We will compare this with the longest QTOF transient averaging time of 1.2s.

## Linearity
The QTOF should be better than the QE at producing linear response.  THis will be tested with the isotopic distribution of AcCoA.

## Experimental
The dionex was set to flow at 50uL/min of 95% A HILIC solvents.  The flow path contained a manual injection valve and a 35mmx0.5mm C18 column.  The column served to increase mixing and clean up the background. 5uL of mixture was injected sequentially, changing the relevant parameters between each injection.  

### QE
Small needle, C, 170 um; SOurce: 30,11,0,3.2,310,60,250,30k,85-1250Da
### QTOF
Regular needle; Source: 200,4,44,300,12,3000,2000,140,65,750

# Setup

```r
knitr::opts_chunk$set(fig.width=11, fig.height=7, fig.align="center", message=F, warning=F, fig.ext="png")

library(ggplot2)

theme_nate <- function(base_size = 16)
{
  theme_grey(base_size = base_size) %+replace%
    theme(
      strip.background = element_rect(colour="grey95"), 
      strip.text = element_text(base_size*0.9, colour="grey40", face="plain"),
      
      panel.background = element_blank(),
      panel.grid.major = element_line(colour="grey95"),
      panel.grid.minor = element_line(colour="grey99"),
      plot.title = element_text(size = rel(1.2)),
      
      axis.title.y = element_text(vjust=1, angle=90),
      axis.title = element_text(colour="grey40"),
      axis.line = element_line(colour="grey95"),
      axis.ticks = element_line(colour="grey95"),
      
      legend.position="top",
      legend.key = element_blank(),
      legend.text = element_text(colour="grey40"),
      legend.title  = element_text(colour="grey40", face="bold")
    )
}

theme_set(theme_nate())
```

# Files

```r
library(xcms)

qe = xcmsRaw("qe_vary_injtime.mzXML", profstep = 0)
qe
```

```
## An "xcmsRaw" object with 1990 mass spectra
## 
## Time range: 0-442.4 seconds (0-7.4 minutes)
## Mass range: 85.0019-1274.8229 m/z
## Intensity range: 99.6792-23712900 
## 
## MSn data on  0  mass(es)
## 	with  0  MSn spectra
## Profile method: bin 
## Profile step: no profile data
## 
## Memory usage: 15.4 MB
```

```r
qt = xcmsRaw("qtof_vary_hz.mzXML", profstep=0)
qt
```

```
## An "xcmsRaw" object with 3252 mass spectra
## 
## Time range: 3.4-411.8 seconds (0.1-6.9 minutes)
## Mass range: 85.0006-1248.809 m/z
## Intensity range: 200-120130 
## 
## MSn data on  0  mass(es)
## 	with  0  MSn spectra
## Profile method: bin 
## Profile step: no profile data
## 
## Memory usage: 0.686 MB
```

# AcCoA Isotopes

```r
accoa.formula = "C23H37N7O17P3S" # Deprotonated
c1213 = 1.003355

accoa = as.data.frame(matrix(c(808,100,809,29.4683,810,12.0383,811,2.659,812,0.5707,813,0.0948,814,0.014,815,0.0015,816,0.0001), ncol=2, byrow = T, dimnames=list(NULL,c("m", "i"))))
accoa$am = sapply(0:8, function(i) { 808.117959 + i*c1213 })
accoa
```

```
##     m        i       am
## 1 808 100.0000 808.1180
## 2 809  29.4683 809.1213
## 3 810  12.0383 810.1247
## 4 811   2.6590 811.1280
## 5 812   0.5707 812.1314
## 6 813   0.0948 813.1347
## 7 814   0.0140 814.1381
## 8 815   0.0015 815.1414
## 9 816   0.0001 816.1448
```

```r
ppm = 808/1E6

ggplot(accoa) + geom_segment(aes(x = am, xend=am, y = i, yend =0))
```

<img src="figure/unnamed-chunk-3-1.png" title="plot of chunk unnamed-chunk-3" alt="plot of chunk unnamed-chunk-3" style="display: block; margin: auto;" />

# EICs

```r
qe.eic = getEIC(qe, mzrange = cbind(accoa[,"am"]-ppm*5, accoa[,"am"]+ppm*5), rtrange = cbind(rep(0, nrow(accoa)), rep(max(qe@scantime), nrow(accoa))))@eic[[1]]

qe.eic = as.data.frame(do.call(rbind, lapply(seq_along(qe.eic), function(i) { cbind(qe.eic[[i]], iso = i) })))
qe.eic$iso = factor(qe.eic$iso)

ggplot(qe.eic) + geom_line(aes(x = rt, y = intensity, colour = iso)) + facet_grid(iso~., scales = "free_y") + ggtitle("QE Isos of AcCoA")
```

<img src="figure/unnamed-chunk-4-1.png" title="plot of chunk unnamed-chunk-4" alt="plot of chunk unnamed-chunk-4" style="display: block; margin: auto;" />
From left to right the ion accumulation times are approximately: 10ms, 5ms, 1ms, 16ms, 30ms, 60ms, 90ms, 90ms, 200ms (100% full trap with all AcCoA)


```r
qt.eic = getEIC(qt, mzrange = cbind(accoa[,"am"]-ppm*5, accoa[,"am"]+ppm*5), rtrange = cbind(rep(0, nrow(accoa)), rep(max(qe@scantime), nrow(accoa))))@eic[[1]]

qt.eic = as.data.frame(do.call(rbind, lapply(seq_along(qt.eic), function(i) { cbind(qt.eic[[i]], iso = i) })))
qt.eic$iso = factor(qt.eic$iso)

ggplot(qt.eic) + geom_line(aes(x = rt, y = intensity, colour = iso)) + facet_grid(iso~., scales = "free_y") + ggtitle("QTOF Isos of AcCoA")
```

<img src="figure/unnamed-chunk-5-1.png" title="plot of chunk unnamed-chunk-5" alt="plot of chunk unnamed-chunk-5" style="display: block; margin: auto;" />
From left to right the scan rates are: 1Hz, 5Hz, 10Hz, 20Hz, 0.86Hz, 1Hz

# Conlusions
Notation: 
QTOF@1 means QTOF at 1 scan/second
QE@30 means QE at 30 milliseconds accumulation

The QE@10 approximates or superscedes the LOD of the QTOF@1.

# Brief Anecdote About Regions to Split
*Caution, approximations*

In an analysis of E.Coli the minimum IT (at maximum ion flux) was 1ms. The maximum IT was 4ms. The mean was 3ms.

We can achieve a scan rate of:
8Hz @ 35k
4Hz @ 70k
2Hz @ 140k

Minimum peak width for the Luna method is ~15s.
We want 15 scans across each peak.

Therefore at 70k RP we can carve up the mass range into 4 regions.  If flux is equally distributed across them then at maximum ion flux we will underpreform the QTOF.  See table for more examples.

|Hz|RP|Max Regions|IT Per Region (min)|IT Per Region (max)|%QTOF (Way Approx) (max)|
|---|---|---|---|---|---|---|
|8|35k|8|8ms|32ms|320%|
|4|70k|4|4ms|16ms|160%|
|2|140k|2|2ms|8ms|80%|

**Note: A large amount of the ion flux in this method is persistent background!! The use of a cleaner method would make the QE scream compared to the QTOF.  (Motivation)**

----

# Further Analysis if Necessary
 - Linearity - deviation from theoretical isotope distribution
 - Linearity at different inject times and different scan rates
 - In depth analysis of regions where the QE will outpreform and regions where the QTOF will outpreform in an actual sample