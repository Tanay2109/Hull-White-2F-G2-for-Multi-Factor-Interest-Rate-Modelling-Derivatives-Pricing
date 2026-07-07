# Hull White 2F for Multi-Factor Interest-Rate Modelling

A Python implementation of the Hull-White 2-Factor (G2++) short-rate model, calibrated to SOFR OIS swap curves and swaption volatilities, and used to price fixed-income derivatives and estimate interest rate VaR through Monte Carlo simulation.

## Overview

This project builds a two-factor short-rate model from the ground up and fits it to real market conventions rather than toy inputs. Starting from SOFR OIS par curves (2018-2023), it constructs a zero curve and discount curve, builds a swaption volatility surface priced under the Bachelier model, and calibrates the G2++ model's five parameters to match those market swaption prices. The calibrated model is then used to simulate short-rate paths, price a swaption out of sample, and compute a 10-day VaR for a bond position.

## What the project covers

**1. Market Curve Construction**
SOFR OIS par rates across 11 tenors (3 months to 30 years) are converted into a continuously-compounded zero curve and discount factor curve. A cubic spline on log-discount factors gives a smooth, arbitrage-consistent discount function and instantaneous forward curve for use in pricing.

**2. Swaption Market Data**
A normal (Bachelier) volatility surface across three expiries and three swap tenors is used to generate market swaption prices, with par swap rates computed directly from the discount curve.

**3. G2++ Model Formulation**
The two-factor Gaussian short-rate model is implemented analytically, including the zero-coupon bond pricing formula and the variance function that ties the model to the market curve exactly at every tenor.

**4. Monte Carlo Calibration**
Correlated factor paths for the two G2++ state variables are simulated using exact Ornstein-Uhlenbeck discretization with antithetic variates for variance reduction. Swaption prices from these simulations are converted back into implied normal vols and compared against market vols. The five model parameters (a, b, sigma, eta, rho) are calibrated by minimizing the RMSE between model and market implied vols using Nelder-Mead optimization.

**5. Out-of-Sample Pricing Check**
Once calibrated, the model prices a 2Y-into-5Y payer swaption using a full 10,000-path simulation and compares it against the market price, giving an out-of-sample check on calibration quality.

**6. Risk Simulation and VaR**
10,000 short-rate paths are simulated over a 5-year horizon, and a separate 10-day-ahead simulation is used to build the P&L distribution of a 10-year zero-coupon bond position. The 99% VaR is estimated from this distribution and validated through a train/test split backtest.

## Key Results

| Metric | Result |
|---|---|
| Calibrated parameters (a, b) | 0.0417, 0.2673 |
| Calibrated volatility parameters (sigma, eta) | 0.0145, 0.0087 |
| Calibrated correlation (rho) | -0.8282 |
| Swaption calibration RMSE | 6.24 bps |
| Out-of-sample swaption pricing error | 5.42 bps |
| 10-day 99% VaR (per 100 notional) | 2.6971 |
| Empirical coverage of 99% VaR | 98.9% |

The calibration RMSE of 6.24 bps shows the model fits the swaption vol surface closely across all nine expiry-tenor combinations. The out-of-sample pricing error of 5.42 bps on a held-out swaption confirms the calibrated parameters generalize rather than just overfitting the calibration set. On the risk side, the backtested VaR shows 98.9% empirical coverage against a 99% target, close enough to the target to validate the model without the excess conservatism that would erode capital efficiency.

## Visuals

<!-- Insert: SOFR OIS curve evolution 2018-2023 -->
<img width="678" height="470" alt="image" src="https://github.com/user-attachments/assets/f59096e9-6c0f-4141-92ca-0ac3f91c64fa" />


<!-- Insert: Swaption calibration fit, market vs model implied vols -->
<img width="708" height="483" alt="image" src="https://github.com/user-attachments/assets/4a891e9a-f20d-4c88-bdeb-ec903e7a949e" />


<!-- Insert: Simulated G2++ short rate paths -->
<img width="689" height="470" alt="image" src="https://github.com/user-attachments/assets/86666fc5-b3be-495e-803e-6862bfa41956" />


<!-- Insert: 10-day P&L distribution with 99% VaR threshold -->
<img width="695" height="470" alt="image" src="https://github.com/user-attachments/assets/c077ea46-f3d5-4d6e-83c6-c06b184fe121" />


## Tech Stack

- Python (NumPy, pandas, SciPy, Matplotlib)
- `scipy.optimize` for Nelder-Mead calibration
- `scipy.interpolate` for cubic spline curve construction
- `scipy.stats` for the Bachelier normal pricing model

## Model Background

The G2++ model extends the 1 factor Hull-White model by adding a second stochastic factor, giving it enough flexibility to fit both the short and long end of the volatility surface simultaneously, something a one-factor model structurally cannot do. It remains fully analytically tractable for bond pricing while adding the extra degree of freedom needed for realistic swaption calibration.
