<script lang="ts">
	import type { Ethereum } from '../data/ethereum/types'
	import type { Covalent } from '../data/analytics/covalent'
	import { getHistoricalPricesByTickerSymbol, getHistoricalPricesByAddress } from '../data/analytics/covalent'
	import type { QuoteCurrency, TickerSymbol } from '../data/currency/currency'
	import type { PriceScale } from './PriceChart.svelte'
	
	export let historicalPriceProvider
	export let currencies: (TickerSymbol | Ethereum.ContractAddress)[]
	export let chainID: Ethereum.ChainID = 1
	export let quoteCurrency: QuoteCurrency
	export let fromDate = '2017-01-01'
	export let toDate = new Date().toISOString().slice(0, 10) // today's date
	export let fromPrice = 0
	export let toPrice = 1000
	export let priceScale: PriceScale


	const isAddress = query => /^0x[0-9a-f]{40}$/i.test(query)


	const PromiseAllFulfilled = promises =>
		Promise.allSettled(promises).then(results =>
			Object.values(results)
				.filter(({status}) => status === 'fulfilled')
				.map(({value}) => value)
		)


	import Loader from './Loader.svelte'
	import PriceChart from './PriceChart.svelte'
</script>

{#if historicalPriceProvider === 'Covalent' && currencies}
	<Loader
		fromPromise={
			() => PromiseAllFulfilled(
				currencies.map(async currency => {
					if(currency.toLowerCase() === '0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee')
						currency = 'ETH'
					
					const _isAddress = isAddress(currency)

					const data = _isAddress
						? await getHistoricalPricesByAddress({chainID, contractAddress: currency, quoteCurrency, from: fromDate, to: toDate})
						: await getHistoricalPricesByTickerSymbol({tickerSymbol: currency, quoteCurrency, from: fromDate, to: toDate})

					return {
						currency: _isAddress ? data.contract_ticker_symbol : currency,
						prices: data.prices.map(({date, price}) => ({time: date, price}))
					}
				})
			)
		}

		loadingIcon="/logos/covalent-logomark.svg"
		loadingIconName={historicalPriceProvider}
		loadingMessage="Retrieving price history from {historicalPriceProvider}..."
		let:then={data}
		showIf={data => data.length}
	>
		<slot name="header"></slot>

		<PriceChart
			{data}
			{quoteCurrency}
			timeRange={[fromDate, toDate]}
			priceRange={[fromPrice, toPrice]}}
			{priceScale}
		/>
	</Loader>
{/if}

<!-- () => Promise.all(
	currencies.map(async currency => {
		const data = await getHistoricalPrices({tickerSymbol: currency, quoteCurrency, from: fromDate, to: toDate})
		console.log(data)
		return {
			currency,
			prices: data.prices.map(({date, price}) => ({time: date, price}))
		}
	})
) -->

<!-- currencies ?
	() => Promise.all(
		currencies.map(async currency => {
			const data = await getHistoricalPrices({tickerSymbol: currency, quoteCurrency, from: fromDate, to: toDate})
			console.log(data)
			return {
				currency,
				prices: data.prices.map(({date, price}) => ({time: date, price}))
			}
		})
	)
: contractAddresses ?
	async () => {
		const data = await getHistoricalPricesByAddresses({contractAddresses, quoteCurrency, from: fromDate, to: toDate, chainID})
		console.log('getHistoricalPricesByAddresses', data)
		return data.map(tokenWithPrices => ({
			currency: tokenWithPrices.contract_ticker_symbol,
			prices: tokenWithPrices.prices.map(({date, price}) => ({time: date, price}))
		}))
	}
:
	undefined -->