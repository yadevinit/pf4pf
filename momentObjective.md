Exploring Objectives with `R`-Optimization Infrastructure (`ROI`)
================
Vinit Kaushik
December 13, 2020

We shall adapt various vignettes for this research viewpoint, while building further on what's discussed at earlier viewpoint [Risk-free Nay Ratna](https://github.com/yadevinit/pf4pf/blob/main/riskfreeNayRatna.md). That way, we raise workability and avoid inadvertent programming errors. From each heading's hyperlink, readers can also refer to the vignette for the corresponding heading's text to aid understanding. As a vignette says, this is to "solve complex optimization problems." The reader might find this viewpoint technically challenging, but wading through this will help the reader prefer a smaller set of Objectives to refine further (e.g., using custom Moments) and test them in out-of-sample settings using re-balancing, before possibly considering investing using them. The [author](mailto:yadevinit@gmail.com) recommends that the reader at least browse through the content under heading [6.4 Maximize quadratic utility with ROI](#maximize-quadratic-utility-with-roi); that seems to call for further support using a coherent risk measure `ES`, maybe superior Moment estimates, and out-of-sample test via re-balancing.

A. [Custom Moment and Objective Functions 2018-May](https://cran.r-project.org/web/packages/PortfolioAnalytics/vignettes/custom_moments_objectives.pdf) Adapted
===============================================================================================================================================================

1 Getting Started: Load Packages and Data
-----------------------------------------

``` r
library(ROI)
library(ROI.plugin.glpk)
library(ROI.plugin.quadprog)
# library(DEoptim)
library(PortfolioAnalytics)

date()
```

    ## [1] "Sun Dec 13 23:55:14 2020"

``` r
matchVignette <- FALSE # TRUE iff vignette data (including dates) and outputs are to be matched
quRiskAversion <- c(0.25, 2, 10, 25)[ifelse(matchVignette, 1, 2)]
  # Alt: 10 which shows up in Example 4. 25 is within 20-30 range mentioned in CMU paper.
if(matchVignette){
  data(edhec)
    # Use the first 4 columns in edhec for a returns object
  cDateRange <- '::2014-01-31' # Was '::2018-05-17' coz specified vignette's date is 2018-May-17
  R <- edhec[cDateRange, 1:4] # Subset to match cDateRange, instead of tail() with 2019 data too.
  colnames(R) <- c("CA", "CTAG", "DS", "EM")
} else {
  cMyPath <- "F:/pf4pf/data/indices/"; warning("cannot include others' data into repos")
  cFileIndex <- "assets-"
  gIx <- readRDS(file=paste0(cMyPath, cFileIndex, ".rds"))
  cDateRange <- c("2001/", # gets earliest large slice of assets.
    "2007/", # gets at least a window of a year to train/learn on before entering 2008 GFC.
      # "2004/" includes GLD and GOLDBEES. Early "2006" includes SLV too.
    "2010-07/" # TSLA data starts.
  )[3]; cDateRange <- paste0(cDateRange, "2020-11")
  argIsLogReturns <- FALSE
  if(! argIsLogReturns){
    R <- equity.data <- myAssetsWithReturnsOverCompletePeriod(argIsLogReturns, argix=gIx,
      dateRange=cDateRange, myPeriod="weeks", minProp=1.0)
  } else {
    R <- equity.data <- myAssetsWithReturnsOverCompletePeriod(argIsLogReturns, argix=gIxLogR,
      dateRange=cDateRange, myPeriod="weeks", minProp=1.0)
  }
}
```

    ## Warning: cannot include others' data into repos

    ## Daily periodicity from 2010-07-01 05:30:00 to 2020-11-30 05:30:00 
    ## [1] 3567  118
    ## [1] 543  99

``` r
str(R)
```

    ## An 'xts' object on 2010-07-18 05:30:00/2020-11-30 05:30:00 containing:
    ##   Data: num [1:543, 1:99] 0.00824 0.00759 -0.01018 0.01291 0.00535 ...
    ##  - attr(*, "dimnames")=List of 2
    ##   ..$ : NULL
    ##   ..$ : chr [1:99] "BSE200" "BSEAUTO" "BSEBASICMAT" "BSECD" ...
    ##   Indexed by objects of class: [POSIXct,POSIXt] TZ: 
    ##   xts Attributes:  
    ##  NULL

``` r
head(R, 2)
```

    ##                          BSE200      BSEAUTO BSEBASICMAT      BSECD     BSECDGS
    ## 2010-07-18 05:30:00 0.008239170 -0.003825049 0.002444461 0.03839841 0.010919144
    ## 2010-07-25 05:30:00 0.007594995  0.009131884 0.030051809 0.01675882 0.003169761
    ##                          BSECG      BSECPSE  BSEDOLLEX20    BSEENERGY
    ## 2010-07-18 05:30:00 0.02411872 -0.015656079 0.0082353551 -0.006427263
    ## 2010-07-25 05:30:00 0.02117535  0.009216533 0.0005506979 -0.002110699
    ##                          BSEFIN      BSEFMCG  BSEHEALTHCA BSEINDUSTRI
    ## 2010-07-18 05:30:00 0.028336166  0.005077060 -0.003545624  0.02223386
    ## 2010-07-25 05:30:00 0.005831841 -0.002527536 -0.017617141  0.01315488
    ##                            BSEIT    BSEMETAL    BSEOILGAS    BSEPOWER
    ## 2010-07-18 05:30:00 -0.003373770 0.004878393 -0.010390482 0.008661369
    ## 2010-07-25 05:30:00  0.002551948 0.039811106  0.001043825 0.003520056
    ##                           BSEPSU   BSESENSEX      BSETECK  BSETELECOM
    ## 2010-07-18 05:30:00 -0.008444176 0.006833342 -0.006679993 -0.02583848
    ## 2010-07-25 05:30:00  0.014515151 0.009707781  0.007341158  0.03199057
    ##                          BSEUTIL         DAX         DJCA         DJIA
    ## 2010-07-18 05:30:00 -0.008771642 -0.00412540 -0.008279836 -0.009867083
    ## 2010-07-25 05:30:00  0.008384869  0.02065676  0.038369418  0.031842835
    ##                         FTSE100         HSI          NDX      SNP500
    ## 2010-07-18 05:30:00 0.005052577 -0.00632558 -0.006251627 -0.01220825
    ## 2010-07-25 05:30:00 0.029357979  0.02752704  0.039093174  0.03486333
    ##                       BANKNIFTY   CNXINFRA        CNXIT   CNXMIDCAP CNXNIFTYJUN
    ## 2010-07-18 05:30:00 0.030391929 0.01011040 -0.003657391 0.011236342 0.011732552
    ## 2010-07-25 05:30:00 0.008002092 0.01382238  0.002346860 0.003383233 0.003655916
    ##                       CNXREALTY     GOLDBEES         IDBI         INFY
    ## 2010-07-18 05:30:00 0.062631644  0.007385025 -0.010595046 -0.034263502
    ## 2010-07-25 05:30:00 0.007203017 -0.001938146 -0.003282728  0.003393538
    ##                     NIFTYMIDCAP   S.PCNX500     SENIFTY         A50
    ## 2010-07-18 05:30:00 0.003653168 0.006367598 0.007714285 0.014449038
    ## 2010-07-25 05:30:00 0.011003871 0.007366924 0.010181772 0.001167267
    ##                           AQVL30        B50  commodities         L50
    ## 2010-07-18 05:30:00 0.0008246419 0.01668495 -0.005734708 0.001796895
    ## 2010-07-25 05:30:00 0.0081002175 0.00566949  0.018769416 0.004994857
    ##                           media        N10GS    N10GSclean        NE50
    ## 2010-07-18 05:30:00  0.02211045  0.002164096  0.0006954955 0.007814224
    ## 2010-07-25 05:30:00 -0.01525502 -0.001635340 -0.0031679272 0.009434791
    ##                      NGrowthS15       pharma      V20N50     BPCL.BO
    ## 2010-07-18 05:30:00  0.01153909 -0.008793705 0.008826769 -0.07395073
    ## 2010-07-25 05:30:00 -0.00214595 -0.013418648 0.014045194 -0.04418973
    ##                       BIOCON.BO         VNQ          GLD         SLV
    ## 2010-07-18 05:30:00 -0.00805758 -0.01476196 -0.014381416 -0.01362883
    ## 2010-07-25 05:30:00  0.01776709  0.05918654 -0.004983702  0.01306460
    ##                              DBA          USO         DBB           BND
    ## 2010-07-18 05:30:00  0.029313443 -0.005255494 -0.03646654  0.0055331847
    ## 2010-07-25 05:30:00 -0.004760128  0.035373911  0.06387998 -0.0004894736
    ##                               UUP         RSP         IWR         MDY
    ## 2010-07-18 05:30:00 -0.0177506637 -0.01526001 -0.01829170 -0.01787781
    ## 2010-07-25 05:30:00 -0.0008333945  0.04411759  0.04604512  0.04859518
    ##                              VB         IJR       W5000         RUT         IWM
    ## 2010-07-18 05:30:00 -0.02256974 -0.02611693 -0.01388730 -0.03071652 -0.03016106
    ## 2010-07-25 05:30:00  0.05788439  0.05696459  0.03900164  0.06387378  0.06205905
    ##                              SH       TSLA        FCHI    STOXX50E         N225
    ## 2010-07-18 05:30:00  0.01080900 0.17076073 -0.01540012 -0.01336273 -0.018634101
    ## 2010-07-25 05:30:00 -0.03610233 0.03100654  0.03008162  0.02741023  0.002399197
    ##                              BFX        STI       JKSE         NZ50        KS11
    ## 2010-07-18 05:30:00 -0.008154158 0.01380475 0.01635855 -0.006506435 0.008921116
    ## 2010-07-25 05:30:00  0.025591362 0.00531092 0.01642934  0.003056485 0.011217078
    ##                            TWII        GSPTSE          MXX        KLSE
    ## 2010-07-18 05:30:00 0.002262291 -6.912689e-05 -0.006926752 0.009274888
    ## 2010-07-25 05:30:00 0.012531224  1.241217e-02  0.031668532 0.006733000
    ##                            IPSA         MERV      TA125.TA         SSMI
    ## 2010-07-18 05:30:00 0.009677999 -0.002812395  0.0200984850 -0.004214675
    ## 2010-07-25 05:30:00 0.024426788  0.039560566 -0.0008613025  0.002725724
    ##                            AXJO SSE000001.S   PFIZER.NS  GRAPHITE.NS
    ## 2010-07-18 05:30:00 0.005987180 -0.01906094  0.05311250 -0.009900176
    ## 2010-07-25 05:30:00 0.008039522  0.05916386 -0.02721141  0.009257105
    ##                          HEG.NS       ONGC.NS  BANKBARODA.  YESBANK.NS
    ## 2010-07-18 05:30:00 -0.03920931 -0.0275828926  0.015442932 0.047005178
    ## 2010-07-25 05:30:00 -0.02456158  0.0009955463 -0.003303891 0.007982693
    ##                         IDEA.NS    SUVENadj
    ## 2010-07-18 05:30:00 -0.02658725  0.01663932
    ## 2010-07-25 05:30:00  0.05540385 -0.05425059

``` r
tail(R, 2)
```

    ##                         BSE200    BSEAUTO BSEBASICMAT      BSECD    BSECDGS
    ## 2020-11-29 05:30:00 0.01178693 0.02443564  0.02611533 0.01316303 0.02111778
    ## 2020-11-30 05:30:00 0.00000000 0.00000000  0.00000000 0.00000000 0.00000000
    ##                           BSECG    BSECPSE BSEDOLLEX20  BSEENERGY     BSEFIN
    ## 2020-11-29 05:30:00 0.001977576 0.02691363   0.0145085 0.01703918 0.01088152
    ## 2020-11-30 05:30:00 0.000000000 0.00000000   0.0000000 0.00000000 0.00000000
    ##                        BSEFMCG BSEHEALTHCA BSEINDUSTRI       BSEIT   BSEMETAL
    ## 2020-11-29 05:30:00 0.01197878  0.02976199  0.01520111 0.007866626 0.05774032
    ## 2020-11-30 05:30:00 0.00000000  0.00000000  0.00000000 0.000000000 0.00000000
    ##                      BSEOILGAS   BSEPOWER     BSEPSU   BSESENSEX     BSETECK
    ## 2020-11-29 05:30:00 0.02571177 0.01855182 0.02699201 0.006076675 0.001579093
    ## 2020-11-30 05:30:00 0.00000000 0.00000000 0.00000000 0.000000000 0.000000000
    ##                      BSETELECOM    BSEUTIL          DAX        DJCA
    ## 2020-11-29 05:30:00 -0.02889731 0.03805014  0.014991443  0.01985916
    ## 2020-11-30 05:30:00  0.00000000 0.00000000 -0.003343997 -0.00965757
    ##                             DJIA      FTSE100         HSI          NDX
    ## 2020-11-29 05:30:00  0.021864921  0.002536358  0.01661412 0.0291164864
    ## 2020-11-30 05:30:00 -0.009126328 -0.016050977 -0.02078323 0.0008244134
    ##                           SNP500  BANKNIFTY    CNXINFRA      CNXIT  CNXMIDCAP
    ## 2020-11-29 05:30:00  0.022460987 0.01267923 0.005695435 0.01647551 0.03891886
    ## 2020-11-30 05:30:00 -0.004606081 0.00000000 0.000000000 0.00000000 0.00000000
    ##                     CNXNIFTYJUN  CNXREALTY    GOLDBEES       IDBI         INFY
    ## 2020-11-29 05:30:00  0.01265623 0.03863895 -0.02238986 0.01854358 -0.003040827
    ## 2020-11-30 05:30:00  0.00000000 0.00000000  0.00000000 0.00000000  0.000000000
    ##                     NIFTYMIDCAP  S.PCNX500     SENIFTY        A50     AQVL30
    ## 2020-11-29 05:30:00  0.04912803 0.01489717 0.008510195 0.03245673 0.01920617
    ## 2020-11-30 05:30:00  0.00000000 0.00000000 0.000000000 0.00000000 0.00000000
    ##                            B50 commodities        L50      media        N10GS
    ## 2020-11-29 05:30:00 0.05851812  0.02021176 0.01238525 0.02567353 -0.001122271
    ## 2020-11-30 05:30:00 0.00000000  0.00000000 0.00000000 0.00000000  0.000000000
    ##                       N10GSclean       NE50  NGrowthS15     pharma    V20N50
    ## 2020-11-29 05:30:00 -0.002281185 0.01802093 0.005570704 0.02674142 0.0131527
    ## 2020-11-30 05:30:00  0.000000000 0.00000000 0.000000000 0.00000000 0.0000000
    ##                         BPCL.BO  BIOCON.BO          VNQ          GLD
    ## 2020-11-29 05:30:00 -0.02750403 0.02177966  0.003303814 -0.046007934
    ## 2020-11-30 05:30:00  0.00000000 0.00000000 -0.010539490 -0.006697358
    ##                             SLV          DBA          USO        DBB
    ## 2020-11-29 05:30:00 -0.06617025  0.017391743  0.064495782 0.01405187
    ## 2020-11-30 05:30:00  0.00000000 -0.006406172 -0.008039921 0.00406145
    ##                               BND          UUP         RSP          IWR
    ## 2020-11-29 05:30:00 -0.0001130993 -0.006842450  0.02833422  0.025701277
    ## 2020-11-30 05:30:00  0.0013569822  0.002420332 -0.01076543 -0.007580388
    ##                             MDY          VB         IJR        W5000
    ## 2020-11-29 05:30:00  0.02723896  0.03171905  0.03755485  0.027550592
    ## 2020-11-30 05:30:00 -0.01723404 -0.01587121 -0.02264507 -0.005737333
    ##                             RUT         IWM           SH        TSLA
    ## 2020-11-29 05:30:00  0.03842139  0.03797397 -0.023392932  0.17930104
    ## 2020-11-30 05:30:00 -0.01929268 -0.01833704  0.004294156 -0.03149326
    ##                            FCHI    STOXX50E         N225         BFX
    ## 2020-11-29 05:30:00  0.01844101  0.01720888  0.042839491  0.03660911
    ## 2020-11-30 05:30:00 -0.01432647 -0.01004235 -0.007954015 -0.01413414
    ##                             STI        JKSE       NZ50        KS11        TWII
    ## 2020-11-29 05:30:00  0.01510395  0.03728823 0.01579040  0.03082979  0.01092325
    ## 2020-11-30 05:30:00 -0.01761690 -0.02999938 0.01012978 -0.01611960 -0.01045318
    ##                          GSPTSE          MXX         KLSE IPSA        MERV
    ## 2020-11-29 05:30:00  0.02193854 -0.005521899  0.008646412    0  0.07363851
    ## 2020-11-30 05:30:00 -0.01192945  0.002506085 -0.028314673    0 -0.01554075
    ##                        TA125.TA          SSMI         AXJO  SSE000001.S
    ## 2020-11-29 05:30:00  0.04041510  0.0005266792  0.009421453  0.009011773
    ## 2020-11-30 05:30:00 -0.01578683 -0.0023596599 -0.012699451 -0.004866690
    ##                       PFIZER.NS GRAPHITE.NS      HEG.NS    ONGC.NS BANKBARODA.
    ## 2020-11-29 05:30:00 0.008657211  0.03052876 -0.01938343 0.09130544  0.07688313
    ## 2020-11-30 05:30:00 0.000000000  0.00000000  0.00000000 0.00000000  0.00000000
    ##                     YESBANK.NS     IDEA.NS    SUVENadj
    ## 2020-11-29 05:30:00  0.0102565 -0.02519025 -0.01175855
    ## 2020-11-30 05:30:00  0.0000000  0.00000000  0.00000000

``` r
# Get a character vector of the fund names
funds <- colnames(R)
```

2 Setting the Portfolio Moments
-------------------------------

``` r
library(PortfolioAnalytics)
# Construct initial portfolio with basic constraints.
init.portFL <- portfolio.spec(assets=funds)
init.portFL <- add.constraint(portfolio=init.portFL, type="full_investment")
init.portFL <- add.constraint(portfolio=init.portFL, type="long_only")
# Portfolio with standard deviation as an objective
SD.portf <- add.objective(portfolio=init.portFL, type="risk", name="StdDev")
# Portfolio with expected shortfall as an objective
ES.portf <- add.objective(portfolio=init.portFL, type="risk", name="ES")
sd.moments <- set.portfolio.moments(R, SD.portf)
names(sd.moments)
```

    ## [1] "mu"    "sigma"

``` r
es.moments <- set.portfolio.moments(R, ES.portf)
names(es.moments)
```

    ## [1] "mu"    "sigma" "m3"    "m4"

3 Custom Moment Functions
-------------------------

``` r
sigma.robust <- function(R){
  require(MASS)
  out <- list()
  set.seed(1234)
  out$sigma <- cov.rob(R, method="mcd")$cov
    # Ref https://r.789695.n4.nabble.com/Using-optimize-portfolio-td4763805.html:
    # sigma <- cov(returns) # but then I changed it to
    # sigma <- cov(returns, use = "pairwise.complete.obs") # per Brian's suggestion
  return(out)
}
opt.sd <- optimize.portfolio(R, SD.portf,
  optimize_method="ROI",
  momentFUN="sigma.robust")
opt.sd
```

    ## ***********************************
    ## PortfolioAnalytics Optimization
    ## ***********************************
    ## 
    ## Call:
    ## optimize.portfolio(R = R, portfolio = SD.portf, optimize_method = "ROI", 
    ##     momentFUN = "sigma.robust")
    ## 
    ## Optimal Weights:
    ##      BSE200     BSEAUTO BSEBASICMAT       BSECD     BSECDGS       BSECG 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##     BSECPSE BSEDOLLEX20   BSEENERGY      BSEFIN     BSEFMCG BSEHEALTHCA 
    ##      0.0000      0.0000      0.0003      0.0000      0.0000      0.0000 
    ## BSEINDUSTRI       BSEIT    BSEMETAL   BSEOILGAS    BSEPOWER      BSEPSU 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##   BSESENSEX     BSETECK  BSETELECOM     BSEUTIL         DAX        DJCA 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0018 
    ##        DJIA     FTSE100         HSI         NDX      SNP500   BANKNIFTY 
    ##      0.0019      0.0000      0.0000      0.0000      0.4640      0.0000 
    ##    CNXINFRA       CNXIT   CNXMIDCAP CNXNIFTYJUN   CNXREALTY    GOLDBEES 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0010 
    ##        IDBI        INFY NIFTYMIDCAP   S.PCNX500     SENIFTY         A50 
    ##      0.0005      0.0007      0.0000      0.0000      0.0000      0.0000 
    ##      AQVL30         B50 commodities         L50       media       N10GS 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0031 
    ##  N10GSclean        NE50  NGrowthS15      pharma      V20N50     BPCL.BO 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##   BIOCON.BO         VNQ         GLD         SLV         DBA         USO 
    ##      0.0000      0.0000      0.0021      0.0000      0.0030      0.0000 
    ##         DBB         BND         UUP         RSP         IWR         MDY 
    ##      0.0000      0.0081      0.0060      0.0182      0.0000      0.0000 
    ##          VB         IJR       W5000         RUT         IWM          SH 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.4881 
    ##        TSLA        FCHI    STOXX50E        N225         BFX         STI 
    ##      0.0000      0.0003      0.0000      0.0000      0.0000      0.0000 
    ##        JKSE        NZ50        KS11        TWII      GSPTSE         MXX 
    ##      0.0003      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##        KLSE        IPSA        MERV    TA125.TA        SSMI        AXJO 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0005 
    ## SSE000001.S   PFIZER.NS GRAPHITE.NS      HEG.NS     ONGC.NS BANKBARODA. 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##  YESBANK.NS     IDEA.NS    SUVENadj 
    ##      0.0000      0.0000      0.0000 
    ## 
    ## Objective Measure:
    ##    StdDev 
    ## 0.0004377

``` r
  # Beware: possibly due to data range, this output differs
  # from that of vignette (though the Objective StdDev is close) which says:
  # Optimal Weights:
  # CA CTAG DS EM
  # 0.6598 0.1441 0.1961 0.0000
  # Objective Measure:
  # StdDev
  # 0.008646
  # but StdDev verifies sqrt() == extractObjectiveMeasures()$StdDev.
  # So, robust estimate of the covariance matrix was indeed used.
weights <- extractWeights(opt.sd)
sigma <- sigma.robust(R)$sigma
sqrt(t(weights) %*% sigma %*% weights)
```

    ##              [,1]
    ## [1,] 0.0004377499

``` r
extractObjectiveMeasures(opt.sd)$StdDev
```

    ##       StdDev 
    ## 0.0004377499

``` r
chart.Weights(opt.sd)
```

![](momentObjective_files/figure-markdown_github/CustomMoment-1.png)

B. [Introduction to PortfolioAnalytics 2018-May](https://cran.r-project.org/web/packages/PortfolioAnalytics/vignettes/portfolio_vignette.pdf) Adapted
=====================================================================================================================================================

6.1 Initial Portfolio Object
----------------------------

``` r
if(matchVignette){
  data(edhec)
  R <- edhec[cDateRange, 1:6]
  colnames(R) <- c("CA", "CTAG", "DS", "EM", "EQMN", "ED")
  dim(R)
  head(R, 2); tail(R, 2)
  funds <- colnames(R)
  # Create an initial portfolio object with leverage and box constraints
  init <- portfolio.spec(assets=funds)
  init <- add.constraint(portfolio=init, type="leverage",
    min_sum=0.99, max_sum=1.01)
  init <- add.constraint(portfolio=init, type="box", min=0.05, max=0.65)
} else {
  init <- init.portFL # portf0LI
}
```

6.2 Maximize mean return with ROI
---------------------------------

``` r
maxret <- add.objective(portfolio=init, type="return", name="mean")
print(maxret)
```

    ## **************************************************
    ## PortfolioAnalytics Portfolio Specification 
    ## **************************************************
    ## 
    ## Call:
    ## portfolio.spec(assets = funds)
    ## 
    ## Number of assets: 99 
    ## Asset Names
    ##  [1] "BSE200"      "BSEAUTO"     "BSEBASICMAT" "BSECD"       "BSECDGS"    
    ##  [6] "BSECG"       "BSECPSE"     "BSEDOLLEX20" "BSEENERGY"   "BSEFIN"     
    ## More than 10 assets, only printing the first 10
    ## 
    ## Constraints
    ## Enabled constraint types
    ##      - full_investment 
    ##      - long_only 
    ## 
    ## Objectives:
    ## Enabled objective names
    ##      - mean

``` r
# Run the optimization.
opt_maxret <- optimize.portfolio(R=R, portfolio=maxret, optimize_method="ROI", trace=TRUE)
print(opt_maxret)
```

    ## ***********************************
    ## PortfolioAnalytics Optimization
    ## ***********************************
    ## 
    ## Call:
    ## optimize.portfolio(R = R, portfolio = maxret, optimize_method = "ROI", 
    ##     trace = TRUE)
    ## 
    ## Optimal Weights:
    ##      BSE200     BSEAUTO BSEBASICMAT       BSECD     BSECDGS       BSECG 
    ##           0           0           0           0           0           0 
    ##     BSECPSE BSEDOLLEX20   BSEENERGY      BSEFIN     BSEFMCG BSEHEALTHCA 
    ##           0           0           0           0           0           0 
    ## BSEINDUSTRI       BSEIT    BSEMETAL   BSEOILGAS    BSEPOWER      BSEPSU 
    ##           0           0           0           0           0           0 
    ##   BSESENSEX     BSETECK  BSETELECOM     BSEUTIL         DAX        DJCA 
    ##           0           0           0           0           0           0 
    ##        DJIA     FTSE100         HSI         NDX      SNP500   BANKNIFTY 
    ##           0           0           0           0           0           0 
    ##    CNXINFRA       CNXIT   CNXMIDCAP CNXNIFTYJUN   CNXREALTY    GOLDBEES 
    ##           0           0           0           0           0           0 
    ##        IDBI        INFY NIFTYMIDCAP   S.PCNX500     SENIFTY         A50 
    ##           0           0           0           0           0           0 
    ##      AQVL30         B50 commodities         L50       media       N10GS 
    ##           0           0           0           0           0           0 
    ##  N10GSclean        NE50  NGrowthS15      pharma      V20N50     BPCL.BO 
    ##           0           0           0           0           0           0 
    ##   BIOCON.BO         VNQ         GLD         SLV         DBA         USO 
    ##           0           0           0           0           0           0 
    ##         DBB         BND         UUP         RSP         IWR         MDY 
    ##           0           0           0           0           0           0 
    ##          VB         IJR       W5000         RUT         IWM          SH 
    ##           0           0           0           0           0           0 
    ##        TSLA        FCHI    STOXX50E        N225         BFX         STI 
    ##           1           0           0           0           0           0 
    ##        JKSE        NZ50        KS11        TWII      GSPTSE         MXX 
    ##           0           0           0           0           0           0 
    ##        KLSE        IPSA        MERV    TA125.TA        SSMI        AXJO 
    ##           0           0           0           0           0           0 
    ## SSE000001.S   PFIZER.NS GRAPHITE.NS      HEG.NS     ONGC.NS BANKBARODA. 
    ##           0           0           0           0           0           0 
    ##  YESBANK.NS     IDEA.NS    SUVENadj 
    ##           0           0           0 
    ## 
    ## Objective Measure:
    ##     mean 
    ## 0.009382

``` r
plot(opt_maxret, risk.col="StdDev", return.col="mean",
  main="Maximum Return Optimization", chart.assets=TRUE)
```

![](momentObjective_files/figure-markdown_github/maxMeanROI-1.png)

``` r
  # , xlim=c(0, 0.05), ylim=c(0,0.0085)
# extractStats(opt_maxret)
chart.Weights(opt_maxret)
```

![](momentObjective_files/figure-markdown_github/maxMeanROI-2.png) That does maximize (mean) return, but if the investor is unwilling to tolerate the associated risk (or in this case, volatility in the form of `StdDev`) and such a concentrated portfolio, we need to explore further.

6.4 Maximize quadratic utility with ROI
---------------------------------------

You might recall earlier research viewpoint [Nay Ratna](https://github.com/yadevinit/pf4pf/blob/main/nayRatna/nayRatna.md) sharing a commitment to transform the wealth ecosystem onto another realm of growth with harmonized incentive structures, e.g., a business model with fees based on Net Present Value. That seems to be consistent with an investor-customizable risk aversion, as part of a (CRRA-maximization or other) portfolio objective. And the harmonization vibes with ["Skin in the Game: Hidden Asymmetries in Daily Life"](https://en.wikipedia.org/wiki/Skin_in_the_Game_(book)):

> If an actor pockets some rewards from a policy they enact or support without accepting any of the risks, economists consider it to be a problem of "missing incentives". In contrast, to Taleb, the problem is more fundamentally one of asymmetry: one actor gets the rewards, the other is stuck with the risks.\[1\] Taleb argues that "For social justice, focus on symmetry and risk sharing. You cannot make profits and transfer the risks to others, as bankers and large corporations do ... Forcing skin in the game corrects this asymmetry better than thousands of laws and regulations."

``` r
qu <- add.objective(portfolio=init, type="return", name="mean")
qu <- add.objective(portfolio=qu, type="risk", name="var", risk_aversion=quRiskAversion)
# Run the optimization.
opt_qu <- optimize.portfolio(R=R, portfolio=qu, optimize_method="ROI", trace=TRUE)
print(opt_qu)
```

    ## ***********************************
    ## PortfolioAnalytics Optimization
    ## ***********************************
    ## 
    ## Call:
    ## optimize.portfolio(R = R, portfolio = qu, optimize_method = "ROI", 
    ##     trace = TRUE)
    ## 
    ## Optimal Weights:
    ##      BSE200     BSEAUTO BSEBASICMAT       BSECD     BSECDGS       BSECG 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##     BSECPSE BSEDOLLEX20   BSEENERGY      BSEFIN     BSEFMCG BSEHEALTHCA 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ## BSEINDUSTRI       BSEIT    BSEMETAL   BSEOILGAS    BSEPOWER      BSEPSU 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##   BSESENSEX     BSETECK  BSETELECOM     BSEUTIL         DAX        DJCA 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##        DJIA     FTSE100         HSI         NDX      SNP500   BANKNIFTY 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##    CNXINFRA       CNXIT   CNXMIDCAP CNXNIFTYJUN   CNXREALTY    GOLDBEES 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##        IDBI        INFY NIFTYMIDCAP   S.PCNX500     SENIFTY         A50 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##      AQVL30         B50 commodities         L50       media       N10GS 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##  N10GSclean        NE50  NGrowthS15      pharma      V20N50     BPCL.BO 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0562 
    ##   BIOCON.BO         VNQ         GLD         SLV         DBA         USO 
    ##      0.2859      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##         DBB         BND         UUP         RSP         IWR         MDY 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##          VB         IJR       W5000         RUT         IWM          SH 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##        TSLA        FCHI    STOXX50E        N225         BFX         STI 
    ##      0.2918      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##        JKSE        NZ50        KS11        TWII      GSPTSE         MXX 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##        KLSE        IPSA        MERV    TA125.TA        SSMI        AXJO 
    ##      0.0000      0.0000      0.2344      0.0000      0.0000      0.0000 
    ## SSE000001.S   PFIZER.NS GRAPHITE.NS      HEG.NS     ONGC.NS BANKBARODA. 
    ##      0.0000      0.0516      0.0000      0.0000      0.0000      0.0000 
    ##  YESBANK.NS     IDEA.NS    SUVENadj 
    ##      0.0000      0.0000      0.0801 
    ## 
    ## Objective Measure:
    ##     mean 
    ## 0.006288 
    ## 
    ## 
    ##  StdDev 
    ## 0.03318

``` r
  # Output differs from vignette's, especially weights for DS and EM assets and objectives
  # mean and StdDev:
  # Optimal Weights:
  # CA CTAG DS EM EQMN ED
  # 0.0500 0.0500 0.2714 0.5386 0.0500 0.0500
  # Objective Measure:
  # mean
  # 0.007926
  # StdDev
  # 0.02663
plot(opt_qu, risk.col="StdDev", return.col="mean",
  main=paste0("Risk Aversion=", quRiskAversion, " Quadratic Utility Optimization"),
  chart.assets=TRUE) # , xlim=c(0, 0.05), ylim=c(0, 0.0085)
```

![](momentObjective_files/figure-markdown_github/maxQuadUtilityROI-1.png)

``` r
chart.Weights(opt_qu)
```

![](momentObjective_files/figure-markdown_github/maxQuadUtilityROI-2.png)

6.5 Minimize expected tail loss with ROI
----------------------------------------

If the exploration so far is making the investor loss averse, let's take this position now, with a coherent measure of risk `ETL` (also known as `CVaR` or `ES`) instead of `StdDev`.

``` r
etl <- add.objective(portfolio=init, type="risk", name="ETL")
# Run the optimization.
opt_etl <- optimize.portfolio(R=R, portfolio=etl, optimize_method="ROI", trace=TRUE)
print(opt_etl)
```

    ## ***********************************
    ## PortfolioAnalytics Optimization
    ## ***********************************
    ## 
    ## Call:
    ## optimize.portfolio(R = R, portfolio = etl, optimize_method = "ROI", 
    ##     trace = TRUE)
    ## 
    ## Optimal Weights:
    ##      BSE200     BSEAUTO BSEBASICMAT       BSECD     BSECDGS       BSECG 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##     BSECPSE BSEDOLLEX20   BSEENERGY      BSEFIN     BSEFMCG BSEHEALTHCA 
    ##      0.0000      0.0000      0.0000      0.0000      0.0030      0.0000 
    ## BSEINDUSTRI       BSEIT    BSEMETAL   BSEOILGAS    BSEPOWER      BSEPSU 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##   BSESENSEX     BSETECK  BSETELECOM     BSEUTIL         DAX        DJCA 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0059 
    ##        DJIA     FTSE100         HSI         NDX      SNP500   BANKNIFTY 
    ##      0.0246      0.0000      0.0000      0.0186      0.3745      0.0000 
    ##    CNXINFRA       CNXIT   CNXMIDCAP CNXNIFTYJUN   CNXREALTY    GOLDBEES 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0132 
    ##        IDBI        INFY NIFTYMIDCAP   S.PCNX500     SENIFTY         A50 
    ##      0.0000      0.0030      0.0000      0.0000      0.0000      0.0000 
    ##      AQVL30         B50 commodities         L50       media       N10GS 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0125 
    ##  N10GSclean        NE50  NGrowthS15      pharma      V20N50     BPCL.BO 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##   BIOCON.BO         VNQ         GLD         SLV         DBA         USO 
    ##      0.0011      0.0000      0.0000      0.0000      0.0052      0.0000 
    ##         DBB         BND         UUP         RSP         IWR         MDY 
    ##      0.0000      0.0565      0.0457      0.0009      0.0000      0.0000 
    ##          VB         IJR       W5000         RUT         IWM          SH 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.4277 
    ##        TSLA        FCHI    STOXX50E        N225         BFX         STI 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##        JKSE        NZ50        KS11        TWII      GSPTSE         MXX 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##        KLSE        IPSA        MERV    TA125.TA        SSMI        AXJO 
    ##      0.0000      0.0069      0.0000      0.0000      0.0000      0.0000 
    ## SSE000001.S   PFIZER.NS GRAPHITE.NS      HEG.NS     ONGC.NS BANKBARODA. 
    ##      0.0000      0.0007      0.0000      0.0000      0.0000      0.0000 
    ##  YESBANK.NS     IDEA.NS    SUVENadj 
    ##      0.0000      0.0000      0.0000 
    ## 
    ## Objective Measure:
    ##      ETL 
    ## 0.002683

``` r
# Differs from vignette's CTAG and EQMN weights as well as objective ETL:
# Optimal Weights:
# CA CTAG DS EM EQMN ED
# 0.0500 0.2968 0.0500 0.0500 0.4932 0.0500
# Objective Measure:
# ETL
# 0.01967
plot(opt_etl, risk.col="ES", return.col="mean", main="ETL Optimization", chart.assets=TRUE)
```

![](momentObjective_files/figure-markdown_github/minETLwithROI-1.png)

``` r
  # , xlim=c(0, 0.14), ylim=c(0,0.0085)
chart.Weights(opt_etl)
```

![](momentObjective_files/figure-markdown_github/minETLwithROI-2.png)

C. [Maximizing Modified Sharpe Ratio Demo 2014](https://github.com/braverock/PortfolioAnalytics/blob/master/demo/demo_max_STARR.R) Adapted
==========================================================================================================================================

This gets you what's called as a tangency portfolio.

``` r
#' This script demonstrates how to solve a constrained portfolio optimization 
#' problem to maximize modified Sharpe Ratio using ES as the risk measure.
# full_investment and long_only without relaxed weights are preferred.
init.portf <- add.objective(portfolio=init.portFL, type="return", name="mean")
init.portf <- add.objective(portfolio=init.portf, type="risk", name="ES",
                            arguments=list(p=0.925))
init.portf
```

    ## **************************************************
    ## PortfolioAnalytics Portfolio Specification 
    ## **************************************************
    ## 
    ## Call:
    ## portfolio.spec(assets = funds)
    ## 
    ## Number of assets: 99 
    ## Asset Names
    ##  [1] "BSE200"      "BSEAUTO"     "BSEBASICMAT" "BSECD"       "BSECDGS"    
    ##  [6] "BSECG"       "BSECPSE"     "BSEDOLLEX20" "BSEENERGY"   "BSEFIN"     
    ## More than 10 assets, only printing the first 10
    ## 
    ## Constraints
    ## Enabled constraint types
    ##      - full_investment 
    ##      - long_only 
    ## 
    ## Objectives:
    ## Enabled objective names
    ##      - mean 
    ##      - ES

``` r
#' Maximizing STARR Ratio can be formulated as a linear programming 
#' problem and solved very quickly using optimize_method="ROI". 
#' The default action if "mean" and "ES" are specified as objectives with
#' optimize_method="ROI" is to maximize STARR. If we want to use
#' both mean and ES in the objective function, but only minimize ES, we need to 
#' pass in maxSTARR=FALSE to optimize.portfolio.
maxSTARR.lo.ROI <- optimize.portfolio(R=R, portfolio=init.portf, optimize_method="ROI",
  trace=TRUE) # maxSTARR=TRUE default for mean-ES
maxSTARR.lo.ROI
```

    ## ***********************************
    ## PortfolioAnalytics Optimization
    ## ***********************************
    ## 
    ## Call:
    ## optimize.portfolio(R = R, portfolio = init.portf, optimize_method = "ROI", 
    ##     trace = TRUE)
    ## 
    ## Optimal Weights:
    ##      BSE200     BSEAUTO BSEBASICMAT       BSECD     BSECDGS       BSECG 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##     BSECPSE BSEDOLLEX20   BSEENERGY      BSEFIN     BSEFMCG BSEHEALTHCA 
    ##      0.0000      0.0000      0.0000      0.0000      0.0214      0.0000 
    ## BSEINDUSTRI       BSEIT    BSEMETAL   BSEOILGAS    BSEPOWER      BSEPSU 
    ##      0.0000      0.0261      0.0000      0.0000      0.0000      0.0000 
    ##   BSESENSEX     BSETECK  BSETELECOM     BSEUTIL         DAX        DJCA 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##        DJIA     FTSE100         HSI         NDX      SNP500   BANKNIFTY 
    ##      0.0000      0.0000      0.0000      0.0016      0.0000      0.0000 
    ##    CNXINFRA       CNXIT   CNXMIDCAP CNXNIFTYJUN   CNXREALTY    GOLDBEES 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.1057 
    ##        IDBI        INFY NIFTYMIDCAP   S.PCNX500     SENIFTY         A50 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##      AQVL30         B50 commodities         L50       media       N10GS 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.4066 
    ##  N10GSclean        NE50  NGrowthS15      pharma      V20N50     BPCL.BO 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##   BIOCON.BO         VNQ         GLD         SLV         DBA         USO 
    ##      0.0322      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##         DBB         BND         UUP         RSP         IWR         MDY 
    ##      0.0000      0.0945      0.1654      0.0000      0.0000      0.0000 
    ##          VB         IJR       W5000         RUT         IWM          SH 
    ##      0.0000      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##        TSLA        FCHI    STOXX50E        N225         BFX         STI 
    ##      0.0153      0.0000      0.0000      0.0000      0.0000      0.0000 
    ##        JKSE        NZ50        KS11        TWII      GSPTSE         MXX 
    ##      0.0000      0.1029      0.0000      0.0000      0.0000      0.0000 
    ##        KLSE        IPSA        MERV    TA125.TA        SSMI        AXJO 
    ##      0.0000      0.0000      0.0069      0.0000      0.0000      0.0000 
    ## SSE000001.S   PFIZER.NS GRAPHITE.NS      HEG.NS     ONGC.NS BANKBARODA. 
    ##      0.0028      0.0118      0.0000      0.0000      0.0000      0.0000 
    ##  YESBANK.NS     IDEA.NS    SUVENadj 
    ##      0.0024      0.0000      0.0044 
    ## 
    ## Objective Measure:
    ##     mean 
    ## 0.001581 
    ## 
    ## 
    ##       ES 
    ## 0.007594

``` r
chart.RiskReward(maxSTARR.lo.ROI, risk.col="ES", return.col="mean", chart.assets=TRUE)
```

![](momentObjective_files/figure-markdown_github/maxModifiedSharpeSTARR-1.png)

``` r
chart.Weights(maxSTARR.lo.ROI)
```

![](momentObjective_files/figure-markdown_github/maxModifiedSharpeSTARR-2.png)

``` r
# Calculate STARR ratio from efficient frontier and optimization; they are identical:
ef2 <- create.EfficientFrontier(R=R, portfolio=init.portf, type="mean-ES", n.portfolios=100)
```

    ## Warning: executing %dopar% sequentially: no parallel backend registered

``` r
max(ef2$frontier[,"mean"] / ef2$frontier[,"ES"])
```

    ## [1] 0.2080282

``` r
maxSTARR.lo.ROI$objective_measures$mean / maxSTARR.lo.ROI$objective_measures$ES
```

    ## [1] 0.2082183

``` r
chart.EfficientFrontier(ef2)
```

![](momentObjective_files/figure-markdown_github/maxModifiedSharpeSTARR-3.png)

Appendix
--------

Here's the runtime environment used. It's reported here for reproducibility:

``` r
Sys.info()[['sysname']]
```

    ## [1] "Windows"

``` r
sessionInfo()
```

    ## R version 3.6.2 (2019-12-12)
    ## Platform: x86_64-w64-mingw32/x64 (64-bit)
    ## Running under: Windows 8.1 x64 (build 9600)
    ## 
    ## Matrix products: default
    ## 
    ## locale:
    ## [1] LC_COLLATE=English_India.1252  LC_CTYPE=English_India.1252   
    ## [3] LC_MONETARY=English_India.1252 LC_NUMERIC=C                  
    ## [5] LC_TIME=English_India.1252    
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ## [1] ROI.plugin.quadprog_1.0-0  ROI.plugin.glpk_1.0-0     
    ## [3] ROI_1.0-0                  MASS_7.3-51.4             
    ## [5] PortfolioAnalytics_1.1.0   PerformanceAnalytics_2.0.4
    ## [7] foreach_1.4.8              xts_0.12.1                
    ## [9] zoo_1.8-7                 
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_1.0.3          knitr_1.27          magrittr_1.5       
    ##  [4] lattice_0.20-38     rlang_0.4.4         quadprog_1.5-8     
    ##  [7] stringr_1.4.0       tools_3.6.2         grid_3.6.2         
    ## [10] xfun_0.19           registry_0.5-1      htmltools_0.4.0    
    ## [13] iterators_1.0.12    yaml_2.2.1          digest_0.6.24      
    ## [16] numDeriv_2016.8-1.1 Rglpk_0.6-4         codetools_0.2-16   
    ## [19] evaluate_0.14       slam_0.1-47         rmarkdown_2.1      
    ## [22] stringi_1.4.6       compiler_3.6.2
