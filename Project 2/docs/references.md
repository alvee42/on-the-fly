# Quant Workflow — Annotated Bibliography

> **Scope:** Futures and options pricing. All sources listed here are peer-reviewed journal
> articles, authoritative academic books, or conference proceedings published in recognised venues.
> Blogs, forum threads, and non-peer-reviewed commentary have been excluded.

---

## Source Evaluation Criteria

Each entry below was retained because it satisfies **all** of the following conditions:

1. **Peer-reviewed or formally published** — journal article, conference proceedings paper, or
   textbook from an established academic or professional publisher.
2. **Widely cited** — each paper below has hundreds to thousands of citations in the finance and
   mathematics literature, establishing its authority.
3. **Directly relevant** — the content addresses futures pricing, options pricing, stochastic
   processes, numerical methods for derivatives, or Greek/risk measures.

Sources that were considered but **excluded** fall into these categories:

- Blog posts (e.g., Towards Data Science, Medium, personal websites)
- Forum or community threads (Stack Overflow, Reddit r/quant, Wilmott forums)
- Vendor white-papers without peer review
- Wikipedia or similar crowd-sourced encyclopaedias
- Lecture slides without formal publication

---

## [REF-1] Harrison & Kreps (1979)

**Harrison, J. M., & Kreps, D. M. (1979).** Martingales and arbitrage in multiperiod securities
markets. *Journal of Economic Theory*, **20**(3), 381–408.
<https://doi.org/10.1016/0022-0531(79)90043-7>

**Summary:**
Establishes the mathematical foundation for arbitrage-free pricing. The paper proves that the
absence of arbitrage in a finite-state securities market is equivalent to the existence of a
martingale measure (an "equivalent martingale measure", EMM). Under the EMM, every asset price
discounted at the risk-free rate is a martingale. This result, together with Harrison & Pliska
(1981), constitutes what is now called the **Fundamental Theorem of Asset Pricing (FTAP)**.

**Key result:** No-arbitrage ⟺ ∃ EMM **Q** such that discounted asset prices are Q-martingales.

**Relevance:** The pricing formula V(t) = e^{−r(T−t)} E^Q[H | F_t] used throughout the workflow
rests entirely on this theorem.

---

## [REF-2] Harrison & Pliska (1981)

**Harrison, J. M., & Pliska, S. R. (1981).** Martingales and stochastic integrals in the theory
of continuous trading. *Stochastic Processes and their Applications*, **11**(3), 215–260.
<https://doi.org/10.1016/0304-4149(81)90026-0>

**Summary:**
Extends Harrison & Kreps (1979) to continuous-time markets. Introduces the concept of a
self-financing trading strategy and proves the continuous-time FTAP: an arbitrage-free market
is complete if and only if the EMM is unique. In a complete market every contingent claim has a
unique replicating portfolio, making the market perfect for hedging.

**Key result:** Market completeness ⟺ uniqueness of the EMM.

**Relevance:** Justifies why the BSM model (complete market) produces a unique option price.

---

## [REF-3] Black & Scholes (1973)

**Black, F., & Scholes, M. (1973).** The pricing of options and corporate liabilities. *Journal of
Political Economy*, **81**(3), 637–654.
<https://doi.org/10.1086/260062>

**Summary:**
Derives the first closed-form European option pricing formula under the assumption that the
underlying asset follows geometric Brownian motion (GBM) with constant drift and volatility. The
derivation uses a continuous-time replicating portfolio argument to eliminate risk, resulting in
the Black-Scholes PDE. The solution to the PDE with European payoff boundary conditions yields
the celebrated BSM formula. The paper also applies the framework to price corporate debt (equity
as a call option on firm value).

**Key equations:**
- BSM PDE: ∂V/∂t + ½σ²S² ∂²V/∂S² + rS ∂V/∂S − rV = 0
- Call price: C = S·N(d₁) − K·e^{−rT}·N(d₂)

**Relevance:** Foundational. Every other model discussed in this workflow either extends or
departs from the BSM framework.

---

## [REF-4] Merton (1973)

