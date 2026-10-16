# Portfolio Risk Management Research Paper
## This is being re-writen - I just wanted to update it to a good enough state. 
## Disclaimer

Informational and educational purposes only. Nothing in this paper or its accompanying repository, including its VaR/CVaR output, Kelly sizing suggestions, or any other result, is a recommendation, offer, or solicitation to buy, sell, or hold any security, and none of it should be construed as investment, financial, legal, or tax advice. This is a personal project, not a product of a registered investment adviser, broker-dealer, or fiduciary. The code and its output are provided “AS IS,” without warranty of any kind: simulated results depend entirely on model assumptions and calibration choices, may contain errors, and are not guaranteed to reflect real-world outcomes, past or future. Anyone using this code is solely responsible for their own investment decisions and should consult a licensed financial professional before acting on anything produced by it.

## Abstract

Value-oriented retail investors often lack rigorous, quantitative tools for estimating portfolio-level risk, let alone sizing new positions optimally. This paper presents a Monte Carlo risk-modeling framework that combines jump-diffusion, stochastic volatility, and Student's-$t$ distributed returns to estimate Value at Risk (VaR) and Conditional Value at Risk (CVaR) across multiple confidence levels and time horizons. Building on this framework, a multi-asset, continuous-time extension of the Kelly criterion is applied to determine the optimal capital allocation for a candidate position, incorporating its correlation to the existing portfolio. Together, these components form a framework for reconciling quantitative risk modeling with fundamentals-driven investment decisions.

Value investing is defined as “an investment strategy that involves buying assets that appear underpriced relative to their intrinsic value” (Hayes, n.d.). While that is theoretically sound, in practice you can suffer from long mark-to-market losses. The same behaviors that create assets that trade away from their intrinsic value, like macroeconomic stress or sector shifts, are usually the same set of behaviors that continue to make the asset suffer. Value investors like myself watch their portfolios erode on a daily basis for weeks or months at a time.

Discretionary conviction is not a sufficient risk-management strategy. To successfully endure the psychologically demanding wait, investors require peace of mind. Without a rigorous mathematical model and understanding of potential maximum drawdowns, a 20% loss can feel like a wrong decision rather than an expected outcome the model already accounted for.

This project started when I wanted a way to have peace of mind and estimate how much money I could lose with a given confidence level over the next 10 days as well as over the next year (252 trading days). Standard risk tools work poorly for this problem. Traditional parametric VaR models assume asset returns follow a normal distribution and that historical volatility is a reliable guide for the future. Both assumptions break when you need them the most.During rough times, the stock market experiences fat-tailed shocks: moves beyond 4 standard deviations, which a normal distribution says should almost never happen. The stock market's volatility went past 4 standard deviations approximately every 2 years (Yadav, 2025). Relying on basic calculations can leave investors blindsided with exponential risk.

I wanted to stress-test my portfolio to see how much risk I'm actually taking. Value at Risk (VaR) at 95% confidence is the loss that should be exceeded only 5% of the time over a chosen horizon. Conditional Value at Risk (CVaR) is the average loss in those worst 5% of cases (Rockafellar & Uryasev, 2000). However, this doesn't help with new positions. That's why I added Kelly Criterion to find the optimal amount of an asset to buy for long term geometric growth.

Kelly Criterion is defined as (Kelly, 1956; Thorp, 2006):

$$
f^{\ast} = \frac{\mu}{\sigma^{2}}
$$

Where:

- $f^{\ast}$: The fraction of capital to put into the asset
- $\mu$: The asset's expected return in excess of the risk-free rate
- $\sigma^{2}$: The asset's return variance

You may be wondering why a value investor cares about optimal risk sizing if the goal is to buy stocks when the underlying stock is below intrinsic value. Charlie Munger describes a similar philosophy to what I'm designing: “And then all that is required is a willingness to bet heavily when the odds are extremely favorable, using resources available as a result of prudence and patience in the past” (Munger, 2023). The difference here is that value investing traditionally relies on qualitative judgment which I find is very subjective. That's why I like more quantitative metrics. My design uses Multi-Asset Continuous-Time Kelly Criterion to find the optimal amount of an asset to buy. The single-asset formula above only works when sizing one bet in isolation. I want to know how much of a new candidate to add to my portfolio, given how that candidate moves relative to the portfolio. Losses and gains aren't symmetric: losing 50% of your capital requires a 100% gain just to recover it, and the bigger the loss, the worse it becomes. Because of this, my design uses 'Half-Kelly,' taking half the output from the formula (Thorp, 2006).

Multi-Asset Continuous-Time Kelly Criterion is defined as (Merton, 1969):

$$
f^{\ast} = \Sigma^{-1} \cdot \mu
$$

Where:

- $f^{\ast} = [f_p^{\ast}, f_c^{\ast}]$: The optimal capital sizing of the two assets
- $\mu = [\mu_p, \mu_c]$: Expected annual excess return of each asset

