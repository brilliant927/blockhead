<script lang="ts">
	import type { Ethereum } from '../data/ethereum/types'


	export let network: Ethereum.Network
	export let provider: Ethereum.Provider
	export let blockNumber: number


	// import Loader from './Loader.svelte'
	// import EthereumBlockNumber from './EthereumBlockNumber.svelte'
	// import TokenIcon from './TokenIcon.svelte'

	import EthereumLatestBlockNumber from './EthereumLatestBlockNumber.svelte'
</script>

<div class="bar">
	<h3>Block Height</h3>
	<!-- <span class="card-annotation">{$preferences.rpcNetwork}</span> -->
</div>
<!-- <Loader
	loadingMessage="Retrieving statistics..."
	fromPromise={provider && (() => new Promise(r => provider.once('block', r)))}
>
	<TokenIcon slot="loadingIcon" symbol={network.nativeCurrency.symbol} />

	<p class="centered">
		<span>The {network.name} blockchain is </span>
		<EthereumBlockNumber {network} {blockNumber} />
		<span> blocks long.</span>
	</p>
</Loader> -->


<p class="centered">
	<span>The {network.name} blockchain is </span>
	<EthereumLatestBlockNumber {network} {provider} />
	<span> blocks long.</span>
</p>