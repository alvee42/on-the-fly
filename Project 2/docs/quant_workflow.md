# Quant Workflow: Futures & Options Pricing

> **Scope:** A research-grade quantitative workflow for pricing, hedging, and analysing futures and
> options. This document synthesises the canonical academic literature, the numerical methods used in
> practice, the Python libraries required to implement them, and representative code snippets.

---

## Table of Contents

1. [Conceptual Map](#1-conceptual-map)
2. [Foundational Theory](#2-foundational-theory)
   - 2.1 [Arbitrage-Free Pricing & Risk-Neutral Measure](#21-arbitrage-free-pricing--risk-neutral-measure)
   - 2.2 [Black-Scholes-Merton Framework](#22-black-scholes-merton-framework)
   - 2.3 [Futures & Forward Pricing](#23-futures--forward-pricing)
3. [Volatility Models](#3-volatility-models)
   - 3.1 [Stochastic Volatility — Heston Model](#31-stochastic-volatility--heston-model)
   - 3.2 [Local Volatility — Dupire Framework](#32-local-volatility--dupire-framework)
   - 3.3 [Jump-Diffusion Models](#33-jump-diffusion-models)
4. [Interest Rate Models (for options on rates & futures)](#4-interest-rate-models)
   - 4.1 [Vasicek & CIR](#41-vasicek--cir)
   - 4.2 [Heath-Jarrow-Morton Framework](#42-heath-jarrow-morton-framework)
5. [Numerical Methods](#5-numerical-methods)
   - 5.1 [Binomial & Trinomial Trees](#51-binomial--trinomial-trees)
   - 5.2 [Finite Difference Methods](#52-finite-difference-methods)
   - 5.3 [Monte Carlo Simulation](#53-monte-carlo-simulation)
   - 5.4 [Least-Squares Monte Carlo (American Options)](#54-least-squares-monte-carlo-american-options)
   - 5.5 [Fourier / Characteristic-Function Methods](#55-fourier--characteristic-function-methods)
6. [Greeks & Risk Management](#6-greeks--risk-management)
7. [Python Libraries](#7-python-libraries)
8. [Implementation Code](#8-implementation-code)
   - 8.1 [Black-Scholes Pricer](#81-black-scholes-pricer)
   - 8.2 [Futures Pricing & Basis](#82-futures-pricing--basis)
   - 8.3 [Heston Model via Characteristic Function](#83-heston-model-via-characteristic-function)
   - 8.4 [Binomial Tree (CRR)](#84-binomial-tree-crr)
   - 8.5 [Monte Carlo with Variance Reduction](#85-monte-carlo-with-variance-reduction)
   - 8.6 [Longstaff-Schwartz LSMC for American Options](#86-longstaff-schwartz-lsmc-for-american-options)
   - 8.7 [Implied Volatility Surface](#87-implied-volatility-surface)
9. [References](#9-references)

---

## 1. Conceptual Map

```
Market Observables
  ├── Spot Prices, Futures Curves, Dividend Yields, Repo Rates
  └── Implied Volatility Surface

No-Arbitrage Pricing
  ├── Risk-Neutral Measure  (Harrison & Kreps 1979; Harrison & Pliska 1981)
  └── Fundamental Theorem of Asset Pricing

Underlying Process Models
  ├── Geometric Brownian Motion  (Black-Scholes-Merton 1973)
  ├── Stochastic Volatility      (Heston 1993; Hull & White 1987)
  ├── Local Volatility           (Dupire 1994; Derman & Kani 1994)
  ├── Jump-Diffusion             (Merton 1976; Kou 2002)
  └── Multi-Factor Term Structure (Vasicek 1977; Cox-Ingersoll-Ross 1985; HJM 1992)

Numerical Engines
  ├── Closed-Form Formulae
  ├── Binomial/Trinomial Trees   (Cox, Ross & Rubinstein 1979)
  ├── Finite Difference PDEs     (Brennan & Schwartz 1977)
  ├── Monte Carlo                (Boyle 1977; Glasserman 2004)
  └── Fourier / FFT Methods      (Carr & Madan 1999)

Risk / Calibration
  ├── Greeks (Delta, Gamma, Vega, Theta, Rho)
  └── Model Calibration to Market Quotes
```

---

## 2. Foundational Theory

### 2.1 Arbitrage-Free Pricing & Risk-Neutral Measure

The cornerstone of modern derivatives pricing is the equivalence between the absence of arbitrage
and the existence of an equivalent martingale measure (EMM).

**Key result:** Under the risk-neutral measure **Q**, the price of any derivative paying _H_ at
maturity _T_ is

```
V(t) = e^{-r(T-t)} * E^Q[ H | F_t ]
```

where _r_ is the risk-free rate and _F_t_ is the filtration up to _t_.

**Source:** Harrison & Kreps (1979) [REF-1]; Harrison & Pliska (1981) [REF-2].

---

### 2.2 Black-Scholes-Merton Framework

The Black-Scholes-Merton (BSM) model assumes the underlying asset _S_ follows:

```
dS = μ S dt + σ S dW^P
```

Under **Q** this becomes:

```
dS = r S dt + σ S dW^Q
```

The resulting **Black-Scholes PDE** is:

```
∂V/∂t + ½ σ² S² ∂²V/∂S² + r S ∂V/∂S − r V = 0
```

**Closed-form call price:**

```
C = S N(d₁) − K e^{−rT} N(d₂)

d₁ = [ln(S/K) + (r + σ²/2) T] / (σ √T)
d₂ = d₁ − σ √T
```

**Sources:** Black & Scholes (1973) [REF-3]; Merton (1973) [REF-4].

**Assumptions & known limitations:**
- Constant volatility (contradicted by the volatility smile / skew)
- Continuous trading, no transaction costs
- Log-normal returns (fat tails observed empirically)

---

### 2.3 Futures & Forward Pricing

**Cost-of-carry model for a non-dividend-paying asset:**

```
F₀ = S₀ e^{rT}
```

**With continuous dividend yield _q_:**

```
F₀ = S₀ e^{(r − q)T}
```

**With convenience yield _y_ (commodities):**

```
F₀ = S₀ e^{(r + u − y)T}
```

where _u_ is storage cost as a continuous proportion.

**Options on futures — Black (1976) model:**

```
C = e^{−rT} [F N(d₁) − K N(d₂)]

d₁ = [ln(F/K) + σ²T/2] / (σ √T)
d₂ = d₁ − σ √T
```

**Sources:** Black (1976) [REF-5]; Cox, Ingersoll & Ross (1985b) [REF-7].

---

## 3. Volatility Models

### 3.1 Stochastic Volatility — Heston Model

Heston (1993) introduces mean-reverting stochastic variance _v_:

```
dS  = r S dt  + √v S dW₁
dv  = κ(θ − v) dt + ξ √v dW₂
cov(dW₁, dW₂) = ρ dt
```

Parameters:
| Symbol | Meaning |
|--------|---------|
| κ | Mean-reversion speed |
| θ | Long-run variance |
| ξ | Volatility of volatility (vol-of-vol) |
| ρ | Correlation (typically negative for equities) |
| v₀ | Initial variance |

**Semi-analytic price via characteristic function:**

```
C = S P₁ − K e^{−rT} P₂

Pⱼ = ½ + (1/π) ∫₀^∞ Re[ e^{−iφ ln K} fⱼ(φ) / (iφ) ] dφ
```

where _fⱼ_ are the conditional characteristic functions of ln _S_T_.

**Source:** Heston (1993) [REF-8].

The Feller condition (2κθ > ξ²) ensures the variance process stays strictly positive.

---

### 3.2 Local Volatility — Dupire Framework

Dupire (1994) showed that any arbitrage-free implied volatility surface is consistent with a unique
**local volatility function** σ_loc(S, t):

```
σ_loc²(K, T) = [∂C/∂T + r K ∂C/∂K] / [½ K² ∂²C/∂K²]
```

This allows exact calibration to all market quotes, but the smile dynamics are poor (Hagan et al.
2002 [REF-16]).

**Sources:** Dupire (1994) [REF-9]; Derman & Kani (1994) [REF-10].

---

### 3.3 Jump-Diffusion Models

**Merton (1976)** adds a compound Poisson jump process:

```
dS/S = (μ − λk̄) dt + σ dW + (J − 1) dN

J = jump size multiplier (log-normal)
N = Poisson process with intensity λ
k̄ = E[J − 1] = mean percentage jump
```

**Price** is a weighted sum of BSM prices over Poisson probabilities:

```
C_Merton = Σ_{n=0}^∞ [ e^{−λ'T} (λ'T)ⁿ / n! ] × BSM(S, K, rₙ, σₙ, T)
```

**Kou (2002)** uses a double-exponential (asymmetric) jump distribution, giving a better fit to
the short-term implied vol skew.

**Sources:** Merton (1976) [REF-11]; Kou (2002) [REF-12].

---

## 4. Interest Rate Models

### 4.1 Vasicek & CIR

**Vasicek (1977)** — short rate follows an Ornstein-Uhlenbeck process:

```
dr = κ(θ − r) dt + σ dW
```

Allows negative rates; closed-form bond and caplet prices.

**Cox-Ingersoll-Ross (1985a)** — short rate:

```
dr = κ(θ − r) dt + σ √r dW
```

Positive rates (if 2κθ > σ²); closed-form bond prices.

**Sources:** Vasicek (1977) [REF-13]; Cox, Ingersoll & Ross (1985a) [REF-14].

---

### 4.2 Heath-Jarrow-Morton Framework

HJM models the entire **forward-rate curve** f(t, T):

```
df(t, T) = α(t, T) dt + σ(t, T) dW(t)
```

No-arbitrage drift condition:

```
α(t, T) = σ(t, T) ∫ₜᵀ σ(t, u) du
```

HJM generalises Vasicek and CIR as special cases and is the foundation for the LIBOR/SOFR
market models.

**Source:** Heath, Jarrow & Morton (1992) [REF-15].

---

## 5. Numerical Methods

### 5.1 Binomial & Trinomial Trees

**Cox-Ross-Rubinstein (CRR, 1979)** binomial tree:

```
u = e^{σ √Δt}       (up factor)
d = 1/u              (down factor)
p = (e^{rΔt} − d) / (u − d)   (risk-neutral up probability)
```

Convergence to BSM is O(1/N). Trinomial trees improve convergence to O(1/N²) and better handle
early-exercise boundaries for American options.

**Source:** Cox, Ross & Rubinstein (1979) [REF-6].

---

### 5.2 Finite Difference Methods

The BSM PDE is discretised on a grid (S, t). Three standard schemes:

| Scheme | Stability | Accuracy |
|--------|-----------|----------|
| Explicit (forward Euler) | Conditional | O(Δt, ΔS²) |
| Implicit (backward Euler) | Unconditional | O(Δt, ΔS²) |
| Crank-Nicolson | Unconditional | O(Δt², ΔS²) |

Free-boundary conditions handle American early exercise.

**Source:** Brennan & Schwartz (1977) [REF-17].

---

### 5.3 Monte Carlo Simulation

**Boyle (1977)** introduced Monte Carlo to option pricing:

1. Simulate N paths of the risk-neutral process.
2. Compute payoff for each path.
3. Average and discount.

**Variance reduction techniques:**
- Antithetic variates
- Control variates (use BSM price as control for exotic payoff)
- Importance sampling
- Quasi-Monte Carlo (Sobol, Halton sequences)

**Sources:** Boyle (1977) [REF-18]; Glasserman (2004) [REF-19]; Broadie & Glasserman (1996)
[REF-20].

---

### 5.4 Least-Squares Monte Carlo (American Options)

**Longstaff & Schwartz (2001)** LSMC algorithm:

1. Simulate forward paths.
2. Work backwards from expiry.
3. At each step, regress (OLS) continuation value onto basis functions of state variables.
4. Compare estimated continuation value with immediate exercise value; take the maximum.

**Source:** Longstaff & Schwartz (2001) [REF-21].

---

### 5.5 Fourier / Characteristic-Function Methods

**Carr & Madan (1999)** FFT pricing: for any model with a known characteristic function φ(u):

```
C(K) = (e^{−α ln K} / π) ∫₀^∞ e^{−iφ ln K} ψ(φ) dφ
```

where ψ is a modified characteristic function. A single FFT prices a whole strike grid simultaneously.

**Source:** Carr & Madan (1999) [REF-22].

---

## 6. Greeks & Risk Management

| Greek | Definition | BSM Formula (Call) |
|-------|------------|--------------------|
| Delta (Δ) | ∂V/∂S | N(d₁) |
| Gamma (Γ) | ∂²V/∂S² | n(d₁) / (S σ √T) |
| Vega (ν) | ∂V/∂σ | S n(d₁) √T |
| Theta (Θ) | ∂V/∂t | −[S n(d₁) σ / (2√T)] − rK e^{−rT} N(d₂) |
| Rho (ρ) | ∂V/∂r | K T e^{−rT} N(d₂) |
| Vanna | ∂Δ/∂σ | −n(d₁) d₂ / σ |
| Volga | ∂ν/∂σ | S n(d₁) √T d₁ d₂ / σ |

Delta-Gamma-Vega hedging of a portfolio (Δ_portfolio = 0, Γ_portfolio = 0, ν_portfolio = 0) requires
at least two additional option positions.

---

## 7. Python Libraries

### Core Scientific Stack

| Library | Version (min) | Purpose |
|---------|--------------|---------|
| `numpy` | ≥ 1.24 | Vectorised array operations |
| `scipy` | ≥ 1.10 | Statistical distributions, integration, optimisation |
| `pandas` | ≥ 2.0 | Time-series data management |
| `matplotlib` | ≥ 3.7 | Plotting |
| `plotly` | ≥ 5.14 | Interactive visualisations |

### Quantitative Finance

| Library | Purpose |
|---------|---------|
| `QuantLib` (via `QuantLib-Python`) | Industry-standard C++ engine: trees, FD, Monte Carlo, yield curves, swaptions, caps/floors |
| `py_vollib` | Black-Scholes, Greeks, implied vol via Jaeckel's method |
| `py_vollib_vectorized` | Vectorised version of py_vollib for surface fitting |
| `mibian` | Black-Scholes, Garman-Kohlhagen (FX options), Greeks |
| `arch` | GARCH family models for realised-vol estimation |
| `statsmodels` | Econometrics, regression, time-series |
| `heston_nandi` | Heston-Nandi GARCH option pricing |

### Market Data

| Library | Purpose |
|---------|---------|
| `yfinance` | Yahoo Finance: equity prices, option chains |
| `pandas-datareader` | FRED, Fama-French, World Bank data |
| `ibapi` | Interactive Brokers TWS API (live/paper trading) |
| `ccxt` | Cryptocurrency exchange connectors |

### Machine Learning / Calibration

| Library | Purpose |
|---------|---------|
| `scikit-learn` | Regression basis functions for LSMC |
| `tensorflow` / `torch` | Deep Hedging, neural network option pricing |
| `optuna` | Bayesian hyperparameter optimisation for calibration |

### Installation

```bash
pip install numpy scipy pandas matplotlib plotly \
            QuantLib-Python py_vollib py_vollib_vectorized \
            mibian arch statsmodels yfinance pandas-datareader \
            scikit-learn optuna
```

---

## 8. Implementation Code

> All snippets assume Python ≥ 3.10 and the libraries listed above.

---

### 8.1 Black-Scholes Pricer

```python
import numpy as np
from scipy.stats import norm

def black_scholes(S: float, K: float, T: float, r: float,
                  sigma: float, option_type: str = "call") -> float:
    """
    European option price under BSM.

    Parameters
    ----------
    S          : Current spot price
    K          : Strike price
    T          : Time to expiry in years
    r          : Risk-free rate (continuous compounding)
    sigma      : Implied / historical volatility (annualised)
    option_type: 'call' or 'put'

    Returns
    -------
    float: Option premium

    References
    ----------
    Black & Scholes (1973), Merton (1973).
    """
    d1 = (np.log(S / K) + (r + 0.5 * sigma**2) * T) / (sigma * np.sqrt(T))
    d2 = d1 - sigma * np.sqrt(T)

    if option_type == "call":
        price = S * norm.cdf(d1) - K * np.exp(-r * T) * norm.cdf(d2)
    elif option_type == "put":
        price = K * np.exp(-r * T) * norm.cdf(-d2) - S * norm.cdf(-d1)
    else:
        raise ValueError("option_type must be 'call' or 'put'")
    return price


def bsm_greeks(S: float, K: float, T: float, r: float,
               sigma: float, option_type: str = "call") -> dict:
    """Return a dict of first-order Greeks for a European option."""
    d1 = (np.log(S / K) + (r + 0.5 * sigma**2) * T) / (sigma * np.sqrt(T))
    d2 = d1 - sigma * np.sqrt(T)
    n_d1 = norm.pdf(d1)

    delta = norm.cdf(d1) if option_type == "call" else norm.cdf(d1) - 1
    gamma = n_d1 / (S * sigma * np.sqrt(T))
    vega  = S * n_d1 * np.sqrt(T)
    theta_call = (- S * n_d1 * sigma / (2 * np.sqrt(T))
                  - r * K * np.exp(-r * T) * norm.cdf(d2))
    theta = theta_call if option_type == "call" else (
        theta_call + r * K * np.exp(-r * T))
    rho_call = K * T * np.exp(-r * T) * norm.cdf(d2)
    rho = rho_call if option_type == "call" else (
        -K * T * np.exp(-r * T) * norm.cdf(-d2))

    return {"delta": delta, "gamma": gamma,
            "vega": vega / 100,   # per 1-vol-point
            "theta": theta / 365, # per calendar day
            "rho": rho / 100}     # per 1 bp move in r
```

---

### 8.2 Futures Pricing & Basis

```python
def futures_price(S: float, r: float, q: float, T: float,
                  u: float = 0.0, y: float = 0.0) -> float:
    """
    Cost-of-carry futures / forward price.

    Parameters
    ----------
    S : Spot price
    r : Risk-free rate (continuous)
    q : Continuous dividend / convenience credit yield
    T : Time to delivery (years)
    u : Storage cost (continuous proportion)
    y : Convenience yield (continuous)

    Returns
    -------
    float: Theoretical futures price

    References
    ----------
    Black (1976); Cox, Ingersoll & Ross (1985b).
    """
    return S * np.exp((r - q + u - y) * T)


def black76(F: float, K: float, T: float, r: float,
            sigma: float, option_type: str = "call") -> float:
    """
    Black (1976) model: European option on a futures contract.

    References
    ----------
    Black (1976).
    """
    d1 = (np.log(F / K) + 0.5 * sigma**2 * T) / (sigma * np.sqrt(T))
    d2 = d1 - sigma * np.sqrt(T)

    if option_type == "call":
        return np.exp(-r * T) * (F * norm.cdf(d1) - K * norm.cdf(d2))
    else:
        return np.exp(-r * T) * (K * norm.cdf(-d2) - F * norm.cdf(-d1))
```

---

### 8.3 Heston Model via Characteristic Function

```python
import numpy as np
from scipy.integrate import quad

def heston_char_func(phi: complex, S: float, v0: float, kappa: float,
                     theta: float, xi: float, rho: float,
                     r: float, T: float, j: int) -> complex:
    """Characteristic function for the Heston model (Heston 1993)."""
    i = complex(0, 1)
    if j == 1:
        u, b = 0.5, kappa - rho * xi
    else:
        u, b = -0.5, kappa

    a  = kappa * theta
    x  = np.log(S)
    d  = np.sqrt((rho * xi * i * phi - b)**2 - xi**2 * (2 * u * i * phi - phi**2))
    g  = (b - rho * xi * i * phi + d) / (b - rho * xi * i * phi - d)
    C  = (r * i * phi * T
          + (a / xi**2) * ((b - rho * xi * i * phi + d) * T
                           - 2 * np.log((1 - g * np.exp(d * T)) / (1 - g))))
    D  = ((b - rho * xi * i * phi + d) / xi**2
          * (1 - np.exp(d * T)) / (1 - g * np.exp(d * T)))
    return np.exp(C + D * v0 + i * phi * x)


def heston_price(S: float, K: float, T: float, r: float,
                 v0: float, kappa: float, theta: float,
                 xi: float, rho: float,
                 option_type: str = "call") -> float:
    """
    Semi-analytic Heston (1993) option price using numerical integration.

    References
    ----------
    Heston (1993).
    """
    def integrand(phi, j):
        cf = heston_char_func(phi, S, v0, kappa, theta, xi, rho, r, T, j)
        return np.real(np.exp(-1j * phi * np.log(K)) * cf / (1j * phi))

    P1 = 0.5 + (1 / np.pi) * quad(integrand, 0, 200, args=(1,), limit=200)[0]
    P2 = 0.5 + (1 / np.pi) * quad(integrand, 0, 200, args=(2,), limit=200)[0]

    call = S * P1 - K * np.exp(-r * T) * P2
    if option_type == "call":
        return call
    else:
        return call - S + K * np.exp(-r * T)  # put-call parity
```

---

### 8.4 Binomial Tree (CRR)

```python
def crr_tree(S: float, K: float, T: float, r: float,
             sigma: float, N: int = 500,
             option_type: str = "call",
             style: str = "european") -> float:
    """
    Cox-Ross-Rubinstein binomial tree.

    Parameters
    ----------
    N     : Number of time steps
    style : 'european' or 'american'

    References
    ----------
    Cox, Ross & Rubinstein (1979).
    """
    dt = T / N
    u  = np.exp(sigma * np.sqrt(dt))
    d  = 1.0 / u
    p  = (np.exp(r * dt) - d) / (u - d)
    disc = np.exp(-r * dt)

    # Terminal asset prices
    j  = np.arange(N + 1)
    ST = S * u**j * d**(N - j)

    # Terminal payoffs
    if option_type == "call":
        V = np.maximum(ST - K, 0)
    else:
        V = np.maximum(K - ST, 0)

    # Backward induction
    for i in range(N - 1, -1, -1):
        j   = np.arange(i + 1)
        Si  = S * u**j * d**(i - j)
        V   = disc * (p * V[1:i+2] + (1 - p) * V[0:i+1])
        if style == "american":
            if option_type == "call":
                V = np.maximum(V, Si - K)
            else:
                V = np.maximum(V, K - Si)
    return float(V[0])
```

---

### 8.5 Monte Carlo with Variance Reduction

```python
def mc_european(S: float, K: float, T: float, r: float,
                sigma: float, n_paths: int = 200_000,
                option_type: str = "call",
                antithetic: bool = True,
                seed: int = 42) -> dict:
    """
    Monte Carlo pricing with antithetic variates.

    References
    ----------
    Boyle (1977); Glasserman (2004).
    """
    rng = np.random.default_rng(seed)
    Z   = rng.standard_normal(n_paths // 2)
    if antithetic:
        Z = np.concatenate([Z, -Z])

    ST = S * np.exp((r - 0.5 * sigma**2) * T + sigma * np.sqrt(T) * Z)

    payoffs = (np.maximum(ST - K, 0) if option_type == "call"
               else np.maximum(K - ST, 0))
    discounted = np.exp(-r * T) * payoffs

    price = discounted.mean()
    se    = discounted.std(ddof=1) / np.sqrt(len(discounted))
    return {"price": price, "std_error": se,
            "ci_95": (price - 1.96 * se, price + 1.96 * se)}
```

---

### 8.6 Longstaff-Schwartz LSMC for American Options

```python
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression

def lsmc_american(S: float, K: float, T: float, r: float,
                  sigma: float, n_paths: int = 50_000,
                  n_steps: int = 252, degree: int = 3,
                  option_type: str = "put",
                  seed: int = 0) -> float:
    """
    Longstaff-Schwartz Least-Squares Monte Carlo for American options.

    References
    ----------
    Longstaff & Schwartz (2001).
    """
    rng = np.random.default_rng(seed)
    dt  = T / n_steps
    disc = np.exp(-r * dt)

    # Simulate paths (n_steps + 1 columns, n_paths rows)
    Z = rng.standard_normal((n_paths, n_steps))
    increments = (r - 0.5 * sigma**2) * dt + sigma * np.sqrt(dt) * Z
    log_S = np.log(S) + np.cumsum(increments, axis=1)
    S_paths = np.hstack([np.full((n_paths, 1), S), np.exp(log_S)])

    # Payoff at each node
    if option_type == "put":
        payoffs = np.maximum(K - S_paths, 0)
    else:
        payoffs = np.maximum(S_paths - K, 0)

    # Cash-flow matrix (terminal)
    cashflows = payoffs[:, -1].copy()

    # Backward regression
    poly = PolynomialFeatures(degree=degree, include_bias=True)
    for t in range(n_steps - 1, 0, -1):
        itm = payoffs[:, t] > 0
        if itm.sum() == 0:
            continue
        X   = poly.fit_transform(S_paths[itm, t].reshape(-1, 1))
        Y   = disc * cashflows[itm]
        reg = LinearRegression(fit_intercept=False).fit(X, Y)
        continuation = reg.predict(X)
        exercise = payoffs[itm, t]
        exercise_now = exercise > continuation
        cashflows[itm] = np.where(exercise_now, exercise, disc * cashflows[itm])
        cashflows[~itm] *= disc

    return float(np.mean(disc * cashflows))
```

---

### 8.7 Implied Volatility Surface

```python
from scipy.optimize import brentq

def implied_vol(market_price: float, S: float, K: float,
                T: float, r: float,
                option_type: str = "call",
                tol: float = 1e-6) -> float:
    """
    Newton-Brentq implied volatility solver.

    References
    ----------
    Jaeckel (2015) [REF-23] for high-accuracy root-finding.
    """
    def objective(sigma):
        return black_scholes(S, K, T, r, sigma, option_type) - market_price

    try:
        return brentq(objective, 1e-6, 10.0, xtol=tol)
    except ValueError:
        return float("nan")


def build_vol_surface(option_chain: "pd.DataFrame",
                      S: float, r: float) -> "pd.DataFrame":
    """
    Compute implied volatility for every row of an option chain.

    Expected columns: strike, expiry_years, mid_price, option_type
    Returns the input DataFrame with an added 'iv' column.
    """
    import pandas as pd
    chain = option_chain.copy()
    chain["iv"] = chain.apply(
        lambda row: implied_vol(row["mid_price"], S,
                                row["strike"], row["expiry_years"],
                                r, row["option_type"]),
        axis=1,
    )
    return chain
```

---

## 9. References

See [`references.md`](./references.md) for the complete annotated bibliography.

Key citations in-text use the format **[REF-N]** throughout this document.

---

*Document version: 1.0 — Quant Workflow Project (Project 2)*
