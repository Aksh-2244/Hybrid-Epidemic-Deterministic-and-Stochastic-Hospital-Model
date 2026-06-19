# Hybrid SIHR Epidemic Model — Deterministic & Stochastic Framework

> **MATH F420 Group Project | Academic Year 2025–2026**  
> Akshat Sangal · Prakhar Kunwar · Varun Chaudhary · Mohit Rajvir · Raaj Verma  
> BITS Pilani

---

## Overview

This project develops a **hybrid epidemic model** that combines stochastic and deterministic methods to realistically simulate disease spread and hospital utilization during a pandemic.

The model extends the classic SIR framework into an **SIHR (Susceptible → Infected → Hospitalized → Recovered)** system. A key feature is a **behavioral feedback mechanism**: as hospitals fill up, the transmission rate β(H) decreases, reflecting real-world public responses like social distancing and lockdowns.

The model is fitted to empirical COVID-19 hospital admission data using **Markov Chain Monte Carlo (MCMC)** simulations, and the full analysis (equilibria, reproduction number, phase portraits, suppression thresholds) is carried out in **MATLAB**.

---

## Table of Contents

- [Background & Motivation](#background--motivation)
- [Model Structure](#model-structure)
- [Key Features](#key-features)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Parameter Reference](#parameter-reference)
- [Results Summary](#results-summary)
- [References](#references)

---

## Background & Motivation

During an epidemic, hospitals face surging patient loads with finite capacity. Standard SIR models do not account for:
- How hospital congestion feeds back into transmission rates
- The stochastic (random) nature of early hospitalization dynamics
- How public behavioral responses dampen disease spread

This project addresses all three by building a framework where:
1. The **hospitalized compartment H(t)** is initialized using a **Continuous-Time Markov Chain (CTMC)**, capturing the randomness of early admissions and discharges.
2. The system then transitions into a **deterministic ODE model**, which is tractable for long-term stability analysis.
3. The **transmission rate β(H)** is a function of hospital load, encoding public awareness and behavioral change.

---

## Model Structure

### The ODE System

$$\frac{dS}{dt} = \Lambda - \beta(H)SI - \mu S$$

$$\frac{dI}{dt} = \beta(H)SI - (\gamma + p + \mu)I$$

$$\frac{dH}{dt} = pI - (u + \mu)H$$

$$\frac{dR}{dt} = \gamma I + uH - \mu R$$

### Transmission Rate with Behavioral Feedback

$$\beta(H) = \beta_0 \left[1 + \alpha\left(\frac{H}{C} - 1\right)^2 - \alpha\right]$$

- When H = 0: β(H) = β₀ (baseline, no public response)
- When H → C: β(H) → β₀(1 − α), i.e., transmission is suppressed by a factor α
- α = 0: no public feedback (worst case)
- α = 0.75: strong public feedback (strong suppression)

### CTMC Initialization for H(t)

The number of hospitalized individuals X(t) is modelled as a birth–death process:
- **Admission rate:** λ(t) = p · I(t)
- **Discharge rate:** μ(X) = u · X

Using Dynkin's formula with f(x) = x, the expected value H(t) = E[X(t)] satisfies exactly the ODE for dH/dt above. This rigorously justifies using the deterministic equation as a macroscopic approximation of the stochastic process.

---

## Key Features

**Equilibrium Analysis**
- Disease-Free Equilibrium (DFE): E₀ = (Λ/μ, 0, 0)
- Endemic Equilibrium (EE): obtained by solving a cubic polynomial in H*, arising from the nonlinear β(H)

**Basic Reproduction Number (R₀)**

Computed via the **Next Generation Matrix (NGM)** method:

$$R_0 = \frac{\beta_0 \Lambda}{\mu(\gamma + p + \mu)}$$

The system transitions from epidemic to endemic behavior at R₀ = 1.

**Analytical Suppression Thresholds**

For β₀ = 0.6 and α = 0.75:
- To achieve a **50% reduction** in transmission → hospitalizations must exceed **42% of capacity**
- To achieve **extreme suppression** → hospitalizations must exceed **67% of capacity**

**MCMC Parameter Fitting**

Baseline transmission rate β₀ and initial infected count I₀ are reverse-engineered from COVID-19 hospital admission CSV data. The short-term (60-day) model uses a closed population (μ = 0) for accuracy. The inferred R₀ from fitted data is **2.79**.

---

## Repository Structure

```
├── matlab/
│   ├── sihr_odes.m              # ODE system definition
│   ├── equilibrium_analysis.m   # DFE and EE computation
│   ├── reproduction_number.m    # NGM method for R0
│   ├── phase_portrait.m         # I vs H phase portraits (R0 > 1 and R0 < 1)
│   ├── alpha_effect.m           # Comparison of α = 0 vs α = 0.75
│   ├── suppression_thresholds.m # Analytical bounds on β(H)
│   └── mcmc_fitting.m           # MCMC simulation for β0 and I0 estimation
├── data/
│   └── covid_hospital_admissions.csv   # Empirical data used for MCMC fitting
├── report/
│   └── Mathematical_Modelling_Project.pdf
└── README.md
```

---

## Getting Started

### Prerequisites

- MATLAB R2021a or later
- Statistics and Machine Learning Toolbox (for MCMC sampling)

### Running the Model

**1. Solve and plot the ODE system:**
```matlab
run matlab/sihr_odes.m
```

**2. Compute equilibria and R₀:**
```matlab
run matlab/equilibrium_analysis.m
run matlab/reproduction_number.m
```

**3. Generate phase portraits:**
```matlab
run matlab/phase_portrait.m
```
Set `R0_mode = 'high'` (R₀ = 2.5) or `R0_mode = 'low'` (R₀ = 0.75) inside the script.

**4. Compare α values:**
```matlab
run matlab/alpha_effect.m
```

**5. Run MCMC fitting on COVID-19 data:**
```matlab
run matlab/mcmc_fitting.m
```
This reads `data/covid_hospital_admissions.csv` and outputs the posterior distribution of β₀ and the best-fit trajectory plot.

---

## Parameter Reference

| Parameter | Symbol | Description |
|-----------|--------|-------------|
| Baseline transmission rate | β₀ | Rate of disease spread in the absence of behavioral response |
| Behavioral feedback strength | α | How strongly public awareness suppresses transmission (0 = none, 1 = full) |
| Healthcare capacity | C | Scaling constant representing maximum hospital capacity |
| Recovery rate (infected) | γ | Rate at which infected individuals recover without hospitalization |
| Hospitalization rate | p | Rate at which infected individuals are admitted to hospital |
| Discharge/recovery rate | u | Rate at which hospitalized individuals leave hospital |
| Birth rate | Λ | Constant inflow of new susceptibles |
| Natural death rate | μ | Background mortality rate (set to 0 for short-term 60-day model) |

---

## Results Summary

| Analysis | Key Result |
|----------|-----------|
| Disease-Free Equilibrium | Stable when R₀ < 1 |
| Endemic Equilibrium | Exists when R₀ > 1 (positive root of cubic in H*) |
| Basic Reproduction Number | R₀ = β₀Λ / μ(γ + p + μ) |
| 50% transmission reduction | Requires H > 0.42 × C |
| Extreme suppression | Requires H > 0.67 × C |
| MCMC-inferred R₀ (COVID-19 data) | **2.79** (60-day surge window) |
| Minimum achievable β (α = 0.75, β₀ = 0.6) | **0.15** |

### Phase Portrait Behaviour

- **R₀ > 1:** Trajectories in the I–H plane converge to the Endemic Equilibrium. Disease persists.
- **R₀ < 1:** All trajectories converge to the Disease-Free Equilibrium. The epidemic dies out.

### Effect of α (Public Awareness)

- **α = 0:** Infected and hospitalized populations peak sharply and remain elevated for a long time.
- **α = 0.75:** Peaks are significantly lower and occur earlier; the system stabilizes faster. This illustrates the concrete epidemiological value of public health communication and behavioral change.

---

## Modeling Notes

**Short-term vs Long-term modeling:**

For the 60-day MCMC fitting, vital dynamics (birth/death) are dropped (μ = 0) because demographic rates (~0.000039/day) are negligible compared to epidemic rates over two months. This gives a much cleaner parameter fit.

For long-term (endemic) analysis, μ > 0 is necessary — without a constant influx of new susceptibles, the susceptible pool artificially drains to zero and the model is forced into a disease-free state, which is not realistic for endemic diseases.

---

## References

1. Allen, L. J. S. (2010). *An Introduction to Stochastic Processes with Applications to Biology* (2nd ed.). CRC Press.
2. Diekmann, O., Heesterbeek, J. A. P., & Britton, T. (2013). *Mathematical Tools for Understanding Infectious Disease Dynamics*. Princeton University Press.
3. van den Driessche, P., & Watmough, J. (2002). Reproduction numbers and sub-threshold endemic equilibria for compartmental models of disease transmission. *Mathematical Biosciences*, 180(1–2), 29–48.
4. Funk, S., Salathé, M., & Jansen, V. A. (2010). Modelling the influence of human behaviour on the spread of infectious diseases: a review. *Journal of the Royal Society Interface*, 7(49), 1247–1256.
5. Capasso, V., & Serio, G. (1978). A generalization of the Kermack-McKendrick deterministic epidemic model. *Mathematical Biosciences*, 42(1–2), 43–61.

---

*MATH F420 — Mathematical Modelling | BITS Pilani | 2025–2026*



