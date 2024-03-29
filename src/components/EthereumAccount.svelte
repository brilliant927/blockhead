<script lang="ts">
	import type { Ethereum } from '../data/ethereum/types'
	import type { TickerSymbol } from '../data/currency/currency'
	import type { Covalent } from '../data/analytics/covalent'
	import type { PriceScale } from './PriceChart.svelte'
	import { getERC20TokenTransfers, getTransactionsByAddress } from '../data/analytics/covalent'
	import { preferences } from '../data/ethereum/preferences'


	export let network: Ethereum.Network
	export let addressOrENSName: Ethereum.Address | string
	export let filterQuery: Ethereum.Address | Ethereum.ContractAddress | Ethereum.BlockNumber
	export let provider: Ethereum.Provider

	export let tokenBalancesProvider
	export let transactionProvider

	$: tokenBalancesProvider = $$props.tokenBalancesProvider || $preferences.tokenBalancesProvider
	$: transactionProvider = $$props.transactionProvider || $preferences.transactionProvider


	let detailLevel: 'summary' | 'detailed' | 'exhaustive' = 'detailed'
	let showValues: 'original' | 'converted' | 'both' = 'original'
	let showFees = false
	let sortBy: 'value-descending' | 'value-ascending' | 'ticker-ascending' = 'value-descending'
	let showSmallValues = false

	$: quoteCurrency = $preferences.quoteCurrency


	let selectedToken: Ethereum.ERC20Token | undefined

	import { isAddress } from '@ethersproject/address'

	$: if(isAddress(filterQuery) && balances){
		selectedToken = balances
			.find(({token}) => token.address.toLowercase() === filterQuery.toLowerCase())
	}
	// $: if(selectedToken)
	// 	filterQuery = selectedToken.address


	let balances: Covalent.ERC20TokenOrNFTContractWithBalance[]

	let priceScale: PriceScale


	import Address from './Address.svelte'
	import Balance from './Balance.svelte'
	import CovalentPriceChart from './CovalentPriceChart.svelte'
	import EnsName from './EnsName.svelte'
	import EnsResolutionLoader from './EnsResolutionLoader.svelte'
	import EthereumBalances from './EthereumBalances.svelte'
	import EthereumTransactionCovalent from './EthereumTransactionCovalent.svelte'
	import EthereumTransactionMoralis from './EthereumTransactionMoralis.svelte'
	import EthereumTransactionEtherspot from './EthereumTransactionEtherspot.svelte'
	import EthereumTransactionsLoader from './EthereumTransactionsLoader.svelte'
	import EthereumTransactionsERC20Loader from './EthereumTransactionsERC20Loader.svelte'
	import TokenName from './TokenName.svelte'
	import TokenBalance from './TokenBalance.svelte'


	import { fade } from 'svelte/transition'
</script>


<style>
	.bar {
		/* --padding-inner: 1.25em; */
		--padding-inner: 1em;
	}
	label, button {
		font-size: 0.9em;
	}

	h3 {
		line-height: 1.6;
	}
	/* .transactions :global(.token-balance-container) {
		font-size: 1.1em;
	} */

	.ethereum-account {
		--echart-height: 20rem;
	}
</style>


