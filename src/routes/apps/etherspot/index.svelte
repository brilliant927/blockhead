<script lang="ts">
	import { blockchainAppsBySlug } from '../../../data/blockchain-apps'


	import TransferAssets from '../../../components/TransferAssets.svelte'


	import { cardStyle } from '../../../utils/card-background'
	import { fly } from 'svelte/transition'
</script>


<section class="column" in:fly={{x: 100}} out:fly={{x: -100}}>
	<div class="card" style={cardStyle(blockchainAppsBySlug['etherspot'].colors)}>
		<div class="bar">
			<h3>Cross-Chain Transfer</h3>
			<div class="card-annotation">Etherspot</div>
		</div>

		<hr>

		<TransferAssets
			transferSolution="Etherspot"
		/>
	</div>
</section>