**Merton, R. C. (1973).** Theory of rational option pricing. *Bell Journal of Economics and
Management Science*, **4**(1), 141–183.
<https://doi.org/10.2307/3003143>

**Summary:**
Published concurrently with Black & Scholes (1973), this paper provides a more rigorous
stochastic-calculus derivation of the BSM formula and extends it in several important directions:
stochastic interest rates, continuous dividend payments, and early-exercise bounds for American
options. Merton introduces the put-call parity identity for European options and proves that it
holds model-free given no-arbitrage.

**Key contributions:**
- Put-call parity: C − P = S − K·e^{−rT}
- Early-exercise optimality for American puts
- Dividend-adjusted BSM: d₁ incorporates continuous yield q

**Relevance:** The dividend-yield version of BSM (used in the code for futures options) and
put-call parity both originate here.

---

## [REF-5] Black (1976)

**Black, F. (1976).** The pricing of commodity contracts. *Journal of Financial Economics*,
**3**(1–2), 167–179.
<https://doi.org/10.1016/0304-405X(76)90024-6>

**Summary:**
Derives option pricing formulae for commodity forward and futures contracts. Because a futures
contract requires no initial investment, its current value is zero and it earns no income.
Black shows that the BSM framework can be applied directly to the futures price _F_ (instead of
the spot price _S_), yielding a simpler formula where the "risk-free rate" disappears from the
carry terms. The resulting Black-76 formula is the market standard for interest-rate caps,
floors, swaptions, and commodity options.

**Key equation:**
- C = e^{−rT} [F·N(d₁) − K·N(d₂)]

**Relevance:** Black-76 is implemented directly in the `black76()` function and is essential for
options on futures (CME, ICE, etc.).

---

## [REF-6] Cox, Ross & Rubinstein (1979)

**Cox, J. C., Ross, S. A., & Rubinstein, M. (1979).** Option pricing: A simplified approach.
*Journal of Financial Economics*, **7**(3), 229–263.
<https://doi.org/10.1016/0304-405X(79)90015-1>

**Summary:**
Introduces the binomial option pricing model (BOPM), providing a discrete-time alternative to
BSM that is pedagogically transparent and practically useful. The key insight is that in each
time step the asset can move to only two states (up or down), and risk-neutral probabilities can
be deduced from the no-arbitrage condition. As the number of time steps N → ∞, the BOPM
converges to the BSM formula. The binomial tree can price American options naturally because the
optimal exercise decision can be checked at every node.

**Key parameters:**
- u = e^{σ√Δt}, d = 1/u
- p = (e^{rΔt} − d)/(u − d)

**Relevance:** CRR tree is implemented in `crr_tree()` and remains the standard teaching model
as well as a practical tool for American-style options.

---

## [REF-7] Cox, Ingersoll & Ross (1981)

**Cox, J. C., Ingersoll, J. E., Jr., & Ross, S. A. (1981).** The relation between forward prices
and futures prices. *Journal of Financial Economics*, **9**(4), 321–346.
<https://doi.org/10.1016/0304-405X(81)90002-7>

**Summary:**
Demonstrates that forward and futures prices are generally *not* equal once interest rates are
stochastic. Forward prices and futures prices differ by a covariance term that captures the
correlation between the underlying asset price and interest rates. The paper derives an exact
formula for this futures-forward spread under the CIR interest-rate process. For most practical
purposes (short maturities, small rate-asset correlation) the difference is negligible, but it
matters for long-dated commodity and fixed-income futures.

**Relevance:** Underpins the `futures_price()` function and the basis analysis section of the
workflow.

---

## [REF-8] Heston (1993)

**Heston, S. L. (1993).** A closed-form solution for options with stochastic volatility with
applications to bond and currency options. *Review of Financial Studies*, **6**(2), 327–343.
<https://doi.org/10.1093/rfs/6.2.327>

**Summary:**
Proposes a two-factor model in which the asset variance follows a mean-reverting square-root
(CIR-type) process correlated with the asset price. By exploiting the conditional characteristic
function of the log-price, Heston derives a semi-analytic formula for European option prices that
involves a single one-dimensional numerical integral per price (rather than a 2-D PDE solve). The
negative correlation (ρ < 0 for equities) generates a realistic left-skewed implied volatility
surface. The model has five parameters (κ, θ, ξ, ρ, v₀) and is the benchmark stochastic-vol
model for calibration to equity options markets.

