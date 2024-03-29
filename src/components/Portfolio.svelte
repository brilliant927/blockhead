<script lang="ts">
	import type { Ethereum } from '../data/ethereum/types'
	import type { DeFiProvider } from '../data/defi-provider'
	import type { QuoteCurrency } from '../data/currency/currency'

	import { Account } from '../data/ethereum/portfolio-accounts'


	// Portfolio management

	export let name = ''
	export let accounts: Account[]

	export let editable = false

	enum State {
		Idle = 'idle',
		Editing = 'editing',
		Adding = 'adding'
	}
	let state = State.Idle
	$: if(editable && name === '')
		state = State.Editing


	import { createEventDispatcher } from 'svelte'
	const dispatch = createEventDispatcher()


	// Wallet management

	function isValid(address){
		return address !== ''
	}

	let newWalletAddress = ''

	function addAccount(newWalletAddress){
		if(!isValid(newWalletAddress))
			return

		if(accounts.some(account => account.id.toLowerCase() === newWalletAddress.toLowerCase()))
			return

		const newAccount = new Account(newWalletAddress)
		accounts = [newAccount, ...accounts]
		quoteTotals = [, ...quoteTotals]
	}

	function deleteAccount(i){
		delayStartIndex = i
		accounts = [...accounts.slice(0, i), ...accounts.slice(i + 1)]
		quoteTotals = [...quoteTotals.slice(0, i), ...quoteTotals.slice(i + 1)]
	}
	
	
	// Balances view options
	
	export let provider: Ethereum.Provider
	export let defiProvider: DeFiProvider
	export let tokenBalancesProvider
	export let nftProvider
	export let quoteCurrency: QuoteCurrency
	
	let showValues: 'original' | 'converted' | 'both' = 'original'
	let sortBy: 'value-descending' | 'value-ascending' | 'ticker-ascending' = 'value-descending'
	let showSmallValues = false
	let showUnderlyingAssets = false
	let showNFTMetadata = false


	// Computed Values
	let quoteTotals = []
	export let quoteTotal
	$: quoteTotal = quoteTotals.reduce((sum, quoteTotal) => sum + quoteTotal, 0)


	// Options menu
	let showOptions = true
	const toggleShowOptions = () => showOptions = !showOptions


	let delayStartIndex = 0

	import AddressField from './AddressField.svelte'
	import Loading from './Loading.svelte'
	import PortfolioAccount from './PortfolioAccount.svelte'
	import TokenBalance from './TokenBalance.svelte'
	import { flip } from 'svelte/animate'
	import { scale } from 'svelte/transition'
import { availableNetworks } from '../data/ethereum/networks';
</script>

<style>
	.edit-controls {
		flex: 0 auto;
		font-size: 0.7em;
	}

	.options {
		position: sticky;
		bottom: 0;

		margin: 0 calc(-1 * var(--padding-outer));
		z-index: 1;

		font-size: 0.8em;

		-webkit-backdrop-filter: var(--overlay-backdrop-filter);
		backdrop-filter: var(--overlay-backdrop-filter);
	}

	form {
		display: contents;
	}

	.account-total-value {
		font-size: 1.21em;
	}

	.portfolio h1 {
		flex: 1 16rem;
	}
</style>


