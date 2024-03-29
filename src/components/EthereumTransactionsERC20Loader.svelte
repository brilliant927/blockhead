<script lang="ts">
	import type { Ethereum } from '../data/ethereum/types'
	import type { TickerSymbol } from '../data/currency/currency'
	import { preferences } from '../data/ethereum/preferences'

	export let network: Ethereum.Network
	export let address: Ethereum.Address
	export let provider: Ethereum.Provider

	export let erc20Token: Ethereum.ERC20Token

	export let quoteCurrency: TickerSymbol
	export let transactionProvider
	export let includeLogs = true

	$: quoteCurrency = $$props.quoteCurrency || $preferences.quoteCurrency
	$: transactionProvider = $$props.transactionProvider || $preferences.transactionProvider


	import { getERC20TokenTransfers } from '../data/analytics/covalent'
	import { chainCodeFromNetwork, MoralisWeb3Api } from '../data/moralis/moralis-web3-api'


	import Loader from './Loader.svelte'
</script>


{#if transactionProvider === 'Covalent'}
	<Loader
		loadingIcon={'/logos/covalent-logomark.svg'}
		loadingIconName={transactionProvider}
		loadingMessage="Retrieving ERC-20 transactions from {transactionProvider}..."
		errorMessage="Error retrieving ERC-20 transactions from {transactionProvider}"
		fromPromise={async () => 
			(await getERC20TokenTransfers({
				chainID: network.chainId,
				address,
				contractAddress: erc20Token.address,
				quoteCurrency
			}))
				.items
		}
		let:then={transactions}
		let:status
	>
		<slot name="header" slot="header" {status} {transactions} />
		<slot {transactions} />
	</Loader>
{:else if transactionProvider === 'Moralis'}
	<Loader
		loadingIcon={'/logos/moralis-logo.svg'}
		loadingIconName={transactionProvider}
		loadingMessage="Retrieving ERC-20 transactions from {transactionProvider}..."
		errorMessage="Error retrieving ERC-20 transactions from {transactionProvider}"
		fromPromise={async () => {
			const { result: transactions, page, page_size, total } = (await MoralisWeb3Api.address.getTokenTransfers({
				chain: chainCodeFromNetwork(network),
				from_block: 0,
				// to_block: ,
				// offset,
				// limit,
				address
			}))

			return includeLogs
				? await Promise.all(transactions.map(async transaction => ({
					value_: transaction.value,
					...(await MoralisWeb3Api.transaction.getTransaction({
						chain: chainCodeFromNetwork(network),
						transactionHash: transaction.hash
					}))
				})))
				: transactions
		}}
		let:then={transactions}
		let:status
	>
		<slot name="header" slot="header" {status} {transactions} />
		<slot {transactions} />
	</Loader>
{/if}