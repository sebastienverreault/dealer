# Price Service
## Overview
A system aware service to provide the bid and ask for balance exchange between BTC and local currency.

It allows for adjusting the exchange rate depending on the conditions of the system.

It provides price quotes for immediate transactions as well as option like locked-in exchange rate for settlement upon reception of the funds at a later time.

## Interface

### Immediate USD Buy
```
GetCentsFromSatsForImmediateBuy(amount: Satoshis): USD
```

This interface takes an amount of BTC in sats and returns the equivalent amount of USD using an internal exchange rate.

#### Internal Exchange Rate
Since the balances are only records in the ledger, and that the USD is ultimately hedged by the Dealer, which in turn uses a market exchange (OKEx) to do so,
it make sense to use at the maximum the **bid** from the exchange directly, apply the transaction fee (ex.: 0.05%) and then some discretionary fees.




### Immediate USD Sell (BTC Buy)
```
GetSatsFromCentsForImmediateSell(amount: USD): Satoshis
```

This interface takes an amount of USD in cents and returns the equivalent amount of BTC in sats using an internal exchange rate.

#### Internal Exchange Rate
For the same reasons as explained [above](./PRICE-SERVICE.md#Immediate-USD-Buy),
it make sense to use at the maximum the **bid** from the exchange directly, apply the transaction fee (ex.: 0.05%) and then some discretionary fees.



### Future USD Buy
```
GetCentsFromSatsForFutureBuy(amount: Satoshis, time-to-maturity: Seconds): USD
```

This interface takes an amount of BTC in sats and returns the equivalent amount of USD using an internal exchange rate.

#### Internal Exchange Rate
For the same reasons as explained [above](./PRICE-SERVICE.md#Immediate-USD-Buy),
and the fact that the rate is agreed upon now for a future settlement, there's an extra component added, namely and optionality spread.

##### Optionality Spread
An invoice that fixes the exchange rate now for reception of the agreed upon funds at a later time, presents an opportunity for abuse.

Ex. 
- Create an invoice for max time-to-maturity
- watch the market
    - if profitable exercise the invoice and liquidate
    - if not let it expire
- Repeat

This type of invoice is indirectly a Call Option i.e.: a contract that gives the right, but not the obligation, to buy an asset (USD) at a specified price (BTC) within a specific time period.

So such a contract has a financial value that in this case is given for free and therefore can be exploited.

To counterbalance that, one can determine the expected return of the option at the time of creation given the market conditions 
and modify it so the expected return is zero, i.e. skew the exchange rate in disfavor of the buyer to nullify the arbitrage opportunity.
In doing so it becomes a zero sum game instead of slightly positive for the buyer.

It would also make sense if the contract was not honored in case of extreme divergence between the exchange rate agreed upon and the one prevailing in the market at the time of execution.

##### Typical value of the option
###### Parameter used for the Option
|Parameter|Value|
|---------|:---:|
|Spot Price|30,000 USD|
|Strike Price|30,000 USD|
|Annualized 30-day Price Volatility|116.62%|
|Risk-free interest rate|0%|
|Time to maturity|2 minutes|

|Call Option Value|Probability of exercise|
|-|-|
|19.45 USD|50%|


###### Distribution of returns

###### Volatility Parameter
Using historical price data, a volatility estimate can be determined empirically assuming log normal price, i.e. normal returns.
The question is how many samples and at what frequency.

- ex. 1. 30 days of daily "close" prices
- ex. 2. 90 past observation of the 2 minutes "close" prices

The series is transformed into log returns and the variance of the distribution estimated.
The volatility is the standard deviation estimate +/- the standard error.

In the Price Service's case, the worse the volatility estimate the more conservative the estimate of the risk associated with the above free option.

So one could look for the maximum annualized volatility from a few different estimation methods and use that static value, refresh it regularly or go for a dynamic live calculation.

Taking a recent 30-day annualized volatility figure of 116.62%, the 2-min volatility can be calculated as 2min Vol = 0.22741%.

```
    Vol_annual = Vol_2-min * sqrt(number of 2-min per year)

    Vol_2-min = Vol_annual / sqrt(365.25*24*60/2) = Vol_annual / sqrt(262980)
    Vol_2-min = Vol_annual / sqrt(262980)
    Vol_2-min = 116.62% / sqrt(262980)
    Vol_2-min = 0.22741%
```

Now that 2-min volatility means that there's a certain probability that within 2-min,
the price of BTC could stay within +/- 1x that volatility percentage,
or to the corollary move outside +/- 1x that percentage.

There's another probability for the 2x range, 3x, etc., the so called sigma levels.

So if we look at the probability of the price of BTC venturing outside a certain sigma level assuming a more fitting distribution (Laplace) than the Normal.

|Outside +/- the sigma level range|Laplace expected frequency|Normal expected frequency|Price change|Loss @ 30k USD/BTC|
|-|-|-|-|-|
|1 sigma|76x / year|80x / year|> +/- 0.22741%|>68.22  USD|
|2 sigma|23x / year|12x / year|> +/- 0.45482%|>136.45 USD|
|3 sigma|7x / year|0.7x / year|> +/- 0.68223%|>204.67 USD|
|4 sigma|2x / year|0.016x c/ year|> +/- 0.90964%|>272.89 USD|
|5 sigma|7x / 10 years|0.0014x / 10 years|> +/- 1.13705%|>341.11 USD|
|6 sigma|2x / 10 years|10^-6x / 10 years|> +/- 1.36446%|>409.34 USD|
|7 sigma|6x / 100 years|10^-8x / 100 years|> +/- 1.59187%|>477.56 USD|

So if we look at the probability of the price of BTC venturing outside a certain sigma level

	- price change of 1 sigma within 2min = 30,000 * 1 * 0.22741 / 100 = 68.22  // i.e. 1 * 0.22741 = 0.22741%
	- price change of 2 sigma within 2min = 30,000 * 2 * 0.22741 / 100 = 136.45	// i.e. 2 * 0.22741 = 0.45482%
	- price change of 3 sigma within 2min = 30,000 * 3 * 0.22741 / 100 = 204.67	// i.e. 3 * 0.22741 = 0.68223%
	- price change of 4 sigma within 2min = 30,000 * 4 * 0.22741 / 100 = 272.89	// i.e. 4 * 0.22741 = 0.90964%
	- price change of 5 sigma within 2min = 30,000 * 5 * 0.22741 / 100 = 341.11	// i.e. 5 * 0.22741 = 1.13705%
	- price change of 6 sigma within 2min = 30,000 * 6 * 0.22741 / 100 = 409.34	// i.e. 6 * 0.22741 = 1.36446%
	- price change of 7 sigma within 2min = 30,000 * 7 * 0.22741 / 100 = 477.56	// i.e. 7 * 0.22741 = 1.59187%


### Future USD Sell (BTC Buy)
```
GetExchangeRateForFutureUsdSell(amount: USD, time-to-maturity: Seconds): Satoshis
```

This interface takes an amount of USD and returns the equivalent amount of BTC in sats using an internal exchange rate.

#### Internal Exchange Rate
Same as [above](./PRICE-SERVICE.md#Future-USD-Buy), but using the **ask** as ceiling.
