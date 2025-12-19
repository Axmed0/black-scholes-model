
# Black–Scholes (Call + Put), Greeks, and Two Interactive 3D Surfaces (Plotly → Browser)

In this notebook I implement Black–Scholes pricing and Greeks, then generate **two interactive 3D surfaces** (opened in my browser) with a **dropdown menu** to switch what’s displayed.

---

## What my code does

- Prices a **European call** and **European put** using a `blackscholes(...)` function
- Computes **Greeks** using a `bs_greeks(...)` function:
  - Delta (call + put)
  - Gamma
  - Vega
  - Theta
  - Rho
- Builds **two interactive 3D Plotly surface plots** and saves/opens a single HTML page in the browser:
  - **Surface A:** Spot **S** vs Volatility **σ** (with **T fixed**)
  - **Surface B:** Spot **S** vs Time to expiry **T** (with **σ fixed**)

---

## Inputs I use

I define these parameters in the notebook:

- `S` = spot price (e.g. 34.03)
- `K` = strike price (e.g. 40.00)
- `r` = risk-free rate (e.g. 0.0412)
- `T` = time to expiry (in years, e.g. 30/365)
- `sigma` = volatility (annualised, e.g. 0.35)

```python
S = 34.03
r = 0.0412
K = 40.00
T = 30/365
sigma = 0.35
````

---

## How to run

If `pip` isn’t available as a standalone command, I install dependencies using:

```bash
python -m pip install numpy plotly jupyterlab notebook
```

Then I launch Jupyter:

```bash
python -m jupyterlab
```

---

# The maths (exactly what my functions implement)

## Black–Scholes d₁ and d₂

Inside my pricing and Greeks code I compute:

$$
d_1 = \frac{\ln(S/K) + (r + \tfrac{1}{2}\sigma^2)T}{\sigma\sqrt{T}},
\qquad
d_2 = d_1 - \sigma\sqrt{T}
$$

Code snippet:

```python
d1 = (np.log(S/K) + (r + 0.5*sigma**2)*T) / (sigma*np.sqrt(T))
d2 = d1 - sigma*np.sqrt(T)
```

---

## Normal CDF N(x) (via erf)

In my code I compute the standard normal CDF using the error function:

$$
N(x) = \frac{1}{2}\left(1 + \operatorname{erf}\left(\frac{x}{\sqrt{2}}\right)\right)
$$

Code snippet:

```python
from math import erf, sqrt

def N(x):
    return 0.5*(1 + erf(x/sqrt(2)))
```

For surfaces (NumPy arrays) I vectorise `erf`:

```python
import math
erf_vec = np.vectorize(math.erf)

def N_vec(x):
    return 0.5*(1.0 + erf_vec(x/np.sqrt(2.0)))
```

---

## Black–Scholes prices

### Call price

$$
C = S,N(d_1) - K e^{-rT} N(d_2)
$$

### Put price

$$
P = K e^{-rT} N(-d_2) - S N(-d_1)
$$

Code snippet (my pricing function):

```python
def blackscholes(r, S, K, T, sigma, type="C"):
    d1 = (np.log(S/K) + (r + 0.5*sigma**2)*T) / (sigma*np.sqrt(T))
    d2 = d1 - sigma*np.sqrt(T)

    from math import erf, sqrt
    def N(x):
        return 0.5*(1 + erf(x/sqrt(2)))

    if type.upper() == "C":
        price = S*N(d1) - K*np.exp(-r*T)*N(d2)
    elif type.upper() == "P":
        price = K*np.exp(-r*T)*N(-d2) - S*N(-d1)
    else:
        raise ValueError("type must be 'C' or 'P'")

    return price
```

---

# Greeks (what `bs_greeks(...)` computes)

## Standard normal PDF n(x)

$$
n(x) = \frac{1}{\sqrt{2\pi}}e^{-x^2/2}
$$

Code snippet:

```python
from math import exp, pi, sqrt

def n(x):
    return (1/sqrt(2*pi)) * exp(-0.5*x*x)