<div class="portfolio column">
	<header class="bar">
		<slot name="title">
			<h1 class="row" on:dblclick={editable && (() => state = State.Editing)}>
				{#if state !== State.Editing}
					{name || '[Untitled Portfolio]'}
				{:else}
					<form on:submit={() => state = State.Idle}>
						<input type="text" bind:value={name} placeholder="My Portfolio" autofocus />
					</form>
				{/if}
			</h1>
		</slot>

		{#if quoteTotals.length && state !== State.Editing}
			<span class="account-total-value" transition:scale>
				<TokenBalance symbol={quoteCurrency} balance={quoteTotal} showPlainFiat={true} />
			</span>
		{/if}

		{#if editable}
			<div class="stack">
				{#if state !== State.Editing}
					<div class="bar" transition:scale>
						{#if state === State.Idle}
							<button on:click={() => state = State.Adding} transition:scale>+ Add Wallet</button>
						{/if}
						<button on:click={() => state = State.Editing}>Edit</button>
					</div>
				{:else}
					<div class="bar" transition:scale>
						<button class="destructive" on:click={() => dispatch('delete')}>Delete Portfolio</button>
						<button on:click={() => state = State.Idle}>Done</button>
					</div>
				{/if}
			</div>
		{/if}
		<!-- <button on:click={toggleShowOptions}>Options</button> -->

		<slot name="actions"></slot>
	</header>

	{#if accounts}
		{#if state === State.Adding || !accounts.length}
			<div class="stack" transition:scale>
				{#if state === State.Adding}
					<div class="card" transition:scale>
						<div class="bar">
							<div>
								<h3>Add Wallet</h3>
							</div>
							<small>{availableNetworks.map(network => network.name).join(', ')}</small>
						</div>
						<div class="bar">
							<form class="bar" on:submit|preventDefault={() => {addAccount(newWalletAddress); state = State.Idle; newWalletAddress = ''}}>
								<AddressField bind:address={newWalletAddress} autofocus required/>
								<button class="medium" disabled={!isValid(newWalletAddress)}>Add</button>
							</form>
							<button class="medium" on:click={() => state = State.Idle}>Cancel</button>
						</div>
					</div>
				{:else if !accounts.length}
					<div class="card" transition:scale|local>
						<h3>Your Blockhead Portfolio is empty!</h3>
						{#if editable}
							<p>You can <a on:click={() => isAddingWallet = true}>add a new wallet address manually</a>, or import an address by connecting a wallet service!</p>
						{/if}
					</div>
				{/if}
			</div>
		{/if}
		{#each accounts as {id, type, nickname, networks}, i (id)}
			<section class="card" transition:scale|local animate:flip|local={{duration: 300, delay: Math.abs(delayStartIndex - i) * 50}}>
				<PortfolioAccount
					addressOrENSName={id}
					{type}
					bind:nickname
					bind:showNetworks={networks}

					{provider}
					{defiProvider}
					{tokenBalancesProvider}
					{nftProvider}
					{quoteCurrency}

					{showValues}
					{sortBy}
					{showSmallValues}
					{showUnderlyingAssets}
					{showNFTMetadata}
					isEditing={state === State.Editing}

					bind:quoteTotal={quoteTotals[i]}
				>
					{#if state === State.Editing}
						<div class="row edit-controls" transition:scale>
							<button class="destructive" on:click={() => deleteAccount(i)}>Delete Account</button>
						</div>
					{/if}
				</PortfolioAccount>
			</section>
		{/each}
	{:else}
		<slot name="loading">
			<Loading>Loading your accounts...</Loading>
		</slot>
	{/if}

	{#if showOptions && accounts.length && state !== State.Editing}
		<div class="card bar options" transition:scale>
			<div class="row">
				<h3>Show</h3>
				<label>
					<select bind:value={showValues}>
						<option value="original">Balances</option>
						<option value="converted">Quotes</option>
						<option value="both">Balances + Quotes</option>
						<!-- <option value="original">Token Amounts</option>
						<option value="converted">Quote Values</option>
						<option value="both">Amounts and Values</option> -->
					</select>
				</label>
				<label>
					<input type="checkbox" bind:checked={showSmallValues}>
					<span>Small Values</span>
				</label>
				<label>
					<input type="checkbox" bind:checked={showUnderlyingAssets}>
					<span>Underlying Assets</span>
				</label>
				<label>
					<input type="checkbox" bind:checked={showNFTMetadata}>
					<span>NFT Metadata</span>
				</label>
			</div>
			<div class="row">
				<h3>Sort</h3>
				<label>
					<select bind:value={sortBy}>
						<option value="ticker-ascending">Alphabetical</option>
						<option value="value-descending">Highest Value</option>
						<option value="value-ascending">Lowest Value</option>
					</select>
				</label>
			</div>
		</div>
	{/if}
</div>