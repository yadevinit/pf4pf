Risk-free Nay Ratna
================

What's on Menu for this Research Viewpoint
------------------------------------------

1.  In NayRatna, optimal portfolios are located at very different positions on the risk-return chart for the Efficient Frontier. While different optimization approaches (including objectives, constraints, and rebalancing) can differ in their outputs, let's check, e.g., whether one approach is tilting to minimize risk while the other is inclined to maximizing return at the same optimization time.
2.  `Rf=0` in NayRatna. Let's include time series of returns of `Rf` say 10-year Govt Securities.
3.  NayRatna showed a portfolio with Annualized Return of `0.40`, which is the same as saying 40% CAGR. Let's include a Target Return of 36% CAGR. With such an absolute benchmark, the (Sharpe-recommended) InformationRatio could be used to assess the goodness of competing approaches.

Positions on Risk-Return Chart
------------------------------

Constraint of `full_investment` on weight sum and `long_only` on boxes were specified. The objective was specified as maximizing `mean` return while also minimizing risk `ES` with `risk_aversion=0.25`. [Portfolio Optimization with ROI in PortfolioAnalytics](https://cran.r-project.org/web/packages/PortfolioAnalytics/vignettes/ROI_vignette.pdf) says `risk_aversion` parameter controls how much portfolio variance is penalized. So, we would expect increasing this parameter would locate the optimal portfolio to a lesser-risk position. But nay, that does not happen. So either the way we specify to the ROI solver was not ok, or the ROI solver does not support that. Ah, the music's stopped. We have to understand this deeper before we can rely on this fuller. Beyond Constraints and Objectives, it looks like there are choices that affect robustness: Estimator (of covariances, etc.) and Solver. And there's something unnerving about diversification disappearing during extreme market events: you might have seen herds of stocks move down together during 2008! ["The Black Swan: The Impact of the Highly Improbable"](https://en.wikipedia.org/wiki/The_Black_Swan:_The_Impact_of_the_Highly_Improbable) and [Fooled by Randomness](https://en.wikipedia.org/wiki/Fooled_by_Randomness) might have crossed your mind by now. Couples passing soon after one another is studied and used in actuarial science (to model longevity for insurance). <https://en.m.wikipedia.org/wiki/Copula_(linguistics)> and <https://www.wired.com/2009/02/wp-quant/amp> give other interpretations. And there's [Emergence](https://en.wikipedia.org/wiki/Emergence). Coming back to Portfolios, it seems you can estimate copulae-based bivariate tail risk, which seems useful for Portfolios constrained by tail-risk budgets. And coming back to present times, we have, or rather, the stock markets have traced a "V", like the "W" of 2008; so, this trailing year would likely show that "going down together as a herd or couple" syndrome. Bear in mind that in the world of (economics and) finance, matters can be far-more speculative than in the world of (science and) engineering; (co-) variance here is unlikely to be constant, and so, you hear of heteroscedasticity, volatility clustering, and regime shifts.

In the `R` source code for ROI approach, changing the risk measure to `var` instead of `ES` and setting `risk_aversion=25` gives us a mean return close to what the Random portfolios approach gave us with `ES`. It also chose as optimal a higher weight for \[BPCL\], instead of mostly for \[TSLA\]. These can be seen in following session output:

``` r
> var_obj <- portfolio_risk_objective(name="var", # Was: ES",
+     risk_aversion=25.0) # Was: 0.25
>   # Combine the objectives into a list
> qu_obj <- list(ret_obj, var_obj)
> opt_qu <- optimize.portfolio(R=returns, portfolio=init_portf, constraints=qu_constr,
+     objectives=qu_obj, optimize_method="ROI", # maxSTARR=TRUE,
+     trace=TRUE)
> extractStats(opt_qu)
       mean      StdDev         out      w.BPCL      w.NBCC w.QUICKHEAL      w.TSLA 
0.005189066 0.040270378 0.035353517 0.518196010 0.096401713 0.164998346 0.220403931 
> var_obj <- portfolio_risk_objective(name="var", # Was: ES",
+     risk_aversion=0.25) # Was: 0.25
> qu_obj <- list(ret_obj, var_obj)
> opt_qu <- optimize.portfolio(R=returns, portfolio=init_portf, constraints=qu_constr,
+     objectives=qu_obj, optimize_method="ROI", # maxSTARR=TRUE,
+     trace=TRUE)
> extractStats(opt_qu)
         mean        StdDev           out        w.BPCL        w.NBCC   w.QUICKHEAL 
 1.409758e-02  8.152840e-02 -1.243586e-02 -1.110223e-16  1.218326e-16  0.000000e+00 
       w.TSLA 
 1.000000e+00 
```

So, what's it that we missed, including that `risk_aversion`?