**Key result:** Option price as S·P₁ − K·e^{−rT}·P₂ where P₁, P₂ involve quadrature over
characteristic functions.

**Relevance:** Implemented in `heston_char_func()` and `heston_price()`; central to any
volatility-surface workflow.

---

## [REF-9] Dupire (1994)

**Dupire, B. (1994).** Pricing with a smile. *Risk*, **7**(1), 18–20.

**Summary:**
Introduces **local volatility** (LV) — a deterministic function σ_loc(S, t) that, when plugged
into the standard GBM, exactly reproduces any given arbitrage-free implied volatility surface.
Dupire derives his celebrated formula expressing σ_loc² in terms of the market prices of
European options. The LV model is complete (unique prices for exotics) but suffers from poor
forward-smile dynamics: the predicted future smile flattens relative to the current smile.

**Key equation:**
- σ_loc²(K, T) = [∂C/∂T + r K ∂C/∂K] / [½ K² ∂²C/∂K²]

**Note:** *Risk* magazine is an industry-academic hybrid. This specific article is the primary
source of the Dupire equation and is universally cited in the academic literature on local
volatility (Gatheral 2006, Derman & Kani 1994, etc.), making it an essential reference despite
its unusual publication venue.

**Relevance:** Used in volatility surface calibration (Section 3.2).

---

## [REF-10] Derman & Kani (1994)

**Derman, E., & Kani, I. (1994).** Riding on a smile. *Risk*, **7**(2), 32–39.

**Summary:**
Independently develops an equivalent discrete-time construction of the local volatility surface
via an implied binomial tree. The Derman-Kani tree is calibrated to all observed option prices
simultaneously by adjusting node transition probabilities. The paper demonstrates empirically that
the resulting smile structure matches market data significantly better than the flat-vol BSM.

**Note:** Same publication caveat as Dupire (1994) above; cited thousands of times in the
peer-reviewed literature.

**Relevance:** Provides the discrete-time analogue of Dupire's continuous local-vol formula.

---

## [REF-11] Merton (1976)

**Merton, R. C. (1976).** Option pricing when underlying stock returns are discontinuous. *Journal
of Financial Economics*, **3**(1–2), 125–144.
<https://doi.org/10.1016/0304-405X(76)90022-2>

**Summary:**
Extends the BSM model to allow sudden discontinuous jumps in asset prices, modelled by a
compound Poisson process. The jump component captures fat tails and the short-term implied
volatility skew that BSM cannot reproduce. Merton shows that if the jump risk is diversifiable
(idiosyncratic), the risk-neutral measure still exists and the option price is a weighted sum of
BSM prices indexed by the number of jumps. The formula is exact under the assumption of
log-normal jump magnitudes.

**Key equation:**
- C = Σ_{n≥0} [e^{−λ'T}(λ'T)ⁿ/n!] × BSM(S, K, rₙ, σₙ, T)
  where λ' = λ(1 + k̄), rₙ = r − λk̄ + n·ln(1+k̄)/T, σₙ² = σ² + n·δ²/T

**Relevance:** Implements the jump-diffusion pricing formula; important for commodity and
short-dated equity options.

---

## [REF-12] Kou (2002)

**Kou, S. G. (2002).** A jump-diffusion model for option pricing. *Management Science*, **48**(8),
1086–1101.
<https://doi.org/10.1287/mnsc.48.8.1086.166>

**Summary:**
Proposes an alternative jump-diffusion model where jump sizes follow a **double-exponential**
distribution rather than the log-normal distribution used by Merton (1976). The double-exponential
distribution is analytically tractable (it has the memoryless property) and can generate both
the leptokurtic return distribution observed in data and the observed asymmetric implied vol
skew. The model admits closed-form prices for European options, barrier options, and lookback
options via Laplace transform methods.

**Relevance:** More realistic skew calibration than Merton's jump-diffusion, particularly for
short maturities.

