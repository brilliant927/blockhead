<script lang="ts">
	import type { Ethereum } from '../data/ethereum/types'
	import type { QuoteCurrency, TickerSymbol } from '../data/currency/currency'
	import type { CurrentPriceProvider } from '../data/ethereum/price/price-feed-provider'
	import { getChainlinkPriceFeed } from '../data/ethereum/price/chainlink'
	// import { getCompoundPriceFeed } from '.../../../data/ethereum/price/compound-price-feed'
	import { getSpotPrices } from '../data/analytics/covalent'


	import { preferences } from '../data/ethereum/preferences'

	export let currentPriceProvider: CurrentPriceProvider | 'auto' = 'auto'
	$: currentPriceProvider = $$props.currentPriceProvider || $preferences.currentPriceProvider

	let _currentPriceProvider
	$: _currentPriceProvider = currentPriceProvider === 'auto' ? 'Chainlink' : currentPriceProvider

	export let token: TickerSymbol
	export let quoteCurrency: QuoteCurrency
	export let provider: Ethereum.Provider
	export let network: Ethereum.Network

	export let blockNumber: number


	let isHidden = false


	import Loader from './Loader.svelte'
	import TokenRate from './TokenRate.svelte'
	import TokenBalance from './TokenBalance.svelte'
	import Address from './Address.svelte'
</script>


<style>
	.rate {
		font-size: 1.8em;
		text-align: center;
	}

	footer {
		font-size: 0.8em;
	}
</style>


<!-- {#if provider.network}
	<Loader
		loadingIcon={priceFeedLogo}
		loadingIconName={currentPriceProvider}
		loadingMessage="Retrieving price from Chainlink..."
		fromPromise={blockNumber && () => getChainlinkPriceFeed(provider, token, quoteCurrency)}
		let:then={priceFeed}
	>
		<div slot="header" class="bar">
			<h3>Current Price</h3>
			<span class="card-annotation">{currentPriceProvider}</span>
		</div>

		<TokenRate
			rate={priceFeed.price}
			quoteToken={quoteCurrency}
			baseToken={token}
			layout="horizontal"
		/>
		<p>Updated {priceFeed.updatedAt.toLocaleTimeString()}
	</Loader>
{/if} -->
<!-- <div class="stack">
	{#key blockNumber}
		{#if provider.network}
			<Loader
				loadingIcon={priceFeedLogo}
				loadingIconName={currentPriceProvider}
				loadingMessage="Retrieving price from Chainlink..."
				fromPromise={() => getChainlinkPriceFeed(provider, token, quoteCurrency)}
				let:then={priceFeed}
			>
				<div slot="header" class="bar">
					<h3>Current Price</h3>
					<span class="card-annotation">{currentPriceProvider}</span>
				</div>
				<TokenRate
					rate={priceFeed.price}
					quoteToken={quoteCurrency}
					baseToken={token}
					layout="horizontal"
				/>
				<p>Updated {priceFeed.updatedAt.toLocaleTimeString()}
			</Loader>
		{/if}
	{/key}
</div> -->

{#if !isHidden}
	<div class="stack">
		{#if _currentPriceProvider === 'Chainlink'}
			<div class="column">
				<Loader
					loadingIcon={'/logos/chainlink.svg'}
					loadingIconName={_currentPriceProvider}
					loadingMessage="Retrieving price from {_currentPriceProvider}..."
					errorMessage="{token} price not available"
					fromPromise={provider && network && blockNumber && (() => getChainlinkPriceFeed(provider, network, token, quoteCurrency))}
					let:then={priceFeed}
					whenErrored={async () => {
						await new Promise(r => setTimeout(r, 1000))
						if(currentPriceProvider === 'auto')
							_currentPriceProvider = 'Covalent'
					}}
				>
					<div slot="header" class="bar" let:status>
						<slot name="title">
							<h3>Current Price</h3>
						</slot>

						<span class="card-annotation">
							{_currentPriceProvider}
							{#if status === 'resolved'}
								›
								<Address {network} address={priceFeed.contractAddress}>{token}/{quoteCurrency} Price Feed</Address>
							{/if}
						</span>
					</div>

					<div class="rate">
						<TokenBalance
							balance={1}
							symbol={token}
							tween={false}
						/>
						=
						<TokenBalance
							balance={priceFeed.price}
							symbol={quoteCurrency}
							showPlainFiat={true}
						/>
					</div>
					<!-- <div class="rate">
						<TokenRate
							rate={priceFeed.price}
							quoteToken={quoteCurrency}
							baseToken={token}
							layout="horizontal"
						/>
					</div> -->
					<footer class="bar">
						<span />
						<span class="card-annotation">Updated {priceFeed.updatedAt.toLocaleTimeString()}</span>
					</footer>
				</Loader>
			</div>
		{:else if _currentPriceProvider === 'Covalent'}
			<div class="column">
				<Loader
					loadingIcon={'/logos/covalent-logomark.svg'}
					loadingIconName={_currentPriceProvider}
					loadingMessage="Retrieving price from {_currentPriceProvider}..."
					errorMessage="{token} price not available"
					fromPromise={async () => {
						const data = await getSpotPrices({tickers: [token]})
						if(data?.items?.[0])
							return {
								price: data.items[0].quote_rate,
								rank: data.items[0].rank,
								icon: data.items[0].logo_url,
								updatedAt: data.updated_at
							}
						throw new Error(`The ${token} spot price isn't currently indexed by Covalent.`)
					}}
					let:then={data}
				>
					<div slot="header" class="bar">
						<slot name="title">
							<h3>Current Price</h3>
						</slot>

						<span class="card-annotation">
							{_currentPriceProvider}
						</span>
					</div>

					<svelte:fragment slot="errorActions" let:cancel>
						<button class="small" on:click={() => {
							cancel()
							isHidden = true
						}}>Dismiss</button>
					</svelte:fragment>

					<div class="rate">
						<TokenBalance
							balance={1}
							symbol={token}
							icon={data.icon}
							tween={false}
						/>
						=
						<TokenBalance
							balance={data.price}
							symbol={/*quoteCurrency*/ 'USD'}
							showPlainFiat={true}
						/>
					</div>
					<!-- <div class="rate">
						<TokenRate
							rate={data.price}
							quoteToken={quoteCurrency}
							baseToken={token}
							layout="horizontal"
						/>
					</div> -->
					<footer class="bar">
						<span>Rank #{data.rank}</span>
						<span class="card-annotation">Updated {new Date(data.updatedAt).toLocaleTimeString()}</span>
					</footer>
				</Loader>
			</div>
		{/if}
	</div>

	<!-- {#if isMounted}
		{#await getCompoundPriceFeed(token, quoteCurrency)}
			<Loading>
				<img slot="icon" src="/logos/chainlink" alt="Chainlink" width="50">
				Retrieving price...
			</Loading>
		{:then rate}
			<section class="card">
				<h3>Current Rate (Chainlink)</h3>
				<TokenRate {rate} quoteToken={token} baseToken={quoteCurrency} />
			</section>
		{/await}
	{/if} -->
{/if}