Rebalancing Weights to AUM to Invest
================
December 18, 2021

This is penned after rebalancing the `bt77` portfolio in 2021-Dec.

Here, the [author](mailto:yadevinit@gmail.com) illustrates a particular `bt77` Portfolio-optimization rebalancing selected from an earlier research viewpoint [500 Invest for 36% CAGR](https://github.com/yadevinit/pf4pf): to move it from its latest-rebalancing weights to the (Assets Under Management) AUM monies required to invest using it where fractional shares are unsupported. This demonstrates feasibility of really investing with it, of course, without any advice or Suitability recommendation to do so for anyone referring to this. Looks like less than INR 5 lakhs (Rs 500,000) suffices to invest on the latest Portfolio rebalancing. And we saw in the previous viewpoint that this `bt77` is one of those whose performance exceeded the return target of 36% CAGR. We also saw there that its Information Ratio exceeded 1 during the period of study when using as benchmark an Equal-Weighted Portfolio rebalancing quarterly across the 460+ stocks being used (an asset universe taken from the `BSE500` market-capitalization-based index).

Considering that `bt77` has been developed using [Free Open Source Software](https://en.wikipedia.org/wiki/Free_and_open-source_software) and is published reproducibly, the author asserts that it can serve as a benchmark index for (pension) fund management.

-   Individual or other investors could use it to negotiate extraordinary performance and incentive structures with their providers of wealth-management services and products. They could also build on it in collaborative or Do-It-Yourself mode.
-   For greater societal impact, [PFRDA](https://www.pfrda.org.in/) could similarly nudge Pension Fund Managers for [Minimum Assured Returns in DC Pension Systems](https://www.pfrda.org.in/writereaddata/links/annexure%20ii%20to%20mars%20final42942760-b4e1-4483-b465-997a062dcf2f.pdf). The author is not a qualified Actuarial professional, but browsing through related published content on `www`, there seems to be an underlying belief that the Equity asset category presently permitted within National Pension System (NPS) is limited to the "market return" provided by `NIFTY50` market-capitalization-based index. Continuing that way, the fund has to continue tolerating Drawdowns ("downsides") of 30% to 50% while the upside is unlikely to be over 15% CAGR long term, with 5-10 years of sideways markets every now and then. Almost certainly then, growth would continue to be ordinary while tolerating high Drawdown (or Expected Tail Loss) risk. Instead, what [PFRDA](https://www.pfrda.org.in/) and India's Ministry of Finance could consider (if not already done so) is extraordinary growth (at possibly-lesser risk as exemplified via `bt77` investible benchmark). Creating that alternative future is consistent with Suitable and sustainable pensions. India's Minister of Finance invited inputs for the upcoming ["Budget like never before"](https://www.businesstoday.in/union-budget-2021/news/fm-nirmala-sitharaman-promises-union-budget-2021/story/427048.html) ("in a way. 100 years of India wouldn't have seen a Budget being made post-pandemic like this"). As an input, may this proposal find its way forward to transform the wealth ecosystem. Through Clawbacks or other loss-sharing provisions, incentive structures can be harmonized:
    -   Harmony can be in quantum and timing with real positive growth in wealth, e.g., as measured by Net Present Value adjusted by Time Value of Money and Risk Premium.
    -   And creating an exemplar fund-management enterprise as a [Social Business](http://www.muhammadyunus.org/index.php/social-business/social-business) could (a) deny the incentive of perpetual dividend to owners of Pension-Fund Manager institutions and (b) devote the enterprise's profits and actions solely and maximally for its societal purpose of serving adequate pensions forever. That enterprise could "own" and maintain `bt77` and other outputs of this [Project Portfolios for Pension Funds](https://github.com/yadevinit/pf4pf), as part of causing Peer production (and aggregation) of wealth-management-related cultural and information objects in the Commons. On the advantages of Commons-based Peer production, you can refer <http://www.benkler.org/CoasesPenguin.html> which says:

> I suggest that we are seeing is the broad and deep emergence of a new, third mode of production in the digitally networked environment. I call this mode "commons-based peer-production," to distinguish it from the property- and contract-based models of firms and markets. Its central characteristic is that groups of individuals successfully collaborate on large-scale projects following a diverse cluster of motivational drives and social signals, rather than either market prices or managerial commands. ... The paper also explains why this mode has systematic advantages over markets and managerial hierarchies when the object of production is information or culture, and where the capital investment necessary for production-computers and communications capabilities-is widely distributed instead of concentrated.

Portfolio Rebalancing Performance and Weights
---------------------------------------------

    ## Warning: Ensure correct files list in flist via DOS cmd: > dir /b /o-d bt* >
    ## flist.txt

    ## [1] "Presumed parts of gettingStarted.R have a Portfolio optimization rebalancing (eg bt77) before entering this code."

    ## [1] "Sat Dec 18 15:06:05 2021"

    ## [1] 1

    ## [1] "bt-rebal13opt_pfESplus-ESarg3Retarg3ES1-202112181413-202112181413.rds"

    ##         ABB  APOLLOHOSP    MINDAIND     KPRMILL     TRIDENT         IRB 
    ## 0.203946555 0.115294433 0.101403996 0.090851838 0.088324279 0.076551614 
    ##  DCMSHRIRAM    KPITTECH       WIPRO   JSWENERGY   BAJAJELEC    EMAMILTD 
    ## 0.053774985 0.053523493 0.053123377 0.038904796 0.035304618 0.033643801 
    ##       TECHM       TANLA  ADANITRANS  CHAMBLFERT 
    ## 0.029015544 0.012003527 0.011483958 0.002849185

    ## Warning in mycharts.PerformanceSummary(btouts.rets[[ix]], main =
    ## paste0("Rebalancing performance")): Rf=0, geometric=TRUE, ylog=FALSE.

![](weightsAUMinvest_files/figure-markdown_github/comboRebalv500p2-1.png)

    ## NULL

![](weightsAUMinvest_files/figure-markdown_github/comboRebalv500p2-2.png)

Translating Portfolio Weights to Share Counts
---------------------------------------------

    ## [1] "This would require the stock-wise wealth or AUM invested to be:"
    ##                          ABB APOLLOHOSP MINDAIND  KPRMILL  TRIDENT      IRB
    ## 2021-12-17 05:30:00 31561.75   17842.39 15692.78 14059.78 13668.62 11846.74
    ##                     DCMSHRIRAM KPITTECH    WIPRO JSWENERGY BAJAJELEC EMAMILTD
    ## 2021-12-17 05:30:00   8321.948 8283.028 8221.108  6020.712  5463.566 5206.546
    ##                      TECHM    TANLA ADANITRANS CHAMBLFERT
    ## 2021-12-17 05:30:00 4490.3 1857.606     1777.2   440.9256
    ## [1] "So, total AUM= 154755"
    ## [1] "Let's pray this works, and you don't face liquidity issues."
    ## [1] "Beware that market prices at Buy could still differ from these prices estimated with:"
    ##                         ABB APOLLOHOSP MINDAIND KPRMILL TRIDENT    IRB
    ## 2021-12-17 05:30:00 2199.75     4786.7  1047.75   589.4   54.85 203.95
    ##                     DCMSHRIRAM KPITTECH WIPRO JSWENERGY BAJAJELEC EMAMILTD
    ## 2021-12-17 05:30:00      994.1    481.6 670.8     286.8   1312.05   542.85
    ##                       TECHM   TANLA ADANITRANS CHAMBLFERT
    ## 2021-12-17 05:30:00 1642.85 1785.35     1777.2     386.75

    ## [1] "For share counts from last-rebalancing weights, use a multiple of the following counts. These are rounded to 5 decimals, which should suffice for AUM up to Rs 10^5L, assuming minimum basket is of AUM Rs 1L:"

    ##                          ABB APOLLOHOSP MINDAIND  KPRMILL  TRIDENT      IRB
    ## 2021-12-17 05:30:00 14.34788    3.72749 14.97759 23.85439 249.2001 58.08652
    ##                     DCMSHRIRAM KPITTECH    WIPRO JSWENERGY BAJAJELEC EMAMILTD
    ## 2021-12-17 05:30:00    8.37134 17.19898 12.25568  20.99272   4.16414  9.59113
    ##                       TECHM   TANLA ADANITRANS CHAMBLFERT
    ## 2021-12-17 05:30:00 2.73324 1.04047          1    1.14008

    ## [1] "Browse further for any deeper understanding you might wish."

(Contribution to) Expected Tail Loss of Portfolio
-------------------------------------------------

    ## [1] "opt_pfESplus-ESarg3Retarg3ES1-202112181413.rds"

    ## List of 3
    ##  $ ES            : num 0.0351
    ##  $ contribution  : Named num [1:464] 0 0 0 0 0 ...
    ##   ..- attr(*, "names")= chr [1:464] "IBULHSGFIN" "IDEA" "TATAMOTORS" "TATAPOWER" ...
    ##  $ pct_contrib_ES: Named num [1:464] 0 0 0 0 0 ...
    ##   ..- attr(*, "names")= chr [1:464] "IBULHSGFIN" "IDEA" "TATAMOTORS" "TATAPOWER" ...

    ##  Named num [1:16] 0.00583 0.00472 0.0028 0.00302 0.00469 ...
    ##  - attr(*, "names")= chr [1:16] "ABB" "APOLLOHOSP" "MINDAIND" "KPRMILL" ...

    ##  Named num [1:16] 0.166 0.1343 0.0797 0.0861 0.1335 ...
    ##  - attr(*, "names")= chr [1:16] "ABB" "APOLLOHOSP" "MINDAIND" "KPRMILL" ...

![](weightsAUMinvest_files/figure-markdown_github/riskBudgets-1.png)![](weightsAUMinvest_files/figure-markdown_github/riskBudgets-2.png)

Normality of Returns and Other Investigations
---------------------------------------------

``` r
# ref https://cran.csiro.au/web/packages/PerformanceAnalytics/vignettes/PA-charts.pdf:
assetRet <- RtrgWin[, names(pfLastWts.sortGt0)] # R[, "GRANULES"] # R
layout(rbind(c(1,2),c(3,4)))
chart.Histogram(assetRet, main = "Plain",
  sub=paste(head(colnames(assetRet)), collapse=","),
  methods = NULL)
# title(paste(head(colnames(assetRet)), collapse=","), cex.main=0.8)
chart.Histogram(assetRet, main = "Density", breaks=40, methods = c("add.density", "add.normal"))
chart.Histogram(assetRet, main = "Skew and Kurt", methods = c("add.centered", "add.rug"))
chart.Histogram(assetRet, main = "Risk Measures", methods = c("add.risk"))
```

![](weightsAUMinvest_files/figure-markdown_github/normalityReturns-1.png)

``` r
layout(c(1,1))
if(! exists("Rf")){ Rf <- 0.08/52 } # else continue. Presuming long-term inflation at 8% CAGR.
if(! exists("Reqwt.rebalQtr")){
  Reqwt.rebalQtr <- Return.portfolio(R, rebalance_on="quarters") # rowSums(R) /
} # else continue
chart.RiskReturnScatter(assetRet, Rf=Rf, main="Risk-Reward tradeoff in trailing training window")
```

![](weightsAUMinvest_files/figure-markdown_github/normalityReturns-2.png)

``` r
  # sharpe.ratio = NULL, add.names = NULL. Beware: Risk =/= ES component!
  # It might be StDev or ES univariate/single marginal.
charts.RollingRegression(Ra=R[, names(pfLastWts.sortGt0)], Rb=Reqwt.rebalQtr, width=52, Rf=Rf,
  colorset=colorset.my, lwd=2)
```

![](weightsAUMinvest_files/figure-markdown_github/normalityReturns-3.png)

Appendix: Runtime Environment
=============================

Here's the runtime environment used. It's reported here for reproducibility:

``` r
sessionInfo() # Sys.info()[['sysname']]
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
    ## [1] parallel  stats     graphics  grDevices utils     datasets  methods  
    ## [8] base     
    ## 
    ## other attached packages:
    ##  [1] unikn_0.3.0                PortfolioAnalytics_1.1.0  
    ##  [3] PerformanceAnalytics_2.0.4 foreach_1.4.8             
    ##  [5] xts_0.12.1                 zoo_1.8-7                 
    ##  [7] ROI.plugin.quadprog_1.0-0  ROI.plugin.glpk_1.0-0     
    ##  [9] ROI_1.0-0                  DEoptim_2.2-5             
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_1.0.3          pillar_1.4.3        compiler_3.6.2     
    ##  [4] iterators_1.0.12    tools_3.6.2         digest_0.6.24      
    ##  [7] tibble_2.1.3        evaluate_0.14       lifecycle_0.1.0    
    ## [10] gtable_0.3.0        lattice_0.20-38     pkgconfig_2.0.3    
    ## [13] rlang_0.4.4         registry_0.5-1      Rglpk_0.6-4        
    ## [16] yaml_2.2.1          xfun_0.19           dplyr_0.8.3        
    ## [19] stringr_1.4.0       knitr_1.27          tidyselect_0.2.5   
    ## [22] grid_3.6.2          glue_1.3.1          R6_2.4.1           
    ## [25] rmarkdown_2.1       purrr_0.3.3         ggplot2_3.3.2      
    ## [28] magrittr_1.5        scales_1.1.0        codetools_0.2-16   
    ## [31] htmltools_0.4.0     assertthat_0.2.1    colorspace_1.4-1   
    ## [34] numDeriv_2016.8-1.1 quadprog_1.5-8      stringi_1.4.6      
    ## [37] munsell_0.5.0       slam_0.1-47         crayon_1.3.4