Covariance Matrix:

$$
\Sigma =
\begin{array}{c|cc}
 & \text{Portfolio} & \text{Candidate Equity} \\
\hline
\text{Portfolio} & \sigma_p^{2} & \rho\,\sigma_p\,\sigma_c \\
\text{Candidate Equity} & \rho\,\sigma_p\,\sigma_c & \sigma_c^{2}
\end{array}
$$

Variables are as follows:

- $\mu_p$: Existing portfolio's expected annual excess return
- $\sigma_p$: Existing portfolio's annual volatility
- $\mu_c$: Candidate equity's expected annual excess return
- $\sigma_c$: Candidate equity's annual volatility
- $\rho$: Correlation between the portfolio's returns and the candidate's returns
- $f_c^{\ast}$: Fraction of total capital to put into the candidate to maximize long-term geometric growth
- $f_p^{\ast}$: Fraction of total capital to allocate to the existing portfolio

Before diving deeper, it's important to start from the simplest possible model of simulating a stock's price: Geometric Brownian Motion. In simple language, it treats a stock's price like a biased random walk. It tends to drift in one direction over time, similar to compounding interest, but at every step it also gets pushed randomly up or down. If visualized, the graph should resemble a zigzag, whether the overall trend is heading in the positive or negative direction.

Geometric Brownian Motion is defined as (Samuelson, 1965):

$$
dS_t = \mu S_t\,dt + \sigma S_t\,dW_t
$$

Where:

- $S_t$: Asset's price at time $t$
- $\mu$: Drift (expected return)
- $\sigma$: Volatility
- $dW_t$: A random push at each instant, equally likely to push the price up or down, with no memory of what happened previously

The standard equation has some limitations regarding its parameters. Both $\mu$ and $\sigma$ are treated as fixed constants for the entire model, but that's not realistic for the market.Volatility especially tends to cluster: calm days are usually followed by more calm days, and turbulent days by more turbulent days. An unchanging $\sigma$ can't capture that. In real markets stocks experience jumps that a continuous process can't produce and actual returns have fatter tails than Gaussian distribution. My design is aimed to fix these problems. Volatility is no longer a constant, Based on Heston's stochastic volatility model, (Heston, 1993) it follows its own random process that rises and falls over time. based on Merton's jump-diffusion model, (Merton, 1976) sudden random jumps are added on top of the normal day-to-day movement. Since the market regularly has had standard deviation above $4\sigma$, I use a Student's-$t$ distribution (Blattberg & Gonedes, 1974), which naturally produces fat tails to better replicate it. The design I use is basically a Bate's model (Bates, 1996) but with a Student's $t$ distribution. Of these two parameters, only $\sigma$ is made dynamic in this design. $\mu$ remains a single calibrated constant for the full simulation. This was a choice I made because it's difficult to reliably estimate drift. Volatility on the other hand, can be estimated with higher accuracy in short term windows.

Geometric Brownian Motion with the 3 parameters added is defined as:

$$
\frac{dS_{i,t}}{S_{i,t}} = \mu_i\,dt + \sqrt{v_{i,t}}\,dW_{i,t} + dJ_{i,t}
$$

Where:

- $i$: An index identifying a specific asset in the portfolio
- $dS_{i,t}$: Change in asset $i$'s price at time $t$
- $S_{i,t}$: Asset $i$'s price at time $t$
- $\mu_i$: Asset $i$'s drift (expected return)
- $dt$: Small increment of time
- $v_{i,t}$: Asset $i$'s instantaneous variance at time $t$, from the Heston process defined above
- $dW_{i,t}$: Student's-$t$ distributed random shock, correlated across assets
- $dJ_{i,t}$: Jump contribution at time $t$, from the jump process defined above

Heston's model gives each stock its own variance, which moves over time:

$$
dv_{i,t} = \kappa_i(\theta_i - v_{i,t})\,dt + \xi_i \sqrt{v_{i,t}}\,dZ_{i,t}
$$

Where:

- $v_{i,t}$: Asset $i$'s variance at time $t$
- $\kappa_i$: How quickly variance is pulled back toward its long-run average
- $\theta_i$: The long-run average variance
- $\xi_i$: How much variance itself bounces around
- $dZ_{i,t}$: A random shock to variance, correlated with the price shock $dW_{i,t}$ by $\rho_i$

Merton's jump-diffusion adds sudden jumps:

$$
dJ_{i,t} = (e^{Y} - 1)\,dN_{i,t}
$$

Where:

- $N_{i,t}$: A counter of jumps, where the chance of a jump in any small time step depends on the jump rate $\lambda$
- $Y$: The size of one jump, drawn from a normal distribution with mean $\mu_j$ and standard deviation $\sigma_j$

