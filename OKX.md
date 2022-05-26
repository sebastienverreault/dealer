# OKX setup with dealer

## OKX account setup

### Open account
### Demo mode
### Create API key
### Setup Margin Trading Window

## Dealer setup


The Dealer will hedge based on the balances received from the graphql service, which are stored in the postgres service.
So updating the balances in the database will create the hedging activity.

The graphql service implements a simple (simulated) version of the Galoy backend API,
so these environment variables need to be set correctly (default should work):
- NETWORK=testnet
- ACTIVE_WALLET="REMOTE_WALLET"
- GRAPHQL_URI
- DATABASE_URL

and these need to be set so the Dealer creates orders on the exchange.

- HEDGING_NOT_IN_SIMULATION="TRUE"
- OKEX5_KEY="<your okx demo api v5 key>"
- OKEX5_SECRET="<your okx demo api v5 secret>"
- OKEX5_PASSWORD="<your okx demo api v5 password>"



Once all this is configured and running,
It is possible to validate that the Dealer has access to the graphql service which in turn has access to postgres 

via Grafana (http://\${DOCKER_HOST_IP}:3000): "Liability in USD & BTC" chart showing two real values

or 

Prometheus (http://\${DOCKER_HOST_IP}:3333/metrics): galoy_dealer_liabilityInUsd & galoy_dealer_liabilityInBtc being real values (265463 sats & 10000 cents)




Now, connecting to and querying postgres it is possible to validate the same numbers:

```
SELECT json_data FROM dealer.wallet;
```
or
```
SELECT 
	json_data -> 0 ->> 'balance' as btcBalanceInSats
FROM dealer.wallet 
WHERE json_data::json -> 0 ->> 'id' = 'BTCWallet';

SELECT 
	json_data -> 1 ->> 'balance' as usdBalanceInCents
FROM dealer.wallet 
WHERE json_data::json -> 1 ->> 'id' = 'USDWallet';
```





It is also possible to validate the Dealer's access to OKX API

via Grafana : "BTC Free/Used/Total Balance on Exchange" chart showing the demo account balance

or 

Prometheus : galoy_dealer_btcFreeBalance, galoy_dealer_btcUsedBalance, galoy_dealer_btcTotalBalance & galoy_dealer_fundingAccountBtcTotalBalance



Finally, the Dealer will execute every 5m

So if the USDbalance in the dealer.wallet table is changed before one execution, next time it runs, it will transact on exchange accordingly:

```
DO $$
    DECLARE usdBalanceInCents FLOAT := -125*100;
    BEGIN
        UPDATE dealer.wallet
        SET json_data = jsonb_set(json_data::jsonb, '{1,balance}', usdBalanceInCents::text::jsonb, false);
END $$;
```

To take a look at errors, traces, etc. OpenTelemetry tracing is integrated and enabled, 
it only needs a JAEGER_HOST environment variable pointing at the docker host, ex. host.docker.internal 
and an instance of Jaeger running:

```
docker run --rm  --name jaeger \
  -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
  -p 5775:5775/udp \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 14250:14250 \
  -p 14268:14268 \
  -p 14269:14269 \
  -p 9411:9411 \
  jaegertracing/all-in-one:1.32
```

Then a browser pointing to http://${DOCKER_HOST_IP}:16686
where there should be a Service called galoy-dev-dealer to search for traces.
(app.exchangeBase.fetchPosition might show and warning level error if there are no positions on exchange)

Alternatively taking a look at the raw logs with this might be easier for isolating specific issues:
```
docker run --name dozzle -d --volume=/var/run/docker.sock:/var/run/docker.sock -p 8888:8080 amir20/dozzle:latest
```
pointing a browser at http://${DOCKER_HOST_IP}:8888

Example:

liability in usd to be hedged: 100 -> 175 -> 125 -> 25 USD

shorted number of contracts: 1-> 2 -> 1 -> 0