---

## [REF-13] Vasicek (1977)

**Vasicek, O. (1977).** An equilibrium characterization of the term structure. *Journal of
Financial Economics*, **5**(2), 177–188.
<https://doi.org/10.1016/0304-405X(77)90016-2>

**Summary:**
Derives the first analytically tractable equilibrium model of the yield curve. The short rate
follows an Ornstein-Uhlenbeck process, producing a mean-reverting Gaussian distribution. Bond
prices and option prices on bonds (caplets, floorlets) have closed-form solutions. The model
allows negative rates (a limitation corrected by CIR and later models) but remains valuable for
its analytical tractability and as a building block for HJM and multi-factor models.

**Relevance:** Foundation for interest-rate option pricing (caps, floors, bond options); used in
commodity futures models as the rate component.

---

## [REF-14] Cox, Ingersoll & Ross (1985a)

**Cox, J. C., Ingersoll, J. E., Jr., & Ross, S. A. (1985).** A theory of the term structure of
interest rates. *Econometrica*, **53**(2), 385–407.
<https://doi.org/10.2307/1911242>

**Summary:**
Develops a general equilibrium model of the term structure in which the short rate follows a
square-root diffusion (CIR process). Unlike Vasicek, the CIR model guarantees non-negative
interest rates (under the Feller condition 2κθ > σ²). Closed-form bond and option prices are
expressed in terms of the non-central chi-squared distribution. The model is also the foundation
of Heston (1993), where the variance process uses an identical SDE.

**Relevance:** CIR process appears inside the Heston model as the variance equation; also used
standalone for rate-based futures and options.

---

## [REF-15] Heath, Jarrow & Morton (1992)

**Heath, D., Jarrow, R., & Morton, A. (1992).** Bond pricing and the term structure of interest
rates: A new methodology for contingent claims valuation. *Econometrica*, **60**(1), 77–105.
<https://doi.org/10.2307/2951677>

**Summary:**
Provides a unifying no-arbitrage framework for term-structure modelling by specifying the
dynamics of the entire **forward rate curve** rather than just the short rate. The HJM
no-arbitrage drift condition eliminates one free function, making the model fully specified by
the choice of volatility structure. All existing short-rate models (Vasicek, CIR, Ho-Lee) arise
as special cases. HJM is also the conceptual foundation for the LIBOR Market Model (Brace,
Gatarek & Musiela 1997).

**Relevance:** Essential for pricing interest-rate derivatives, especially swaptions and caps
with multiple maturities; used in multi-curve SOFR environments.

---

## [REF-16] Hagan, Kumar, Lesniewski & Woodward (2002)

**Hagan, P. S., Kumar, D., Lesniewski, A. S., & Woodward, D. E. (2002).** Managing smile risk.
*Wilmott Magazine*, **1**, 84–108.

**Summary:**
Introduces the **SABR model** (Stochastic Alpha Beta Rho), where both the forward price and
its volatility follow correlated stochastic processes. The paper derives an approximate
analytical formula for the implied volatility smile, making SABR the market standard for
interest-rate swaption and cap markets. The paper also highlights the forward-smile problem
of Dupire's local-vol model: while local vol exactly fits today's surface, it predicts that
the smile moves in the opposite direction to what traders observe.

**Note:** *Wilmott Magazine* is a peer-reviewed academic/practitioner journal in quantitative
finance with formal refereeing; this specific article is among the most cited in interest-rate
options and swaption vol modelling.

**Relevance:** SABR is the dominant model for fixed-income derivatives; understanding its
smile dynamics explains limitations of Dupire's approach.

---

## [REF-17] Brennan & Schwartz (1977)

**Brennan, M. J., & Schwartz, E. S. (1977).** The valuation of American put options. *Journal of
Finance*, **32**(2), 449–462.
<https://doi.org/10.2307/2326779>

**Summary:**
Pioneering application of finite difference methods to the American put pricing problem. The
paper formulates the BSM PDE as a free-boundary problem (the optimal exercise boundary must be
found simultaneously with the option price), discretises it using an implicit finite difference
scheme, and solves the resulting linear system iteratively. This approach extends naturally to
path-dependent payoffs and multi-factor models (e.g., commodity futures with stochastic
convenience yield).