My design uses two kinds of jumps. A stock-specific jump hits one ticker. A market-wide jump hits every holding on the same day. To run this simulation, the model needs six inputs, all inputs are measured from 5 years of daily price history. Jumps are separated out first: any day whose return exceeds three of that stock's own standard deviations counts as a jump, and every other day is ordinary. From the ordinary days, I measure each stock's annualized average growth ($\mu$) and daily volatility ($\sigma$), where $\sigma$ is used in the Kelly sizing step. From the jump days, I measure the jump rate ($\lambda$) per year and, pooling across all tickers to get enough data, the average and spread of jump size. A jump counts as market-wide if at least 20% of the tickers jumped on the same day and never fewer than two.

To run the simulation, the model needs six inputs which are all measured from 5 years of daily price history. Jumps are separated first: any day whose return exceeds three of that stock's own standard deviations counts as a jump, and every other day is ordinary. From the ordinary days, I measure each stock's annualized average growth and daily volatility, where $\sigma$ is used in the Kelly sizing step. From the jump days, I measure the jump rate per year and, pooling across all tickers to get enough data, the average and spread of jump size. A jump counts as market-wide if at least 20% of the tickers jumped on the same day, and never fewer than two. To capture time-varying volatility, I compute realized variance over a rolling window of 20 ordinary trading days and fit an autoregressive relationship between consecutive windows. This yields the mean-reversion speed, the long-run variance level, the volatility of variance, and the correlation between price and variance movements. Cross-asset dependence is captured by two correlation matrices: a calm-regime matrix estimated from the trailing 90 days, and a stress-regime matrix estimated from only the most volatile 25% of days in the 5-year history. During simulation, each path interpolates between the two according to how far its own simulated variance exceeds its long-run level. Finally, I model fat tails with a Student's-$t$ distribution. Its degrees of freedom ($\nu$, which controls how often extreme moves occur relative to ordinary ones) is fit to the everyday returns of all stocks combined, after each stock's returns are rescaled to the same average and spread. A lower $\nu$ means heavier tails and more frequent extreme moves.

With the six inputs measured, the model simulates each stock's price forward one trading day at a time across 500,000 independent paths. Each day, every path is updated in five steps. First, the calm and stress correlation matrices are blended according to how elevated that path's own volatility is. Second, correlated random shocks are drawn for every stock, with a single shared fat-tail draw per path per day so that extreme days tend to hit several stocks together. Third, jumps are drawn for both the specific stock and market-wide. Fourth, each price is updated using drift, the volatility-scaled shock, and jumps if applicable. The drift adjusted so both volatility and jumps do not add extra expected returns. Lastly, each stock's variance is gated to never fall below zero.

The simulation is run for two horizons, 10 trading days and 252 trading days (One year). It's repeated across 20 independent seeds of 500,000 paths each (10,000,000 total paths).

## Sources

Bates, D. S. (1996). Jumps and stochastic volatility: Exchange rate processes implicit in Deutsche mark options. *The Review of Financial Studies*, *9*(1), 69–107. https://doi.org/10.1093/rfs/9.1.69

Blattberg, R. C., & Gonedes, N. J. (1974). A comparison of the stable and Student distributions as statistical models for stock prices. *The Journal of Business*, *47*(2), 244–280.

Hayes, A. ([Year, Month Day]). Value investing definition, how it works, strategies, and risks. *Investopedia*. https://www.investopedia.com/terms/v/valueinvesting.asp

Heston, S. L. (1993). A closed-form solution for options with stochastic volatility with applications to bond and currency options. *The Review of Financial Studies*, *6*(2), 327–343.

Kelly, J. L., Jr. (1956). A new interpretation of information rate. *Bell System Technical Journal*, *35*(4), 917–926. https://doi.org/10.1002/j.1538-7305.1956.tb03809.x

Merton, R. C. (1969). Lifetime portfolio selection under uncertainty: The continuous-time case. *The Review of Economics and Statistics*, *51*(3), 247–257. https://doi.org/10.2307/1926560

Merton, R. C. (1976). Option pricing when underlying stock returns are discontinuous. *Journal of Financial Economics*, *3*(1–2), 125–144.

Munger, C. T. (2023). *Poor Charlie's almanack: The essential wit and wisdom of Charles T. Munger* (P. D. Kaufman, Ed.). Stripe Press. (Original work published 2005)

Rockafellar, R. T., & Uryasev, S. (2000). Optimization of conditional value-at-risk. *Journal of Risk*, *2*(3), 21–41.

Samuelson, P. A. (1965). Rational theory of warrant pricing. *Industrial Management Review*, *6*(2), 13–39.

Thorp, E. O. (2006). The Kelly criterion in blackjack, sports betting, and the stock market. In S. A. Zenios & W. T. Ziemba (Eds.), *Handbook of asset and liability management: Vol. 1* (pp. 385–428). Elsevier.

Yadav, P. (2025, April 12). *Fat tails in financial returns explained*. Risk Hub. https://riskhub.org/blogs/fat-tails-in-financial-returns-explained
