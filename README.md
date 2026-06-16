# Renewable Asset Valuation & Hedging Strategy (France 2023)

Reconstructing the P&L of a 10 MW wind/solar portfolio from one full year of hourly RTE generation and EPEX SPOT Day-Ahead data, then finding the PPA/spot hedge ratio that gives the best risk-adjusted return.

![Risk-adjusted return vs hedge ratio](hedging_frontier.png)

## Summary

The question I wanted to answer is the one a portfolio manager actually faces: how much of a renewable asset's volume should be locked in a PPA, and how much left exposed to the spot market?

Main results for 2023 (France):

* Average baseload spot: 96.9 EUR/MWh. Solar capture price: 86.0 EUR/MWh (cannibalisation of 11.3%). Wind capture price: 89.6 EUR/MWh (7.6%).
* 147 hours of negative prices, mostly during high-renewable, low-demand periods.
* Demand/price correlation of only 33%, which confirms that in 2023 the price was set mainly by the marginal fuel cost (gas), not by French load on its own.
* The risk/return curve is concave with an interior optimum: around 80% PPA for wind, around 40% for solar. A blended hedge near 70% PPA cuts wind revenue volatility by 30% while keeping spot exposure to scarcity pricing.

## Why this matters

A renewable asset produces the most when its own technology is flooding the grid and pushing prices down, so it is structurally exposed to cannibalisation. The real problem for a producer is not the volume of energy produced but how much risk-adjusted revenue can be secured without giving away the upside during scarcity events. This repo treats the asset as a trading book rather than an engineering spec sheet.

## Method

1. Data engineering (ETL). Loaded raw RTE generation (15 min) and EPEX SPOT Day-Ahead prices. Handled the separator and encoding ambiguity, removed artificial zero-production values by linear interpolation, resampled production to hourly, and inner-joined it to prices into a single hourly dataset.

2. System fundamentals. French load is 32% higher in January than in July, which is the structural driver of winter price tension. I also plotted the daily generation mix (nuclear, hydro, renewables) against demand.

3. Peak-load stress test. The annual peak was 83.8 GW on 23 January 2023 at 19:00, clearing at 223 EUR/MWh. Renewables covered only 9.9% of that peak, so the price was set by the residual load (demand minus renewables), which is what the marginal thermal plant has to serve. This is the merit-order logic made concrete.

4. Capture prices and cannibalisation. Volume-weighted capture price per technology against the simple baseload average, computed monthly and annually, to quantify the discount each technology suffers from producing in its own low-price hours.

5. Hedging and risk optimisation. National generation was normalised to a 10 MW asset (implied load factors of 19% for solar and 32% for wind, both consistent with the real French fleet). I then swept the full hedge ratio from 0 to 100% PPA and, for each level, computed the mean and standard deviation of monthly revenue and their ratio as a risk-adjusted return measure. PPA strikes assumed: 65 EUR/MWh for solar, 75 EUR/MWh for wind.

## Findings

Cannibalisation was real but moderate in 2023. Solar lost 11.3% against baseload, wind 7.6%. The gap is smaller than in a normal year because 2023 prices were still high and volatile from the tail of the 2022 gas crisis, which lifted all capture prices.

Wind beat solar on every strategy. Wind generation lines up with high-priced winter demand peaks, while solar produces in the cheaper midday window. Across merchant, PPA and mixed strategies, wind revenue came out roughly 1.0 MEUR per year ahead on a 10 MW-equivalent basis.

There is no single best hedge ratio. The risk-adjusted return curve is concave with an interior maximum. Over-hedging kills the scarcity upside, under-hedging leaves the book too volatile. In 2023 the wind optimum sat around 80% PPA (the curve is flat between 60 and 90%, so 70% is close to optimal and simpler to run), and the solar optimum around 40%. A practical 70% PPA / 30% spot book on the wind asset cuts revenue volatility by 30% while keeping merchant exposure to cold-snap spikes. The point is the shape of the trade-off, not a magic number.

## Limitations

* Single year (2023), so the results depend on the price regime. This is not a walk-forward backtest across several environments.
* The 10 MW asset uses the national fleet profile as a proxy, with no site-specific shape or curtailment.
* PPA strikes are fixed assumptions, not tenor or shape-adjusted prices.
* Day-Ahead only, with no intraday or imbalance settlement.

## Tech stack

Python, Pandas (time series), NumPy (vectorised P&L), Matplotlib.
Data: RTE Open Data (generation and load), EPEX SPOT Day-Ahead (ENTSO-E Transparency Platform).

## Author

Yanis Allas. [LinkedIn](https://www.linkedin.com/in/yanis-allas-5600b4294/)