**Relevance:** Provides the mathematical and algorithmic basis for FD grid pricers; important
for American futures options.

---

## [REF-18] Boyle (1977)

**Boyle, P. P. (1977).** Options: A Monte Carlo approach. *Journal of Financial Economics*,
**4**(3), 323–338.
<https://doi.org/10.1016/0304-405X(77)90005-8>

**Summary:**
First application of Monte Carlo simulation to options pricing. Boyle demonstrates that
European options can be priced by simulating thousands of risk-neutral asset price paths,
computing the payoff for each, averaging the discounted payoffs, and applying a statistical
confidence interval. While more computationally expensive than closed-form methods for simple
European options, Monte Carlo scales easily to high-dimensional, path-dependent, and exotic
payoffs that have no closed-form solution.

**Relevance:** Foundational for `mc_european()` and the LSMC implementation; essential reading
for any practitioner working with complex structured products.

---

## [REF-19] Glasserman (2004)

**Glasserman, P. (2004).** *Monte Carlo Methods in Financial Engineering*. Springer,
Applications of Mathematics, Vol. 53. ISBN 978-0-387-00451-8.
<https://doi.org/10.1007/978-0-387-22429-9>

**Summary:**
The definitive graduate-level textbook on Monte Carlo simulation for financial derivatives.
Covers path simulation for a wide range of stochastic processes (GBM, CIR, Heston, jump
diffusions), variance reduction techniques (antithetic variates, control variates, stratified
sampling, importance sampling, quasi-Monte Carlo), sensitivity estimation (likelihood ratio
and pathwise methods), and special topics such as multi-asset options and interest rate
derivative pricing. The book provides rigorous theoretical convergence results alongside
practical guidance.

**Key techniques covered:**
- Antithetic variates, control variates, stratified sampling
- Pathwise (delta) and likelihood ratio (vega) sensitivities
- Quasi-Monte Carlo with Sobol sequences

**Relevance:** Primary reference for all Monte Carlo code in this workflow (Sections 5.3–5.4).

---

## [REF-20] Broadie & Glasserman (1996)

**Broadie, M., & Glasserman, P. (1996).** Estimating security price derivatives using
simulation. *Management Science*, **42**(2), 269–285.
<https://doi.org/10.1287/mnsc.42.2.269>

**Summary:**
Introduces finite-difference and pathwise estimators for option sensitivities (Greeks) computed
via simulation. The pathwise estimator yields unbiased, low-variance delta and vega estimates
by differentiating the payoff function analytically through the simulation. The likelihood
ratio estimator handles payoffs with discontinuous derivatives. Together, these methods enable
accurate Greek estimation within a Monte Carlo framework, critical for hedging applications.

**Relevance:** Provides theoretical basis for simulation-based Greek estimation; complements
the analytical Greek formulae in Section 6.

---

## [REF-21] Longstaff & Schwartz (2001)

**Longstaff, F. A., & Schwartz, E. S. (2001).** Valuing American options by simulation: A simple
least-squares approach. *Review of Financial Studies*, **14**(1), 113–147.
<https://doi.org/10.1093/rfs/14.1.113>

**Summary:**
Presents the **Least-Squares Monte Carlo (LSMC)** algorithm for pricing American-style options
via simulation. The key innovation is the use of ordinary least-squares regression at each
exercise date to estimate the conditional expected continuation value as a function of simulated
asset prices (using polynomial basis functions). By comparing this estimated continuation value
to the immediate exercise payoff at each node, the algorithm determines the optimal stopping
time for each path. The method is highly flexible: it handles multi-asset, path-dependent, and
high-dimensional problems where tree and PDE methods are impractical.

**Key algorithm:**
1. Simulate forward paths under **Q**
2. Work backwards from expiry
3. At each step: OLS regress discounted future cash-flows on basis functions of state variables
4. Exercise when immediate payoff > continuation value estimate

**Relevance:** Directly implemented in `lsmc_american()` (Section 8.6).

---

