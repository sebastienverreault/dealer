# Ledger Side Profit and Loss

## Intro

The users' BTC and USD balances are numbers in the bitcoin bank's ledger.

The bank is in "physical" possession of the bitcoin, as custodian, but only offers synthetic USD thru hedging.

If not for hedging the bank would expose itself to the variations of the BTC price which could mean capital losses.
(In [Example 1.](./USER_PNL.md#ex-1) below, the 10% profit of the user would create a default situation for the bank 
if not for hedging: the user would be owed 1.1 BTC while the bank would only have 1 BTC in reserve.)

So hedging allows for riskless (less risk?!) USD synthetic.

If there were no hedging costs (trading fees, funding fees, etc.), 
the bank would have no profits nor losses from the activity 
which would mean offering synthetic USD would also have no PnL/cost/downside/risk/etc.

Only the user would incur PnL associated with its own conversion activities.

Given the hedging costs are non-null and therefore the hedging imperfect,
a quantitative measure of how effective the hedge position covers the spot position in relation to the BTC price, is the unrealized profit and loss (uPnL).

## Overall Unrealized Profit and Loss

The overall Unrealized PnL is the sum of the PnLs of the spot position recorded in the ledger by the Galoy backend
and the swap position taken on the exchange by the Dealer.

The spot position (BTC & USD balances) is the aggregate of all users' conversions as a whole.
Each conversion's exchange rate is implicit in the position and therefore averaged.
So the ratio of the ledger's USD balance over BTC balance is the dynamic average open price of the spot position.

- The Spot Unrealized PnL from the bank's point of view, is calculated as:
    ``` 
    Spot PnL in USD = Balance BTC * (CurrentBtcPrice - AverageOpenPrice) USD/BTC
    AverageOpenPrice = Balance USD / Balance BTC
    ``` 
- The Swap Unrealized PnL is calculated as:
    ``` 
    Swap PnL in BTC = ExchangeBalance USD * (CurrentSwapPrice - OpenSwapPrice) BTC/USD
    Swap PnL in USD = Swap PnL in BTC * CurrentBtcPrice USD/BTC
    ``` 
- The Overall Unrealized PnL is calculated as:
    ``` 
    Overall PnL in USD = Balance BTC * (CurrentBtcPrice - AverageOpenPrice) USD/BTC
    AverageOpenPrice = Balance USD / Balance BTC
    ``` 

#### Ex. Spot position and implied exchange rate
##### Exchange rates
|time|exchange rate|
|:-:|:-:|
|t = t0|10,000.00 USD/BTC|
|t > t0|20,000.00 USD/BTC|

##### Sequence of events
|time|User1 BTC balance|User1 USD balance|User2 BTC balance|User2 USD balance|Bank BTC reserve|Bank BTC liability|Bank USD liability|Implied Exchange Rate|Bank Spot Unrealized PnL|
|:-:|-|-|-|-|-|-|-|-|-|
| t \<t0 |1.0 BTC|0.00 USD|1.0 BTC|0.00 USD|2.0 BTC|-|-|-|
| t = t0 |0.0 BTC|10,000.00 USD|1.0 BTC|0.00 USD|2.0 BTC|1.0 BTC|10,000.00 USD|10,000.00 USD/BTC|1.0 BTC * (10k - 10k) USD/BTC = 0.00 USD|
| t0\> t \<t1 |0.0 BTC|10,000.00 USD|1.0 BTC|0.00 USD|2.0 BTC|1.0 BTC|10,000.00 USD|10,000.00 USD/BTC|1.0 BTC * (20k - 10k) USD/BTC = 10,000.00 USD|
| t = t1 |0.0 BTC|10,000.00 USD|0.0 BTC|20,000.00 USD|2.0 BTC|2.0 BTC|30,000.00 USD|15,000.00 USD/BTC|2.0 BTC * (20k - 15k) USD/BTC = 10,000.00 USD|
| t = t2 |0.5 BTC|0.00 USD|0.0 BTC|20,000.00 USD|2.0 BTC|1.5 BTC|20,000.00 USD|13,333.33 USD/BTC|1.5 BTC * (20k - 13.333k) USD/BTC = 10,000.00 USD|
| t = t3 |0.0 BTC|10,000.00 USD|0.5 BTC|10,000.00 USD|2.0 BTC|1.0 BTC|10,000.00 USD|10,000.00 USD/BTC|1.0 BTC * (20k - 10k) USD/BTC = 10,000.00 USD|
| t \>t3 |0.0 BTC|10,000.00 USD|1.0 BTC|0.00 USD|2.0 BTC|0.5 BTC|-|-|-|


## The users' Realized Profit and Loss

Each user's BTC PnL is the sum of all BTC -> USD -> BTC round trip conversions.
It is only perfectly visible to the Galoy backend and not to the Dealer.
The Dealer only sees the aggregate changes in BTC & USD balances.

To demonstrate the above, assume the Dealer only hedges in slices of 100.00 USD 
so absolute USD balance changes in the lower half of that slice (00.00 - 50.00 USD) do not trigger any hedging.
Assume a user was to do sequential BTC -> USD -> BTC round trip conversions in that <50.00 USD range of notional, 
the user would accumulate a realized PnL,
the bank would incur the opposite realized PnL without any hedging realized PnL offset.

## The bank's Realized Profit and Loss

The bank has an opposite and offsetting realized PnL to the users, being the counterpart in the conversion transaction.
As pointed before, the bank's realized PnL in spot is offset by the realized PnL in swap.

The realized PnL in swap is reported by the exchange, is easy to track, is simple to calculate given all transaction history. 
It is an exact number that can be perfectly reconciled.

The exact realized PnL in spot can only be obtained via functionality in the Galoy backend or other data wrangling methods having access to transactional data.
The Dealer does not try to calculate nor infer that figure.


### Trivial case

So for a round trip where the user exchanges 1 BTC to USD at time t0, at a current exchange rate fx(t0), 
then later on, at a time t1, exchanges back the same amount of USD received at time t0 into BTC, at a current exchange rate fx(t1), 
the user ends up in possession of [fx(t0) / fx(t1)] BTC.
The gains and losses are passed directly from and to the exchange via hedging.
The bank is flat. 
This is the simple case, no fees, full round trip (i.e. BTC -> USD -> BTC conversions without leaving the bank's control so no BTC nor USD withdrawals).

#### Ex. 1.
#### Simple case: no bank fees, no hedging fees, no time slippage, no spot vs swap price difference, full round trip, BTC price decreases
##### Exchange rates
|time|exchange rate|
|:-:|:-:|
|t0|10,000.00 USD/BTC|
|t1|9,090.91 USD/BTC|

##### Sequence of events
|time|User BTC balance|User USD balance|spot operation|note|Bank BTC reserve|Bank USD liability|hedging position|hedging pnl|hedging operation|
|:-:|-|-|-|-|-|-|-|-|-|
| t \<t0 | 1.0 BTC| 0 USD|-|-|1.0 BTC|-|-|-|-|
| t = t0 | 0.0 BTC|10,000.00 USD| 1 BTC * 10,000.00 USD/BTC |-|1.0 BTC|-|10 swap contracts|-|10,000.00 USD / 100.00 USD/contract|
| t0\> t \<t1 | 0.0 BTC|10,000.00 USD|-|-|1.0 BTC|-|10,000.00 USD|-|-|
| t = t1 | 1.1 BTC| 0 USD| 10,000.00 USD / 9,090.91 USD/BTC |-|1.0 BTC|-|-|0.1 BTC|10,000.00 USD * [9,090.91^-1 BTC/USD - 10,000.00^-1 BTC/USD]|
| t \>t1 | 1.1 BTC||-|user gained 0.1 BTC (10%), bank is flat as it passes hedging gains to user|1.1 BTC|-|-|-|-|

#### Ex. 2.
#### Simple case: no bank fees, no hedging fees, no time slippage, no spot vs swap price difference, full round trip, BTC price increases
##### Exchange rates
|time|exchange rate|
|:-:|:-:|
|t0|10,000.00 USD/BTC|
|t1|11,111.11 USD/BTC|

##### Sequence of events
|time|User BTC balance|User USD balance|spot operation|note|Bank BTC reserve|Bank USD liability|hedging position|hedging pnl|hedging operation|
|:-:|-|-|-|-|-|-|-|-|-|
| t \<t0 | 1.0 BTC| 0 USD|-|-|1.0 BTC|-|-|-|-|
| t = t0 | 0.0 BTC|10,000.00 USD| 1 BTC * 10,000.00 USD/BTC |-|1.0 BTC|-|10 swap contracts|-|10,000.00 USD / 100.00 USD/contract|
| t0\> t \<t1 | 0.0 BTC|10,000.00 USD|-|-|1.0 BTC|-|10,000.00 USD|-|-|
| t = t1 | 0.9 BTC| 0 USD| 10,000.00 USD / 11,111.11 USD/BTC |-|1.0 BTC|-|-|-0.1 BTC|10,000.00 USD * [11,111.11^-1 BTC/USD - 10,000.00^-1 BTC/USD]|
| t \>t1 | 0.9 BTC||-|user lost 0.1 BTC (10%), bank is flat as it passes hedging losses to user|0.9 BTC|-|-|-|-|


## Introducing complications

Starting at the exchange, there are trading fees, funding fees and slippage (which in this case is the difference in rates quoted to the user and used to record the transaction in the ledger versus the actual executed rate when hedging on the exchange).

Then the bank could/should? have fees of it's own (as service fees, community "penny pot", etc.)

And finally the price quoted to the user could be from a different source/instrument then the one used to hedged (aka spot vs swap).

### Trading fees
Trading fees, versus other fees incurred on the exchange are always an expense. (Unless maybe the hedging becomes important enough to open a market making desk and getting paid to provide liquidity by using the hedging flow...)

Looking at OKX with market orders on perpetual swaps at the lower tier, the trading fees are 0.05% (5 basis points).

Thinking about the impact of that on the [examples](./USER_PNL.md#ex-1) above, the hedging pnl would be down by 0.05%.
So either the bank absorbs it which means it has to be funded from other profitable areas of business of the bank, if any.
Or it is passed on to the user which seems fair as it is the cost of conversion not so different from the price of BTC which is itself not a fix number but a distribution and at 0.05% (the most expensive trading fees on OKX) the trading-fee-included-quoted price to the user is very likely within that distribution (to be backtested).

Therefore the process of passing on the trading fees back to the users is to quote them a BTC price inclusive of it in the proper direction.
So that once realized in the market, when hedging, it is accounted for.

#### Ex. 3.
#### Simple case: no bank fees, trading fees, no funding fees, no time slippage, no spot vs swap price difference, full round trip, BTC price decreases
##### Fees
|fee name|fee|
|:-:|:-:|
|trading fees|0.05%|

##### Exchange rates
|time|exchange rate|exchange rate with fees|
|:-:|:-:|:-:|
|t0|10,000.00 USD/BTC|9,995.00 USD/BTC|
|t1|9,090.91 USD/BTC|9095.46 USD/BTC|

##### Sequence of events
|time|User BTC balance|User USD balance|spot operation|note|Bank BTC reserve|Bank USD liability|hedging position|hedging pnl|hedging operation|
|:-:|-|-|-|-|-|-|-|-|-|
| t \<t0 | 1.0 BTC| 0 USD|-|-|1.0 BTC|-|-|-|-|
| t = t0 | 0.0 BTC|9,995.00 USD| 1 BTC * 10,000.00 USD/BTC * (1 - 0.05%) |-|1.0 BTC|-|10 swap contracts|-0.0005 BTC|10,000.00 USD / 100.00 USD/contract|
| t0\> t \<t1 | 0.0 BTC|9,995.00 USD|-|-|1.0 BTC|-|10,000.00 USD|-0.0005 BTC|-|
| t = t1 | 1.0989 BTC| 0 USD| 9,995.00 USD / 9,090.91 USD/BTC / (1 + 0.05%) |-|1.0 BTC|-|-|0.0990 BTC|10,000.00 USD * [9,090.91^-1 BTC/USD - 10,000.00^-1 BTC/USD]|
| t \>t1 | 1.0989 BTC||-|user gained 0.0989 BTC (9.89%), bank is flat-ish as it passes hedging gains to user|1.0990 BTC|-|-|-|-|

*note the hedging pnl is different from the user pnl due to the contract notional granularity of 100 USD

**note the hedging pnl in this case is favorable to the bank

***note that hedging pnl difference would be exacerbated by larger notional derivative contracts

#### Ex. 4.
#### Simple case: no bank fees, no hedging fees, no time slippage, no spot vs swap price difference, full round trip, BTC price increases
##### Fees
|fee name|fee|
|:-:|:-:|
|trading fees|0.05%|

##### Exchange rates
|time|exchange rate|exchange rate with fees|
|:-:|:-:|:-:|
|t0|10,000.00 USD/BTC|9,995.00 USD/BTC|
|t1|11,111.11 USD/BTC|11116.66 USD/BTC|

##### Sequence of events
|time|User BTC balance|User USD balance|spot operation|note|Bank BTC reserve|Bank USD liability|hedging position|hedging pnl|hedging operation|
|:-:|-|-|-|-|-|-|-|-|-|
| t \<t0 | 1.0 BTC| 0 USD|-|-|1.0 BTC|-|-|-|-|
| t = t0 | 0.0 BTC|9,995.00 USD| 1 BTC * 10,000.00 USD/BTC * (1 - 0.05%) |-|1.0 BTC|-|10 swap contracts|-0.0005 BTC|10,000.00 USD / 100.00 USD/contract|
| t0\> t \<t1 | 0.0 BTC|9,995.00 USD|-|-|1.0 BTC|-|10,000.00 USD|-0.0005 BTC|-|
| t = t1 | 0.89910054 BTC| 0 USD| 9,995.00 USD / 11,111.11 USD/BTC / (1 + 0.05%) |-|1.0 BTC|-|-|-0.101 BTC|10,000.00 USD * [11,111.11^-1 BTC/USD - 10,000.00^-1 BTC/USD]|
| t \>t1 | 0.89910054 BTC||-|user lost ~0.1 BTC (>10%), bank is flat-ish as it passes hedging losses to user|0.899 BTC|-|-|-|-|

*note the hedging pnl is different from the user pnl due to the contract notional granularity of 100 USD

**note the hedging pnl in this case is unfavorable to the bank which owes 10,054 sats it does not have in reserve

***note that hedging pnl difference would be exacerbated by larger notional derivative contracts


### Funding fees
The funding fees are not fees per se but exchanges of profits between market participants in the perpetual swap market in order to tie the instrument price to the underlying via the creation of an artificial arbitrage opportunity on the opposite side of the diverging trend.

The funding fees are hard to account for a priori as they depend on market activity, change frequently (ex. every 8h) and are only known and incurred after hedging activity, making them impossible to accurately pass on to the user unless calculated a posteriori as a pro rata of the USD balance hedged for the user for the sum of the funding fees expense/income during the life of that USD balance.

As an example, one user convert BTC to USD for 1h between the 8h separating two funding fees settlement time, then converts back to BTC.
No funding fees were involved, the user is not charged nor reimbursed.
Another user does the same but his USD balance is actively being hedged during one of the funding fees settlement time and so a funding fees expense/income is recorded.
That user's share of the funding fees is the ratio of his balance and the bank's total exposure being hedged at the time the funding fees was recorded.
(The general case, as stated above, is the sum of all those proportioned funding fees for the life of the USD balance).

It is hard to put in perspective the effect of funding fees on the PnL like the examples above do for trading fees as it depends on time of exposure, market activity, intensity of the fees, etc.

The best would probably be to use historical data to establish expectations and guide decisions on how to handle.
So far it was believed a static short hedge in a price-go-up-forever-laura context would harvest the funding fees, 
but it has been the opposite 2-to-1 in favor of funding expense versus income.

It raises the question whether the perpetual swap, an every 8h settled instrument, is the right one to hedge a long-term maturity liability?
Or should a mix of hourly, daily, weekly, monthly and yearly settled instrument be deployed to match the different duration of liabilities
naturally arising in a steady-state bitcoin bank like insurance companies matching asset-liability maturities.

So for now it can be assumed that the funding fees are absorbed by the bank and funded, if needed, via other activities (ex. forecast funding expenses from historical data and passed on to users via bank fees). It will not be included in calculations.

### Bank fees
Bank fees are discretionary fees indirectly charged to the users by applying a spread on the exchange rate in favor of the bank. 
In the USD conversion, the bank buys BTC low (expected exchange rate to be paid by the bank **less** a percentage) and sells it high (expected exchange rate to be paid by the bank **plus** a percentage) therefore netting a profit.

#### Ex. 5.
#### Simple case: bank fees, trading fees, no funding fees, no time slippage, no spot vs swap price difference, full round trip, BTC price decreases
##### Fees
|fee name|fee|
|:-:|:-:|
|banking fees|0.05%|
|trading fees|0.05%|

##### Exchange rates
|time|exchange rate|exchange rate with fees|
|:-:|:-:|:-:|
|t0|10,000.00 USD/BTC|9,990.00 USD/BTC|
|t1|9,090.91 USD/BTC|9100.00 USD/BTC|

##### Sequence of events
|time|User BTC balance|User USD balance|spot operation|note|Bank BTC reserve|Bank USD liability|hedging position|hedging pnl|hedging operation|
|:-:|-|-|-|-|-|-|-|-|-|
| t \<t0 | 1.0 BTC| 0 USD|-|-|1.0 BTC|-|-|-|-|
| t = t0 | 0.0 BTC|9,990.00 USD| 1 BTC * 10,000.00 USD/BTC * (1 - 2*0.05%) |-|1.0 BTC|-|10 swap contracts|-0.0005 BTC|10,000.00 USD / 100.00 USD/contract|
| t0\> t \<t1 | 0.0 BTC|9,990.00 USD|-|-|1.0 BTC|-|10,000.00 USD|-0.0005 BTC|-|
| t = t1 | 1.09780209 BTC| 0 USD| 9,990.00 USD / 9,090.91 USD/BTC / (1 + 2*0.05%) |-|1.0 BTC|-|-|0.0990 BTC|10,000.00 USD * [9,090.91^-1 BTC/USD - 10,000.00^-1 BTC/USD]|
| t \>t1 | 1.09780209 BTC||-|user gained 0.0978 BTC (9.78%), bank has a 119,791 sats profit|1.0990 BTC|-|-|-|-|


#### Ex. 6.
#### Simple case: no bank fees, no hedging fees, no time slippage, no spot vs swap price difference, full round trip, BTC price increases
##### Fees
|fee name|fee|
|:-:|:-:|
|banking fees|0.05%|
|trading fees|0.05%|

##### Exchange rates
|time|exchange rate|exchange rate with fees|
|:-:|:-:|:-:|
|t0|10,000.00 USD/BTC|9,990.00 USD/BTC|
|t1|11,111.11 USD/BTC|11122.22 USD/BTC|

##### Sequence of events
|time|User BTC balance|User USD balance|spot operation|note|Bank BTC reserve|Bank USD liability|hedging position|hedging pnl|hedging operation|
|:-:|-|-|-|-|-|-|-|-|-|
| t \<t0 | 1.0 BTC| 0 USD|-|-|1.0 BTC|-|-|-|-|
| t = t0 | 0.0 BTC|9,990.00 USD| 1 BTC * 10,000.00 USD/BTC * (1 - 2*0.05%) |-|1.0 BTC|-|10 swap contracts|-0.0005 BTC|10,000.00 USD / 100.00 USD/contract|
| t0\> t \<t1 | 0.0 BTC|9,990.00 USD|-|-|1.0 BTC|-|10,000.00 USD|-0.0005 BTC|-|
| t = t1 | 0.89820189 BTC| 0 USD| 9,990.00 USD / 11,111.11 USD/BTC / (1 + 2*0.05%) |-|1.0 BTC|-|-|-0.101 BTC|10,000.00 USD * [11,111.11^-1 BTC/USD - 10,000.00^-1 BTC/USD]|
| t \>t1 | 0.89820189 BTC||-|user lost ~0.1 BTC (>10%), bank has a 79,811 sats profit|0.899 BTC|-|-|-|-|


### Slippage
Slippage in this case is the difference in rates quoted to the user and used to record the transaction in the ledger versus the actual executed rate when hedging on the exchange. 

It can fall on either side of the PnL and depends on the market movement between the time the ledger is written and the effective hedge for that transaction is executed on the exchange.

#### Ex. 7.
#### Simple case: no bank fees, no hedging fees, slippage up, no spot vs swap price difference, full round trip, BTC price increases
##### Exchange rates
|time|exchange rate|
|:-:|:-:|
|t0|10,000.00 USD/BTC|
|t1|11,111.11 USD/BTC|
|t2|22,222.22 USD/BTC|

##### Sequence of events
|time|User BTC balance|User USD balance|spot operation|note|Bank BTC reserve|Bank USD liability|hedging position|hedging pnl|hedging operation|
|:-:|-|-|-|-|-|-|-|-|-|
| t \<t0 | 1.0 BTC| 0 USD|-|-|1.0 BTC|-|-|-|-|
| t = t0 | 0.0 BTC|10,000.00 USD| 1 BTC * 10,000.00 USD/BTC |-|1.0 BTC|-|-|-|-|
| t = t1 | 0.0 BTC|10,000.00 USD|-|swap position opened at 11,111.11 USD/BTC|1.0 BTC|-|10 swap contracts|-|10,000.00 USD / 100.00 USD/contract|
| t1\> t \<t2 | 0.0 BTC|10,000.00 USD|-|-|1.0 BTC|-|10,000.00 USD|-|-|
| t = t2 | 0.45 BTC| 0 USD| 10,000.00 USD / 22,222.22 USD/BTC |-|1.0 BTC|-|-|-0.45 BTC|10,000.00 USD * [22,222.22^-1 BTC/USD - 11,111.11^-1 BTC/USD]|
| t \>t2 | 0.45 BTC||-|bank is left with 0.55 BTC reserve and owing 0.45 BTC, an accidental profit of 0.1 BTC|0.55 BTC|-|-|-|-|

#### Ex. 8.
#### Simple case: no bank fees, no hedging fees, slippage down, no spot vs swap price difference, full round trip, BTC price increases
##### Exchange rates
|time|exchange rate|
|:-:|:-:|
|t0|10,000.00 USD/BTC|
|t1|9,090.91 USD/BTC|
|t2|22,222.22 USD/BTC|

##### Sequence of events
|time|User BTC balance|User USD balance|spot operation|note|Bank BTC reserve|Bank USD liability|hedging position|hedging pnl|hedging operation|
|:-:|-|-|-|-|-|-|-|-|-|
| t \<t0 | 1.0 BTC| 0 USD|-|-|1.0 BTC|-|-|-|-|
| t = t0 | 0.0 BTC|10,000.00 USD| 1 BTC * 10,000.00 USD/BTC |-|1.0 BTC|-|-|-|-|
| t = t1 | 0.0 BTC|10,000.00 USD|-|swap position opened at 9,090.91 USD/BTC|1.0 BTC|-|10 swap contracts|-|10,000.00 USD / 100.00 USD/contract|
| t1\> t \<t2 | 0.0 BTC|10,000.00 USD|-|-|1.0 BTC|-|10,000.00 USD|-|-|
| t = t2 | 0.45 BTC| 0 USD| 10,000.00 USD / 22,222.22 USD/BTC |-|1.0 BTC|-|-|-0.65 BTC|10,000.00 USD * [22,222.22^-1 BTC/USD - 9,090.91^-1 BTC/USD]|
| t \>t2 | 0.45 BTC||-|bank is left with 0.35 BTC reserve and owing 0.45 BTC, an accidental loss of 0.1 BTC|0.35 BTC|-|-|-|-|

#### Ex. 9.
#### Simple case: no bank fees, no hedging fees, slippage up, no spot vs swap price difference, full round trip, BTC price decreases
##### Exchange rates
|time|exchange rate|
|:-:|:-:|
|t0|10,000.00 USD/BTC|
|t1|11,111.11 USD/BTC|
|t2|7,000.00 USD/BTC|

##### Sequence of events
|time|User BTC balance|User USD balance|spot operation|note|Bank BTC reserve|Bank USD liability|hedging position|hedging pnl|hedging operation|
|:-:|-|-|-|-|-|-|-|-|-|
| t \<t0 | 1.0 BTC| 0 USD|-|-|1.0 BTC|-|-|-|-|
| t = t0 | 0.0 BTC|10,000.00 USD| 1 BTC * 10,000.00 USD/BTC |-|1.0 BTC|-|-|-|-|
| t = t1 | 0.0 BTC|10,000.00 USD|-|swap position opened at 11,111.11 USD/BTC|1.0 BTC|-|10 swap contracts|-|10,000.00 USD / 100.00 USD/contract|
| t1\> t \<t2 | 0.0 BTC|10,000.00 USD|-|-|1.0 BTC|-|10,000.00 USD|-|-|
| t = t2 | 1.42857 BTC| 0 USD| 10,000.00 USD / 7,000.00 USD/BTC |-|1.0 BTC|-|-|0.52857 BTC|10,000.00 USD * [7,000.00^-1 BTC/USD - 11,111.11^-1 BTC/USD]|
| t \>t2 | 1.42857 BTC||-|bank is left with 1.52857 BTC reserve and owing 1.42857 BTC, an accidental profit of 0.1 BTC|1.52857 BTC|-|-|-|-|

#### Ex. 10.
#### Simple case: no bank fees, no hedging fees, slippage down, no spot vs swap price difference, full round trip, BTC price decreases
##### Exchange rates
|time|exchange rate|
|:-:|:-:|
|t0|10,000.00 USD/BTC|
|t1|9,090.91 USD/BTC|
|t2|7,000.00 USD/BTC|

##### Sequence of events
|time|User BTC balance|User USD balance|spot operation|note|Bank BTC reserve|Bank USD liability|hedging position|hedging pnl|hedging operation|
|:-:|-|-|-|-|-|-|-|-|-|
| t \<t0 | 1.0 BTC| 0 USD|-|-|1.0 BTC|-|-|-|-|
| t = t0 | 0.0 BTC|10,000.00 USD| 1 BTC * 10,000.00 USD/BTC |-|1.0 BTC|-|-|-|-|
| t = t1 | 0.0 BTC|10,000.00 USD|-|swap position opened at 9,090.91 USD/BTC|1.0 BTC|-|10 swap contracts|-|10,000.00 USD / 100.00 USD/contract|
| t1\> t \<t2 | 0.0 BTC|10,000.00 USD|-|-|1.0 BTC|-|10,000.00 USD|-|-|
| t = t2 | 1.42857 BTC| 0 USD| 10,000.00 USD / 7,000.00 USD/BTC |-|1.0 BTC|-|-|0.32857 BTC|10,000.00 USD * [7,000.00^-1 BTC/USD - 9,090.91^-1 BTC/USD]|
| t \>t2 | 1.42857 BTC||-|bank is left with 1.42857 BTC reserve and owing 1.42857 BTC, an accidental loss of 0.1 BTC|1.32857 BTC|-|-|-|-|


Given that a small USD conversion might not initiate a hedging transaction until the liability passes the threshold of the notional of a single contract (ex. 100 USD), 
the timing between the ledger writing and the hedge execution can be quite long, but it is mitigated by the size of the transaction <50% of the contract notional.

Assuming there's a trigger mechanism propagating the ledger changed in USD liability to be hedged to the Dealer quick enough, 
and that taking that incremental position does not require additional funds to be transferred on-chain, 
then the time difference between quote and execution can be expected to be a matter of seconds.
If an on-chain fund transfer is required, an average 10min * number of confirmation required by the exchange is to be expected.

In all cases, given the historical market data of the BTC/USD price, a price confidence interval can be calculated for each time frame giving an estimate of PnL that could be expected.
This analysis, in association with statistics on the transaction size, would also motivate, for example keeping more or less funds on the exchange for faster execution and help determine parameters of the hedging strategy, if not motivate moving to LN fund transfers, etc.

So from this it should be clear that determining the Pnl incurred due to slippage is not straight forward and that it can be assumed that it is absorbed by the bank and funded, if needed, via other activities (ex. forecast slippage from historical data and passed on to users via bank fees). It will not be included in calculations.

### Spot vs Swap price difference

The Spot vs Swap price difference comes from the expectation that given a spot trade (BTC to USD conversion) the quoted price should be sourced from the spot market (versus perpetual swap for example) even though the mechanism enabling that synthetic USD is not from that spot market but actually from the perpetual swap market (or else) enabling it.

 A long discussion can be had on "what if futures?", etc., etc. 
 But given that perpetual swaps are tied to the underlying spot, they can't diverge that far and over time they get compensated for via the funding fees if they do, 
 and that at a few basis points difference between spot and swap, a difference in cost that would likely be passed down to the users anyways, why not quote the swap directly.

 If not, then the analysis is similar and consequences the same as for [Slippage](./USER_PNL.md#slippage), 
 i.e. the quoted and ledger recorded exchange rate differs from the execution exchange rate.

 The complication being that two different markets with different dynamics need to be analyzed and that the optimal hedging ratio is not 1.00 anymore but the regression of the differences in prices in spot market versus the differences in prices in hedging instrument market as explained [here](./GRAFANA.md#hedge-effectiveness).

So from this, it should also be clear that determining the Pnl incurred due to Spot vs Swap price difference is not straight forward and that it can be assumed that it is absorbed by the bank and funded, if needed, via other activities (ex. forecast Spot vs Swap price difference from historical data and passed on to users via bank fees). It will not be included in calculations.

*note that we have an indication of the aggregate slippage + Spot vs Swap price difference via the reported BTC and USD liability implying an average open price of (USD liability / BTC liability) USD/BTC yielding a "spot" unrealized Pnl (aggregate user uPnL?) of: BTC-liability BTC * (CurrentBtcPrice - AverageOpenPrice) USD/BTC.

**note that in the average open price proxy above is post bank and trading fees passed on to the user 
the trading fees will be reflected in the BTC balance immediately as the hedging position is entered
the bank fees will be reflected in the USD liability
so if both are 0, then the average open price proxy is exactly the quoted exchange rate which in turn is exactly the swap open price and so spot and swap uPnl will cancel each other for a perfect hedge
if trading fees are non-null, and absorbed by the bank, then the USD liability is untouched by the passing on of the fees and is exactly the quoted exchange rate,
but the BTC liability reflects the fees and they appear on the spot uPnl, and since the swap uPnl only depends on the hedged


and is therefore biased
^--- is this true or isn't the pnl going to reflect the bank fees as the trading fees are passed on, (given a round contract notional exposure)?

***note that in the "spot" uPnL calculation above, if slippage is null and spot price is quoted with swap price, the strategy pnl should reflect the banking fees only

****lets prove the above 2!

#### Ex. 5. from earlier
#### Simple case: bank fees, trading fees, no funding fees, no time slippage, no spot vs swap price difference, full round trip, BTC price decreases
##### Fees
|fee name|fee|
|:-:|:-:|
|banking fees|0.05%|
|trading fees|0.05%|

##### Exchange rates
|time|exchange rate|exchange rate with fees|
|:-:|:-:|:-:|
|t0|10,000.00 USD/BTC|9,990.00 USD/BTC|
|t1|9,090.91 USD/BTC|9100.00 USD/BTC|

##### Sequence of events
|time|User BTC balance|User USD balance|spot operation|note|Bank BTC reserve|Bank USD liability|hedging position|hedging pnl|hedging operation|
|:-:|-|-|-|-|-|-|-|-|-|
| t \<t0 | 1.0 BTC| 0 USD|-|-|1.0 BTC|-|-|-|-|
| t = t0 | 0.0 BTC|9,990.00 USD| 1 BTC * 10,000.00 USD/BTC * (1 - 2*0.05%) |-|1.0 BTC|9,990.00 USD|10 swap contracts|-0.0005 BTC|10,000.00 USD / 100.00 USD/contract|
| t0\> t \<t1 | 0.0 BTC|9,990.00 USD|-|Spot uPnl = (1.00 - 0.0005) BTC * (9,090.91 - (9,990.00 USD / (1.00 - 0.0005) BTC)) USD/BTC |1.0 BTC|9,990.00 USD|10,000.00 USD|-0.0005 BTC|-|
| t = t1 | 1.09780209 BTC| 0 USD| 9,990.00 USD / 9,090.91 USD/BTC / (1 + 2*0.05%) |-|1.0 BTC|-|-|0.0990 BTC|10,000.00 USD * [9,090.91^-1 BTC/USD - 10,000.00^-1 BTC/USD]|
| t \>t1 | 1.09780209 BTC||-|user gained 0.0978 BTC (9.78%), bank has a 119,791 sats profit|1.0990 BTC|-|-|-|-|










## PnL from the user's point of view

## PnL from the bitcoin bank's point of view

The first component of the profit and loss incurred by the bitcoin bank in a local currency (USD) conversion transaction with its users is related to the exchange rate used.