```

---

## Delta (Δ)

* Call delta:
  $$
  \Delta_C = N(d_1)
  $$

* Put delta:
  $$
  \Delta_P = N(d_1) - 1
  $$

---

## Gamma (Γ)

$$
\Gamma = \frac{n(d_1)}{S\sigma\sqrt{T}}
$$

---

## Vega (ν)

$$
\nu = S,n(d_1)\sqrt{T}
$$

When I print “vega per 1% vol”, I divide by 100:

$$
\nu_{\text{per 1%}} = \frac{\nu}{100}
$$

---

## Theta (Θ)

My code computes theta **per year**.

* Call theta:
  $$
  \Theta_C = -\frac{S n(d_1)\sigma}{2\sqrt{T}} - rK e^{-rT}N(d_2)
  $$

* Put theta:
  $$
  \Theta_P = -\frac{S n(d_1)\sigma}{2\sqrt{T}} + rK e^{-rT}N(-d_2)
  $$

When I print “theta per day”, I divide by 365:

$$
\Theta_{\text{per day}} = \frac{\Theta}{365}
$$

---

## Rho (ρ)

* Call rho:
  $$
  \rho_C = KT e^{-rT}N(d_2)
  $$

* Put rho:
  $$
  \rho_P = -KT e^{-rT}N(-d_2)
  $$

When I print “rho per 1% rate”, I divide by 100:

$$
\rho_{1%} = \frac{\rho}{100}
$$

---

## Greeks function (code)

```python
def bs_greeks(r, S, K, T, sigma, type="C"):
    d1 = (np.log(S/K) + (r + 0.5*sigma**2)*T) / (sigma*np.sqrt(T))
    d2 = d1 - sigma*np.sqrt(T)

    from math import erf, sqrt, exp, pi

    def N(x):
        return 0.5*(1 + erf(x/sqrt(2)))

    def n(x):
        return (1/sqrt(2*pi)) * exp(-0.5*x*x)

    if type.upper() == "C":
        delta = N(d1)
        theta = -(S*n(d1)*sigma)/(2*sqrt(T)) - r*K*exp(-r*T)*N(d2)
        rho   = K*T*exp(-r*T)*N(d2)
    elif type.upper() == "P":
        delta = N(d1) - 1
        theta = -(S*n(d1)*sigma)/(2*sqrt(T)) + r*K*exp(-r*T)*N(-d2)
        rho   = -K*T*exp(-r*T)*N(-d2)
    else:
        raise ValueError("type must be 'C' or 'P'")

    gamma = n(d1) / (S*sigma*sqrt(T))
    vega  = S*n(d1)*sqrt(T)

    return {"d1": d1, "d2": d2, "delta": delta, "gamma": gamma, "vega": vega, "theta": theta, "rho": rho}
```

---

# Interactive 3D surfaces

## Surface A: S vs σ (T fixed)

In my code I fix time:

```python
T_fixed = 30/365
```

And I create the grid:

```python
S_vals1  = np.linspace(0.5*K, 1.5*K, 80)
sig_vals = np.linspace(0.05, 1.00, 80)
Sg1, Sig = np.meshgrid(S_vals1, sig_vals)
```

Across that grid I compute:

$$
d_1(S,\sigma) = \frac{\ln(S/K) + (r + \tfrac{1}{2}\sigma^2)T_{\text{fixed}}}{\sigma\sqrt{T_{\text{fixed}}}},
\qquad
d_2(S,\sigma) = d_1(S,\sigma) - \sigma\sqrt{T_{\text{fixed}}}
$$

Then I generate surfaces for call/put prices and Greeks (delta/gamma/vega).

---

## Surface B: S vs T (σ fixed)

In my code I fix volatility:

```python
sigma_fixed = 0.35
```

And I create the grid (starting at 1/365 to avoid T = 0):

```python
S_vals2 = np.linspace(0.5*K, 1.5*K, 80)
T_vals  = np.linspace(1/365, 365/365, 80)
Sg2, Tg = np.meshgrid(S_vals2, T_vals)
```

Across that grid I compute:

$$
d_1(S,T) = \frac{\ln(S/K) + (r + \tfrac{1}{2}\sigma_{\text{fixed}}^2)T}{\sigma_{\text{fixed}}\sqrt{T}},
\qquad
d_2(S,T) = d_1(S,T) - \sigma_{\text{fixed}}\sqrt{T}
$$

Then I generate surfaces for call/put prices and Greeks.

---

# Dropdown + browser output (Plotly → HTML)

I create Plotly `Surface` traces for each metric (Call Price, Put Price, Delta, Gamma, Vega), and use a dropdown menu to toggle visibility.

Finally I export both figures into one HTML page and open it:

```python
html1 = pio.to_html(fig_vol, include_plotlyjs="cdn", full_html=False)
html2 = pio.to_html(fig_time, include_plotlyjs=False, full_html=False)

page = f"""
<html>
<head><meta charset="utf-8"><title>Option Surfaces</title></head>
<body>
<h2>3D Surface 1: Spot vs Volatility (T fixed)</h2>
{html1}
<hr/>
<h2>3D Surface 2: Spot vs Time (σ fixed)</h2>
{html2}
</body>
</html>
"""

filename = "option_surfaces_two_graphs.html"
with open(filename, "w", encoding="utf-8") as f:
    f.write(page)

webbrowser.open("file:///" + os.path.abspath(filename))
```

---

## Libraries I use (exactly what’s in my code)

* `numpy`
* `math`
* `plotly.graph_objects`
* `plotly.io`
* `os`
* `webbrowser`

```


* `plotly.io`
* `os`
* `webbrowser`