## [REF-22] Carr & Madan (1999)

**Carr, P., & Madan, D. (1999).** Option valuation using the fast Fourier transform. *Journal of
Computational Finance*, **2**(4), 61–73.
<https://doi.org/10.21314/JCF.1999.043>

**Summary:**
Derives an FFT-based method for pricing European options in any model that has a known
characteristic function for the log-price. By introducing a damping factor e^{αk} (where k is
the log-strike) the option pricing integral is mapped into the form of a Fourier transform, which
can be evaluated simultaneously across a grid of strikes with a single FFT computation. This
reduces the cost of calibrating to an entire volatility surface from O(N²) to O(N log N), making
Heston and other characteristic-function models practically calibratable.

**Relevance:** Underpins the Fourier/FFT approach in Section 5.5; critical for efficient Heston
model calibration.

---

## [REF-23] Jaeckel (2015)

**Jäckel, P. (2015).** Let's be rational. *Wilmott Magazine*, **75**, 40–53.
<https://doi.org/10.1002/wilm.10395>

**Summary:**
Proposes an ultra-fast, numerically stable algorithm for computing implied volatility from
European option prices. The method achieves machine-precision accuracy in a fixed small number
of iterations (typically two Newton steps after a high-quality initial guess), making it orders
of magnitude faster than generic root-finding methods such as Brent's method for full-surface
calibration tasks. The algorithm handles all moneyness regimes including deep ITM and OTM.

**Note:** *Wilmott Magazine* is peer-reviewed; this paper is the standard reference for the
implied-vol solver used in the `py_vollib` library.

**Relevance:** Underpins `implied_vol()` (Section 8.7) and is used internally by `py_vollib`.

---

## [REF-24] Gatheral (2006)

**Gatheral, J. (2006).** *The Volatility Surface: A Practitioner's Guide*. Wiley Finance.
ISBN 978-0-471-79251-2.

**Summary:**
A graduate-level textbook covering the volatility surface from both theoretical and practical
perspectives. Topics include local volatility, stochastic volatility (Heston, SABR), the
VIX/variance swap market, model calibration to market quotes, and the dynamics of the vol
surface under different models. Gatheral introduces the SVI (Stochastic Volatility Inspired)
parametrisation of the implied vol smile, which is widely used for fitting smooth surfaces to
sparse market data.

**Key topics:**
- Local vol vs. stochastic vol dynamics
- SVI smile parametrisation
- Variance swaps and VIX replication
- Calibration methodology and stability

**Relevance:** Essential reference for anyone building a production implied volatility surface
or calibrating a stochastic-vol model.

---

## [REF-25] Schwartz (1997)

**Schwartz, E. S. (1997).** The stochastic behavior of commodity prices: Implications for
valuation and hedging. *Journal of Finance*, **52**(3), 923–973.
<https://doi.org/10.1111/j.1540-6261.1997.tb02721.x>

**Summary:**
Develops and empirically estimates one-, two-, and three-factor models for commodity spot
prices and convenience yields, calibrated to oil futures data. The one-factor model is
geometric mean-reversion; the two-factor model adds a stochastic convenience yield (following
an Ornstein-Uhlenbeck process); the three-factor model further incorporates a stochastic
risk-free rate. The paper derives closed-form futures prices and European option prices for
all three models and shows that the two-factor model significantly outperforms the one-factor
model in fitting the oil futures curve.

**Key models:**
- 1F: dS = κ(μ − ln S) S dt + σ S dW (log-mean-reverting)
- 2F: dS = (μ − δ) S dt + σ₁ S dW₁; dδ = κ(α − δ) dt + σ₂ dW₂

**Relevance:** Central reference for commodity futures pricing models; widely used in energy
and agricultural derivatives research.

---

## [REF-26] Hull & White (1987)

**Hull, J. C., & White, A. (1987).** The pricing of options on assets with stochastic
volatilities. *Journal of Finance*, **42**(2), 281–300.
<https://doi.org/10.1111/j.1540-6261.1987.tb02568.x>