<div class="ethereum-account card">
	<EnsResolutionLoader
		{addressOrENSName}
		{provider}
		passiveReverseResolution
		let:address
		let:ensName
		let:isReverseResolving
		showIf={({address}) => address}
	>
		<div slot="header" class="bar">
			<div class="row-inline">
				{#if address}
					<h2><Address {network} {address} /></h2>
					{#if ensName}
						<EnsName {ensName} />
					{/if}
				{:else if ensName}
					<h2><EnsName {ensName} /></h2>
				{/if}
				<!-- {#if isReverseResolving}
					{#if address}
						<h2><Address {network} {address} /></h2>
					{/if}
					{#if ensName}
						<EnsName {ensName} />
					{/if}
				{:else}
					{#if ensName}
						<h2><EnsName {ensName} /></h2>
					{/if}
					{#if address}
						<Address {network} {address} />
					{/if}
				{/if} -->
			</div>
			<span class="card-annotation">Ethereum Account</span>
		</div>

		<!-- <Balance {provider} {address} /> -->
		<EthereumBalances
			{network}
			{address}
			{tokenBalancesProvider}
			{quoteCurrency}
			{sortBy}
			{showSmallValues}
			{showValues}
			isSelectable={true}
			bind:selectedToken
			bind:balances
		>
			<svelte:fragment slot="header" let:network let:quoteCurrency let:quoteTotal>
				<hr>

				<div class="bar">
					<h3>{network.name} Tokens (<TokenBalance symbol={quoteCurrency} balance={quoteTotal} showPlainFiat={true} />)</h3>
					<label>
						<input type="checkbox" bind:checked={showSmallValues}>
						<span>Show Small Values</span>
					</label>
					<label>
						<span>Sort</span>
						<select bind:value={sortBy}>
							<option value="ticker-ascending">Alphabetical</option>
							<option value="value-descending">Highest Value</option>
							<option value="value-ascending">Lowest Value</option>
						</select>
					</label>
					<label>
						<span>Show</span>
						<select bind:value={showValues}>
							<option value="original">Balances</option>
							<option value="converted">Quotes</option>
							<option value="both">Balances + Quotes</option>
							<!-- <option value="original">Token Amounts</option>
							<option value="converted">Quote Values</option>
							<option value="both">Amounts and Values</option> -->
						</select>
					</label>
				</div>
			</svelte:fragment>
		</EthereumBalances>

		<hr>

		<div class="stack">
			{#if !selectedToken}
				<div class="column" transition:fade>
					<!-- Regular Ethereum Transactions -->
					<EthereumTransactionsLoader
						{network}
						{address}
						{provider}
						{transactionProvider}
						{quoteCurrency}
						includeLogs={detailLevel === 'exhaustive'}
						let:transactions
					>
						<svelte:fragment slot="header" let:status>
							<div class="bar">
								<h3>
									Transactions
									{#if status === 'resolved'}({transactions.length}{transactions.length === 100 ? '+' : ''}){/if}
								</h3>
								<label>
									<input type="checkbox" bind:checked={showFees}>
									<span>Show Fees</span>
								</label>
								<label>
									<span>View</span>
									<select bind:value={detailLevel}>
										<option value="summary">Summary</option>
										<option value="detailed">Detailed</option>
										<option value="exhaustive">Exhaustive</option>
									</select>
								</label>
							</div>
						</svelte:fragment>

						<div class="transactions-list column" class:scrollable-list={transactions.length > 7}>
							{#each transactions as transaction}
								{#if transactionProvider === 'Covalent'}
									<a class="card" id={transaction.tx_hash} href="#{transaction.tx_hash}">
										<EthereumTransactionCovalent
											{network}
											{transaction}
											{quoteCurrency}
											contextualAddress={address}
											{detailLevel}
											{showValues}
											{showFees}
											layout="inline"
										/>
									</a>
								{:else if transactionProvider === 'Moralis'}
									<a class="card" id={transaction.hash} href="#{transaction.hash}">
										<EthereumTransactionMoralis
											{network}
											{transaction}
											{quoteCurrency}
											contextualAddress={address}
											{detailLevel}
											{showValues}
											{showFees}
											layout="inline"
										/>
									</a>
								{:else if transactionProvider === 'Etherspot'}
									<a class="card" id={transaction.hash} href="#{transaction.hash}">
										<EthereumTransactionEtherspot
											{network}
											{transaction}
											{quoteCurrency}
											contextualAddress={address}
											{detailLevel}
											{showValues}
											{showFees}
											layout="inline"
										/>
									</a>
								{/if}
							{:else}
								<div class="card">No transactions yet.</div>
							{/each}
						</div>
					</EthereumTransactionsLoader>
				</div>
			{:else}{#key selectedToken}
				<div class="column" transition:fade>
					<!-- ERC-20 Transactions -->
					<EthereumTransactionsERC20Loader
						{network}
						{address}
						{provider}
						{transactionProvider}
						erc20Token={selectedToken}
						{quoteCurrency}
						includeLogs={detailLevel === 'exhaustive'}
						let:transactions
					>
						<svelte:fragment slot="header" let:status>
							<div class="bar">
								<h3>
									{selectedToken.name}
									(<TokenName erc20Token={selectedToken} />)
									Transactions
									{#if status === 'resolved'}({transactions.length}{transactions.length === 100 ? '+' : ''}){/if}
								</h3>
								{#if detailLevel !== 'exhaustive'}
									<label>
										<input type="checkbox" bind:checked={showFees}>
										<span>Show Fees</span>
									</label>
								{/if}
								<label>
									<span>View</span>
									<select bind:value={detailLevel}>
										<option value="summary">Summary</option>
										<option value="detailed">Detailed</option>
										<option value="exhaustive">Exhaustive</option>
									</select>
								</label>
								<button on:click={() => selectedToken = undefined}>Back</button>
							</div>
						</svelte:fragment>

						<div class="transactions-list column" class:scrollable-list={transactions.length > 7}>
							{#each transactions as erc20TokenTransaction}
								{#if transactionProvider === 'Covalent'}
									<a class="card" id={erc20TokenTransaction.tx_hash} href="#{erc20TokenTransaction.tx_hash}">
										<EthereumTransactionCovalent
											{network}
											{erc20TokenTransaction}
											{quoteCurrency}
											contextualAddress={address}
											{detailLevel}
											{showValues}
											{showFees}
											layout="inline"
										/>
									</a>
								{:else if transactionProvider === 'Moralis'}
									<a class="card" id={erc20TokenTransaction.hash} href="#{erc20TokenTransaction.hash}">
										<EthereumTransactionMoralis
											{network}
											{erc20TokenTransaction}
											{quoteCurrency}
											contextualAddress={address}
											{detailLevel}
											{showValues}
											{showFees}
											layout="inline"
										/>
									</a>
								{/if}
							{:else}
								<div class="card">No transactions yet.</div>
							{/each}
						</div>
					</EthereumTransactionsERC20Loader>
				</div>
			{/key}{/if}
		</div>

		{#if balances?.length}
			<CovalentPriceChart
				historicalPriceProvider={$preferences.historicalPriceProvider}
				quoteCurrency={$preferences.quoteCurrency}
				chainID={network.chainId}
				currencies={
					selectedToken ? [selectedToken.tokenAddress] :
					balances ? balances.map(tokenWithBalance => tokenWithBalance.contract_address) :
					[]
				}
				{priceScale}
			>
				<svelte:fragment slot="header">
					<hr>

					<div class="bar">
						<h3>Chart</h3>
						<label>
							<span>Price Scale</span>
							<select bind:value={priceScale}>
								<option value="logarithmic">Logarithmic</option>
								<option value="linear">Linear</option>
								<option value="linearFromZero">Linear From Zero</option>
							</select>
						</label>
					</div>
				</svelte:fragment>
			</CovalentPriceChart>
		{/if}
	</EnsResolutionLoader>
</div>