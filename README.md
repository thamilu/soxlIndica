# ⚡ Institutional SMC Pro v4.0 — Smart Money & Adaptive Liquidity Model

![Pine Script v6](https://img.shields.io/badge/Pine%20Script-v6-blue.svg)
![TradingView](https://img.shields.io/badge/TradingView-Indicator-00897B.svg)
![Market Focus](https://img.shields.io/badge/Focus-SOXL%20%2F%20Semiconductors%20%2F%20ETFs-orange.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

**Institutional SMC Pro v4.0** is an advanced, institutional-grade Pine Script v6 indicator and multi-horizon quantitative forecasting engine built for **TradingView**. 

Originally engineered with a specialized focus on leveraged semiconductor instruments like **SOXL** (and applicable across equities, indices, and crypto), this tool combines **Smart Money Concepts (SMC)**, **Order Flow & Liquidity Profiling**, an **Adaptive Multi-Horizon Forecast Engine**, and a **Real-Time HUD Dashboard** to deliver actionable, data-backed trade setups.

---

## 📌 Table of Contents
- [Key Features](#-key-features)
- [Architecture & Core Engines](#-architecture--core-engines)
- [Dashboard & HUD Modes](#-dashboard--hud-modes)
- [Installation & Setup](#-installation--setup)
- [Project Structure](#-project-structure)
- [Documentation & Glossary](#-documentation--glossary)
- [Disclaimer](#-disclaimer)

---

## 🚀 Key Features

### 🔹 1. Smart Money Concepts (SMC) & Structural Analysis
- **Break of Structure (BOS) & Change of Character (CHoCH)** across multi-timeframes (HTF 1D, 4H, 1H).
- **Institutional Order Blocks (OB)** and **Fair Value Gaps (FVG)** with dynamic mitigation tracking.
- **Liquidity Pools & Sweeps**: Dynamic detection of buy-side/sell-side liquidity traps and stop hunts.
- **Opening Range Breakout (ORB)** tracking for 15-minute and 30-minute session windows with live projected bounds.

### 🔹 2. Adaptive Multi-Horizon Forecast Engine
- **Chained Hourly Ladder (+30m → +6H)**: Sequential price prediction path with probability confidence bounds.
- **Multi-Session Outlook**: Real-time targets for Pre-Market (**Pre**), Regular Session (**Reg**), Post-Market (**Post**), Overnight (**Ovn**), and Next-Day (**NxtDay**).
- **Dual Probability Metrics**:
  - **Conf (Confidence)**: Model's internal signal-to-noise ratio adjusted for volatility clustering and VIX regimes.
  - **CalP (Calibrated Probability)**: Online-fitted logistic (Platt calibration) win probability backed by empirical walk-forward resolutions.

### 🔹 3. SOXL & Leveraged ETF Quantitative Mechanics
- **Top 10 Holdings Basket Engine**: Real-time weighted tracking of top semiconductor constituents (NVDA, AMD, MU, INTC, AVGO, AMAT, KLAC, MRVL, LRCX, TSM).
- **Primary Benchmark Alignment**: Integrated tracking against `ICE:ICESEMIT` (NYSE Semiconductor Index).
- **Leverage Decay & Expense Drag Modeling**: Accounting for synthetic 3x daily rebalancing decay and net annual fund expense ratios.

### 🔹 4. Self-Validation & Risk HUD Engine
- **Trade & Next-Day Brier Scoring**: Self-grading accuracy metrics measuring stated probabilities against real binary outcomes.
- **Position Heat & Risk Management**: Live calculation of dollar and account percentage equity at risk based on dynamic stop-loss levels.
- **Dynamic Setup Grading (A+, A, B, C)**: Percentile-based setup evaluation with rolling win rates and expected R-multiples ($ExpR$).
- **Mobile-Friendly UI Mode**: Specialized two-column compact layout formatted for mobile screens and smaller chart viewports.

---

## 📐 Architecture & Core Engines

```
                             ┌─────────────────────────────────────────┐
                             │    Institutional SMC Pro v4.0 Engine    │
                             └────────────────────┬────────────────────┘
                                                  │
         ┌─────────────────────────┬──────────────┴──────────────┬─────────────────────────┐
         ▼                         ▼                             ▼                         ▼
┌──────────────────┐     ┌──────────────────┐          ┌──────────────────┐      ┌───────────────────┐
│ Smart Money      │     │ Multi-Timeframe  │          │ SOXL Basket      │      │ Multi-Horizon     │
│ Structure Engine │     │ & ORB Analytics  │          │ & Leverage Drag  │      │ Forecast Engine   │
└────────┬─────────┘     └────────┬─────────┘          └────────┬─────────┘      └─────────┬─────────┘
         │                        │                             │                          │
         └────────────────────────┴──────────────┬──────────────┴──────────────────────────┘
                                                 ▼
                                     ┌──────────────────────┐
                                     │ Self-Validation      │
                                     │ (Platt & Brier)      │
                                     └───────────┬──────────┘
                                                 ▼
                                     ┌──────────────────────┐
                                     │ Real-Time HUD        │
                                     │ Dashboard (Desktop/  │
                                     │ Mobile)              │
                                     └──────────────────────┘
```

---

## 🖥️ Dashboard & HUD Modes

The HUD is a real-time dashboard displayed directly on your chart (`barstate.islast`).

### Display Modes:
1. **Compact Mode**: 6 high-level essential rows: Title (`⚡SMC`), Trend (`🚦TREND`), Timing (`⏱TIMING`), Score (`📊SCORE`), Risk (`⚠️RISK`), and Next-Day (`🔮NXT-DAY`).
2. **Expanded Mode (Default)**: Full diagnostic HUD including Session Alignment (`🎯ALIGN`), Validation (`🧪VALID`), Tail-Risk (`☢TAIL`), Opening Range (`ORB15m/30m`), Session Outlook Ladder (`Pre/Reg/Post/Ovn/NxtDay`), and the Hourly Forecast Ladder (`+30m → +6H`).
3. **Mobile View (`Mobile Friendly View: ON`)**: Optimized two-column narrow layout with left-aligned text, preventing table overflow on mobile devices.

---

## 🛠️ Installation & Setup

1. **Open TradingView**: Navigate to any stock, index, or ETF chart (e.g., `SOXL`, `NVDA`, `QQQ`).
2. **Open Pine Editor**: In the bottom tab of TradingView, click on **Pine Editor**.
3. **Copy Code**: Open [`mobile view/Institutional_SMC_Pro.pine`](file:///g:/prompt/trade%20-%20compact%20mobile%20view%20base%20previous%20day/mobile%20view/Institutional_SMC_Pro.pine) from this repository, copy all code, and paste it into the Pine Editor.
4. **Save & Apply**: Click **Save**, name your script `Institutional SMC Pro v4.0`, and click **Add to Chart**.
5. **Configure Settings**:
   - Set **Display Mode** (`Beginner`, `Intermediate`, `Expert`).
   - Toggle **Mobile Friendly View** if viewing on mobile devices.
   - Adjust **Asset Leverage Factor** (e.g., `3.0` for SOXL/TQQQ, `1.0` for SPY/NVDA).
   - Toggle Economic Event Flags (FOMC, CPI, Earnings) during high-volatility release days.

---

## 📂 Project Structure

```
.
├── README.md                                # Root Project Documentation
└── mobile view/
    ├── Institutional_SMC_Pro.pine           # Pine Script v6 Source Code (v4.0 Engine)
    └── Trading_Dashboard_Guide.md           # Comprehensive Terminology & Dashboard Reference Guide
```

- [`mobile view/Institutional_SMC_Pro.pine`](file:///g:/prompt/trade%20-%20compact%20mobile%20view%20base%20previous%20day/mobile%20view/Institutional_SMC_Pro.pine): Complete source code for the Pine Script v6 indicator.
- [`mobile view/Trading_Dashboard_Guide.md`](file:///g:/prompt/trade%20-%20compact%20mobile%20view%20base%20previous%20day/mobile%20view/Trading_Dashboard_Guide.md): Detailed reference guide explaining all terms, calculations, base-price conventions, and timing messages.

---

## 📖 Documentation & Glossary

For complete details on row definitions, noise-floor math, base-price calculations, and signal timing statuses, refer to the full [`Trading_Dashboard_Guide.md`](file:///g:/prompt/trade%20-%20compact%20mobile%20view%20base%20previous%20day/mobile%20view/Trading_Dashboard_Guide.md).

### Quick Terminology Cheat Sheet:
| Term | Meaning |
| :--- | :--- |
| **Conf** | Model internal directional confidence score (signal-to-noise ratio). |
| **CalP** | Calibrated empirical win probability (Platt-calibrated logistic transform). |
| **ExpR** | Mean historical realized R-multiple for setups at a given letter grade ($ExpR: +0.42 \pm 0.15$). |
| **Brier Score** | Mean squared calibration error (0.25 = uninformative baseline, 0 = perfect calibration). |
| **Heat** | Dynamic live dollar amount and percentage of account equity at risk at current stop level. |

---

## ⚠️ Disclaimer

*This indicator and repository are for educational and research purposes only. Nothing contained herein constitutes financial, investment, or trading advice. Algorithmic and leveraged ETF trading carries substantial risk of loss. Always perform your own due diligence and test thoroughly on paper trading environments before executing live capital.*