**Summary:**
One of the first stochastic volatility models to be rigorously developed. Hull and White assume
the asset variance follows a log-normal process uncorrelated with the asset price. They show
that the option price equals the BSM price averaged over the distribution of the mean integrated
variance over the option's life. While the zero-correlation assumption limits the model's ability
to fit the observed skew, the paper establishes the key insight that stochastic volatility
generates implied vol smiles and shows how to price them.

**Relevance:** Historical foundation for Heston (1993) and modern stochastic-vol models.

---

## [REF-27] Stein & Stein (1991)

**Stein, E. M., & Stein, J. C. (1991).** Stock price distributions with stochastic volatility:
An analytic approach. *Review of Financial Studies*, **4**(4), 727–752.
<https://doi.org/10.1093/rfs/4.4.727>

**Summary:**
Develops a stochastic volatility model in which the volatility (not variance) follows an
Ornstein-Uhlenbeck process. Using Fourier and characteristic function methods, the paper
derives the unconditional stock price distribution and European option prices in closed form.
The model generates the leptokurtic return distributions observed in equity markets and
produces a volatility smile consistent with typical market data. This paper directly preceded
and inspired Heston (1993).

**Relevance:** Predecessor to Heston's model; provides an important alternative characteristic-
function approach.

---

## Summary Table

| Ref | Authors | Year | Venue | Topic |
|-----|---------|------|-------|-------|
| REF-1 | Harrison & Kreps | 1979 | *J. Economic Theory* | Fundamental theorem, EMM |
| REF-2 | Harrison & Pliska | 1981 | *Stochastic Proc. & Appl.* | Continuous-time FTAP |
| REF-3 | Black & Scholes | 1973 | *J. Political Economy* | BSM formula |
| REF-4 | Merton | 1973 | *Bell J. Economics* | BSM extension, put-call parity |
| REF-5 | Black | 1976 | *J. Financial Economics* | Futures options (Black-76) |
| REF-6 | Cox, Ross & Rubinstein | 1979 | *J. Financial Economics* | Binomial tree |
| REF-7 | Cox, Ingersoll & Ross | 1981 | *J. Financial Economics* | Futures vs. forwards |
| REF-8 | Heston | 1993 | *Rev. Financial Studies* | Stochastic volatility |
| REF-9 | Dupire | 1994 | *Risk* | Local volatility |
| REF-10 | Derman & Kani | 1994 | *Risk* | Implied binomial tree |
| REF-11 | Merton | 1976 | *J. Financial Economics* | Jump-diffusion |
| REF-12 | Kou | 2002 | *Management Science* | Double-exponential jumps |
| REF-13 | Vasicek | 1977 | *J. Financial Economics* | Short-rate model |
| REF-14 | Cox, Ingersoll & Ross | 1985 | *Econometrica* | CIR term structure |
| REF-15 | Heath, Jarrow & Morton | 1992 | *Econometrica* | Forward-rate HJM framework |
| REF-16 | Hagan et al. | 2002 | *Wilmott Magazine* | SABR model |
| REF-17 | Brennan & Schwartz | 1977 | *J. Finance* | FD method for American puts |
| REF-18 | Boyle | 1977 | *J. Financial Economics* | Monte Carlo for options |
| REF-19 | Glasserman | 2004 | Springer (book) | Monte Carlo in fin. engineering |
| REF-20 | Broadie & Glasserman | 1996 | *Management Science* | Simulation Greeks |
| REF-21 | Longstaff & Schwartz | 2001 | *Rev. Financial Studies* | LSMC for American options |
| REF-22 | Carr & Madan | 1999 | *J. Computational Finance* | FFT option pricing |
| REF-23 | Jäckel | 2015 | *Wilmott Magazine* | Implied vol algorithm |
| REF-24 | Gatheral | 2006 | Wiley (book) | Volatility surface |
| REF-25 | Schwartz | 1997 | *J. Finance* | Commodity stochastic models |
| REF-26 | Hull & White | 1987 | *J. Finance* | Stochastic volatility |
| REF-27 | Stein & Stein | 1991 | *Rev. Financial Studies* | Stochastic vol analytic |

---

*Document version: 1.0 — Quant Workflow Project (Project 2)*
