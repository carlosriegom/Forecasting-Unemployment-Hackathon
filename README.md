# Spanish unemployment forecasting - Hackathon

## Table of Contents

1. [Introduction](#introduction)
2. [Time Series Analysis](#time-series-analysis)
3. [General Forecast (2001–2024)](#general-forecast-2001–2024)

   1. [SARIMA](#31-sarima)
   2. [SARIMAX](#32-sarimax)
   3. [NARX (MLP)](#33-narx-mlp)
   4. [Model Comparison](#34-model-comparison)
4. [Forecast for November 2024](#forecast-for-november-2024)
5. [Conclusions](#conclusions)
6. [License](#license)

## Introduction

This repository includes the **Python code and technical report** for a project whose goal is to model Spain’s monthly unemployment series and **predict the total number of unemployed people one month ahead**.  The work was carried out as part of a hackathon in the *Machine Learning I* course of the Big‑Data Master’s programme—**and took first place!**

### Requirements

* **Python 3.11.9**
* Install all dependencies listed in *requirements.txt*:

  ```bash
  pip install -r requirements.txt
  ```
* Tested on Windows 11 and Ubuntu 22.04, but any OS with Python 3.11 should work.

## Time Series Analysis

* **Exploratory decomposition** revealed a clear 12‑month seasonal pattern plus two structural shocks: the 2008 financial crisis and the 2020 – 2022 COVID‑19 period .
* **Stationarity** – One regular difference (d = 1) and one seasonal difference (D = 1, s = 12) were applied after inspecting ACF/PACF plots .
* **Outlier handling** – The pandemic spike was condensed into a single χ²‑shaped intervention variable that later served as an exogenous regressor .
* **Lag selection** – Significant PACF spike at lag 1 and ACF spike at lag 12 suggested a parsimonious AR(1) × SMA(1) structure.

## General Forecast (2001–2024)

### 3.1 SARIMA

| (p,d,q)                                                                                         | (P,D,Q,s)     | Residual Check                              |
| ----------------------------------------------------------------------------------------------- | ------------- | ------------------------------------------- |
| (1, 1, 0)                                                                                       | (0, 1, 1, 12) | Ljung‑Box p = 0.67 → white‑noise residuals  |
| The model captures short‑term persistence (AR 1) and seasonal moving‑average behaviour (SMA 1). |               |                                             |

### 3.2 SARIMAX

Adds the COVID intervention dummy to the SARIMA backbone.  The exogenous term sharpens the fit during 2020–2022 and yields the **lowest test‑window MAPE of 1.06 %** .

### 3.3 NARX (MLP)

| Hidden layers | Activation | Solver | Max iter | Tol      |
| ------------- | ---------- | ------ | -------- | -------- |
| (5, 5, 5)     | ReLU       | L‑BFGS | 500      | 1 × 10⁻⁵ |

Inputs: y<sub>t‑1</sub>, y<sub>t‑2</sub>, y<sub>t‑12</sub>, y<sub>t‑24</sub> plus the COVID dummy.  Hyper‑parameter grid defined in the report ; selected via expanding‑window CV.  Remaining seasonal autocorrelation in the residuals was removed by fitting a minimalist SARIMA(0,0,0)(0,1,1)<sub>12</sub> model .

### 3.4 Model Comparison

| Model       | Test‑window MAPE (%) | Key Strength                                 |
| ----------- | -------------------- | -------------------------------------------- |
| **SARIMAX** | **1.06**             | Interpretable exogenous shock, best accuracy |
| SARIMA      | 1.15                 | Robust baseline                              |
| NARX        | 1.30                 | Captures potential nonlinearities            |

## Forecast for November 2024

| Model       | Predicted unemployed | Abs. Error vs INE | % Error     |
| ----------- | -------------------- | ----------------- | ----------- |
| **SARIMAX** | **2 582 958**        | 3 060             | **0.12 %**  |

> **Official INE figure:** 2 586 018 (published December 2024)

The report focuses on the SARIMAX estimate—which deviated by only **0.12 %** (≈ 0.1 %) from reality and secured the top spot in the classroom hackathon.

## Conclusions

* Carefully specified **seasonal ARIMA models, enhanced with a single domain‑specific regressor, can outperform heavier neural approaches** over short horizons.
* Encoding the pandemic as a χ² pulse delivered a tangible accuracy gain without sacrificing interpretability.
* The final deviation of **0.12 %** demonstrates that statistical rigour plus light feature engineering suffices for month‑ahead macro‑economic forecasting.

## License

Distributed under the **MIT License** – you are free to use, modify and distribute the code provided that attribution to the original authors is preserved.
