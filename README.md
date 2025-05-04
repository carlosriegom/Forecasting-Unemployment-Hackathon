# Spanish Unemployment Forecasting - Hackathon

## **Table of Contents**

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

## **1. Introduction**

This repository includes the **Python code and technical report** for a project whose goal is to model Spain’s monthly unemployment series and **predict the total number of unemployed people on november 2024**.  The work was carried out as part of a hackathon in the *Machine Learning I* course of the Big‑Data Master’s programme—**and took first place!**

### **Requirements**

* **Python 3.11.9**
* Install all dependencies listed in *requirements.txt*:

  ```bash
  pip install -r requirements.txt
  ```

## **2. Time Series Analysis**

* **Exploratory decomposition** – STL decomposition confirmed the dominant 12‑month seasonality and revealed two major level shifts: the 2008 financial crisis and the 2020–2022 COVID‑19 shock.

* **Outlier handling** – The pandemic peak was condensed into a single χ²‑shaped intervention variable, later used as an exogenous regressor in SARIMAX and NARX models.

* **Seasonal pattern** – Summer unemployment spikes once temporary tourism contracts expire, whereas December records a modest drop thanks to holiday hiring in hospitality, retail and courier services.  Together these effects form the pronounced annual cycle illustrated in Figures 1–2 of the report.

* **Stationarity** – After transformation, one regular difference (d = 1) and one seasonal difference (D = 1, s = 12) were required, as indicated by ADF and KPSS tests plus ACF/PACF diagnostics.

* **Scale adjustment & Box‑Cox transform** – Data were divided by 10⁻⁶ (millions of persons) for numerical stability.  A weak level‑variance relation (R² < 0.4) motivated a Box‑Cox power transform (λ ≈ 0.25), yielding more homogeneous residual variance.

## **3. General Forecast (2001–2024)**

### **3.1 SARIMA**

After applying the Box‑Cox transformation, analysis of the PACF and ACF plots revealed a clear PACF spike at lag 1 and an ACF spike at lag 12, indicating that a parsimonious AR(1) × SMA(1) structure is appropriate for the data.

| (p,d,q)                                                                                         | (P,D,Q,s)     | Residual Check                              |
| ----------------------------------------------------------------------------------------------- | ------------- | ------------------------------------------- |
| (1, 1, 0)                                                                                       | (0, 1, 1, 12) | Ljung‑Box p = 0.67 → white‑noise residuals  |

### **3.2 SARIMAX**

Building on the SARIMA foundation, we integrated the COVID‑19 intervention dummy as an exogenous feature. This addition noticeably improves the model’s responsiveness to pandemic‑era anomalies, reducing forecast errors throughout 2020–2022 and achieving the **lowest test‑window MAPE of 1.06%**.


### **3.3 NARX (MLP)**
To capture non-linear patterns in Spain’s unemployment series, we employed a Non-linear Autoregressive with Exogenous Input (NARX) model implemented as a Multi-Layer Perceptron (MLP). This hybrid architecture leverages both past values of unemployment and the COVID intervention dummy to forecast the next observation.

| Hidden layers | Activation | Solver | Max iter | Tol      |
| ------------- | ---------- | ------ | -------- | -------- |
| (5, 5, 5)     | ReLU       | L-BFGS | 500      | 1 × 10⁻⁵ |

* **Input features:** lagged unemployment at lags t–1, t–2, t–12 and t–24, plus the χ²-shaped COVID dummy.

* **Hyperparameter tuning:** a grid search over network sizes, activation functions and regularisation parameters was conducted using five-fold expanding-window cross-validation. The final (5,5,5) architecture offered the best balance between bias and variance.

* **Residual seasonality correction:** after training the MLP, residuals exhibited a mild 12‑month seasonal pattern. We therefore fitted a compact SARIMA(0,0,0)(0,1,1)<sub>12</sub> model on these residuals and added its forecasts back to the MLP outputs for the final prediction.
This combined approach yielded a **MAPE of 1.30 %**.

### **3.4 Model Comparison**

To evaluate each approach, we compared their test‑window accuracy and highlighted key strengths:

| Model       | Test‑window MAPE (%) | Strengths                                                                       |
| ----------- | -------------------- | ------------------------------------------------------------------------------- |
| **SARIMAX** | **1.06**             | Best overall fit with an interpretable exogenous term capturing pandemic shocks |
| SARIMA      | 1.15                 | Solid baseline performance with minimal complexity                              |
| NARX        | 1.30                 | Captures non-linear dependencies but requires heavier tuning                    |

## Forecast for November 2024

In addition to model-driven forecasts, we considered the potential impact of the DANA (isolated high-altitude depression) that struck Valencia on October 29th. Emergency economic aid for affected freelancers, business owners and workers suggests any increase in unemployment would be delayed from official registration. Consequently, we did not adjust upwards for this event and maintained the SARIMAX prediction below.

So, for the final prediction, we focused on the SARIMAX model due to its accuracy‑parsimony balance:

| Model       | Prediction    | Absolute Error vs INE | Percentage Error |
| ----------- | ------------- | --------------------- | ---------------- |
| **SARIMAX** | **2,582,958** | 3,060                 | **0.12%**        |

> **Official INE figure (December 2024): 2,586,018**

This 0.12% deviation (≈0.1%) underscores the model’s precision and secured our team the top ranking in the course hackathon.

## **Conclusions**

* The **SARIMAX model** demonstrated that combining classical time‑series methods with a single, well‑chosen external regressor can outperform more complex neural architectures in short‑horizon forecasting.
* Introducing the **COVID‑19 dummy** captured pandemic anomalies effectively, reducing average forecast error by nearly 0.1 percentage points compared to the SARIMA baseline.
* Despite its flexibility, the **NARX network** offered diminishing returns relative to its tuning demands, achieving a higher MAPE of 1.30%.
* A **final forecast error of 0.12%** illustrates that careful model specification and light feature engineering remain powerful tools for macro‑economic prediction.

## **Further Details**

For comprehensive methodology, intermediate results and full code execution, please consult the technical report (`Report_Assignment_2_RiegoMozas_VisedoTomas_RodriguezSoria.pdf`) and the Jupyter notebook (`pipeline.ipynb`).

## **Authors**

Jose Carlos Riego Mozas

Ángel Visedo Tomás

Pablo Rodríguez Soria

## **License**

Distributed under the **MIT License** – you are free to use, modify and distribute the code provided that attribution to the original authors is preserved.
