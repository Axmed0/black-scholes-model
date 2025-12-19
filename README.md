# Black–Scholes Option Pricing with Interactive 3D Visualizations

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Axmed0/black-scholes-model/blob/main/blackscholescode.ipynb)
[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/Axmed0/black-scholes-model/main?filepath=blackscholescode.ipynb)
[![nbviewer](https://img.shields.io/badge/render-nbviewer-orange.svg)](https://nbviewer.org/github/Axmed0/black-scholes-model/blob/main/blackscholescode.ipynb)

A Python implementation of Black–Scholes pricing for European options with full Greeks calculation and interactive 3D surface plots using Plotly.

## Features

- **Option Pricing**: European call and put options using the Black–Scholes model
- **Greeks Calculation**: Delta, Gamma, Vega, Theta, and Rho for both calls and puts
- **Interactive 3D Visualizations**: Two types of Plotly surface plots with dropdown menus
  - **Surface A**: Spot price (S) vs Volatility (σ) with fixed time to expiry
  - **Surface B**: Spot price (S) vs Time to expiry (T) with fixed volatility
- **Put-Call Parity Verification**: Automatic validation of pricing consistency

## Installation

Install the required dependencies:

```bash
python -m pip install numpy plotly jupyterlab notebook
```

## Usage

1. Launch Jupyter:
```bash
python -m jupyterlab
```

2. Open the notebook and adjust parameters in Cell 2:
```python
S = 34.03      # Spot price
K = 40.00      # Strike price
r = 0.0412     # Risk-free rate
T = 30/365     # Time to expiry (years)
sigma = 0.35   # Volatility (annualized)
```

3. Run all cells to generate:
   - Console output with prices and Greeks for both calls and puts
   - Interactive HTML file with 3D visualizations that opens in your browser

## Mathematical Background

### Black–Scholes Formula

**Call Price:**
$$C = S \cdot N(d_1) - K e^{-rT} N(d_2)$$

**Put Price:**
$$P = K e^{-rT} N(-d_2) - S N(-d_1)$$

where:

$$d_1 = \frac{\ln(S/K) + (r + \frac{1}{2}\sigma^2)T}{\sigma\sqrt{T}}, \quad d_2 = d_1 - \sigma\sqrt{T}$$

### The Greeks

| Greek | Measures | Call | Put |
|-------|----------|------|-----|
| **Delta (Δ)** | Price sensitivity to spot | $N(d_1)$ | $N(d_1) - 1$ |
| **Gamma (Γ)** | Delta sensitivity to spot | $\frac{n(d_1)}{S\sigma\sqrt{T}}$ | Same |
| **Vega (ν)** | Price sensitivity to volatility | $S \cdot n(d_1)\sqrt{T}$ | Same |
| **Theta (Θ)** | Price sensitivity to time | $-\frac{S n(d_1)\sigma}{2\sqrt{T}} - rK e^{-rT}N(d_2)$ | $-\frac{S n(d_1)\sigma}{2\sqrt{T}} + rK e^{-rT}N(-d_2)$ |
| **Rho (ρ)** | Price sensitivity to interest rate | $KT e^{-rT}N(d_2)$ | $-KT e^{-rT}N(-d_2)$ |

where $N(x)$ is the standard normal CDF and $n(x) = \frac{1}{\sqrt{2\pi}}e^{-x^2/2}$ is the standard normal PDF.

## Output

### Console Output
- Prices for both call and put options
- All Greeks with scaled values (per 1% for Vega and Rho, per day for Theta)
- Put-call parity verification

### Interactive 3D Plot
- Dropdown menu with 20 different visualizations
- Rotate, zoom, and pan the 3D surfaces
- Compare how prices and Greeks change across different parameters

## Example Output

```
======================================================================
BLACK-SCHOLES OPTION PRICING
======================================================================

Parameters:
  Spot Price (S):        $34.03
  Strike Price (K):      $40.00
  Risk-free Rate (r):    4.12%
  Time to Expiry (T):    0.0822 years (30 days)
  Volatility (σ):        35.00%

======================================================================
CALL OPTION
======================================================================
Call Option Price:     $0.0906

Greeks:
  Delta (Δ):             0.0634
  Gamma (Γ):             0.0364
  Vega (ν):              1.2131 (per 1% vol: 0.0121)
  Theta (Θ):             -2.6680 per year (-0.0073 per day)
  Rho (ρ):               0.1699 (per 1% rate: 0.0017)

======================================================================
PUT OPTION
======================================================================
Put Option Price:      $5.9250

Greeks:
  Delta (Δ):             -0.9366
  Gamma (Γ):             0.0364
  Vega (ν):              1.2131 (per 1% vol: 0.0121)
  Theta (Θ):             -1.2596 per year (-0.0035 per day)
  Rho (ρ):               -3.1201 (per 1% rate: -0.0312)
```

## Dependencies

- `numpy` - Numerical computations
- `plotly` - Interactive 3D visualizations
- `math` - Mathematical functions
- `os` & `webbrowser` - File operations and browser integration

## Notes

- Gamma and Vega are identical for calls and puts at the same strike
- Call Delta is positive (0 to 1), Put Delta is negative (-1 to 0)
- Call Rho is positive, Put Rho is negative
- Theta is typically negative for both (time decay)

## License

This project is open source and available under the MIT License.

