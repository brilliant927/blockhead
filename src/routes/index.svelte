<script>
	import Preferences from '../components/Preferences.svelte'


	import { fly, scale, fade } from 'svelte/transition'
</script>


<style>
	main {
		--is-hovered: 0;
		--is-active: 0;
		--transition: 10s cubic-bezier(0, 1, 0, 1);

		--card-border-radius: 1.5rem;

		max-width: var(--one-column-width);
		text-align: center;
		gap: 2.5rem;

		min-height: 100vh;
		align-content: center;

		justify-items: center;

		perspective: 1000px;
	}
	main * {
		transform-style: preserve-3d;
	}

	header {
		display: grid;
		gap: var(--padding-inner);
		--padding-inner: 2.5rem;

		cursor: default;
		user-select: none;

		transform: rotateX(0.03turn);
		/* transform-origin: bottom; */
	}

	section {
		/* transform-origin: top; */
		/* transform: rotateX(0.01turn); */
		transform-origin: bottom;
	}
	.buttons {
		transform: rotateX(-0.01turn);
		transform-origin: top;
	}

	b {
		color: rgba(var(--rgb-light-dark-inverse), 0.8);
		text-shadow:
			0 1px 1px rgba(var(--rgb-light-dark-inverse), 0.15),
			0 0 2.5em rgba(var(--rgb-light-dark-inverse), 0.2);
	}


	.logo-wrapper {
		font-size: clamp(2em, calc(1.5em + 5vw), 5em);
		font-weight: 700;

		box-shadow:
			0 0 0 .15em rgba(var(--rgb-light-dark-inverse), calc(0.05 + var(--is-hovered) * 0.1)),
			inset 0 0 calc(0.05em + var(--is-hovered) * 0.15em) rgba(var(--rgb-light-dark-inverse), calc(0.1 + var(--is-hovered) * 0.1)),
			0 0 1em rgba(var(--rgb-light-dark-inverse), calc(0.15 + var(--is-hovered) * 0.1));
		border-radius: 0.3em;
		/* -webkit-background-clip: border-box; */
		/* text-shadow: 0 0 2em rgba(var(--rgb-light-dark-inverse), 0.5);
		text-shadow: 0 0 1em rgba(var(--rgb-light-dark-inverse), 0.15); */
		/* filter: drop-shadow(0 0 1em var(--primary-color), 0 0 1em rgba(var(--rgb-light-dark-inverse), 0.5)); */
		padding: 0 0.44em;
		justify-self: center;
		
		transition: var(--transition);
	}
	.logo-wrapper:hover, b:hover {
		--is-hovered: 1;
	}
	.logo-wrapper:active, b:hover:active {
		--is-hovered: 0;
		--is-active: 1;
		--transition: 0.15s;
	}
	.logo, b {
		--z: calc((1 - var(--is-active)) * 0.1em + var(--is-hovered) * 0.5rem);
		font-size: inherit;
		line-height: 1.3;
		transform: translateZ(var(--z));
		position: relative;
		display: inline-block;

		transition: var(--transition), font-size 0s;
	}
	.logo::after, b::after {
		color: rgba(var(--rgb-light-dark), 0.1);
		content: attr(data-text);
		transform: translateZ(calc(-1 * var(--z)));
		position: absolute;
		left: 0;
		text-shadow: 0 1px 0.05em rgba(var(--rgb-light-dark-inverse), 0.15);

		transition: var(--transition), font-size 0s;
	}
	@supports (-webkit-background-clip: text) {
		.logo {
			background: radial-gradient(rgba(var(--rgb-light-dark-inverse), calc(0.5 + var(--is-hovered) * 0.1)) calc(-20% - var(--is-light) * 100%), var(--primary-color));
			-webkit-background-clip: text;
			-webkit-text-fill-color: #ffffff20;
		}
	}

	.description, h3 {
		text-transform: uppercase;
		font-family: Pally;
		color: rgba(var(--rgb-light-dark-inverse), 0.4);
		letter-spacing: 0.2ch;
	}
	.description.top {
		font-size: 1em;
		letter-spacing: 0.45ch;
		line-height: 1.5;
	}
	.description.bottom {
		display: flex;
		flex-wrap: wrap;
		justify-content: center;
		gap: 0.2em;
		letter-spacing: 0.35ch;

		font-size: calc(1.33em * 0.8);
	}
	.description.bottom > :nth-child(2) {
		font-size: 0.75em;
	}
	.description.bottom b {
		font-size: calc(1em / 0.75);
	}

	.metaverse {
		background: linear-gradient(-45deg, rgb(17, 230, 230), rgb(153, 32, 233), rgb(236, 167, 18), rgb(90, 199, 132)); 
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
	}

	.buttons {
		display: flex;
		flex-wrap: wrap;
		gap: 1rem;
	    justify-content: center;
	}

	.columns {
		display: flex;
		align-items: stretch;
		flex-wrap: wrap;
		gap: 1rem;
		color: rgba(var(--rgb-light-dark-inverse), 0.7);
	}
	.columns > .card {
		justify-content: center;
		flex: 1 20rem;

		--padding-outer: 1.25rem;
		--padding-inner: 0.75rem;
	}
	.card:nth-child(3n + 1) {
		/* transform: translateZ(-1em) rotateY(-0.0075turn); */
		transform: translateZ(-1em);
		transform-origin: 80% center;
	}
	.card:nth-child(3n + 2) {
		transform: translateZ(-1em);
	}
	.card:nth-child(3n + 3) {
		/* transform: translateZ(-1em) rotateY(0.0075turn); */
		transform: translateZ(-1em);
		transform-origin: 20% center;
	}
	.card > * {
		transition: var(--transition);
	}
	.card:hover {
		transform: none;
	}
	/* .card:hover > * {
		transform: translateZ(1em);
	} */

	h3 {
		font-size: 1.2em;
	}

	@media (max-width: 56rem) {
		header {
			max-width: 36rem;
			transform: rotateX(0.01turn);
		}

		.description b {
			display: inline;
		}

		.description.top br {
			display: none;
		}

		.description.bottom {
			gap: 1.25rem;
		}

		.description.bottom :nth-child(2) {
			max-width: 35rem;
		}

		.columns .card {
			transform: none;
		}
	}

	button {
		font-size: 1.3em;
	}

	footer {
		color: rgba(var(--rgb-light-dark-inverse), 0.7);
	}
</style>

<svelte:head>
	<title>Blockhead</title>
</svelte:head>

<main in:fly={{x: 300}} out:fly={{x: -300}}>
	<header>
		<h2 class="description top" transition:scale={{delay: 150}}>
			<b>Track</b>,
			<b>visualize</b>
			&
			<b>explore</b>
			<br>
			<!-- the
			<b>blockchain</b>/<b>web 3.0</b>
			<b>metaverse</b> -->
			<!-- <b>blockchains</b> & -->
			the
			<b>decentralized world wide web</b>
			with
		</h2>
		<div class="logo-wrapper" transition:scale={{delay: 200}}>
			<h1 class="logo" data-text="Blockhead">Blockhead</h1>
		</div>
		<p class="description bottom" transition:scale={{delay: 250}}>
			<span>
				 a
				<b>crypto portfolio</b>,
				<b>DeFi dashboard</b>,
				<b>NFT viewer</b>
				&
				<b>data explorer</b>
			</span>
			<span>
				for
				<b style="color: var(--ethereum-blue)">Ethereum</b>,
				<b style="color: var(--polygon-purple)">Polygon</b>,
				<b style="color: var(--avalanche-red)">AVAX</b>,
				<b style="color: var(--binance-gold)">BSC</b>,
				<b style="color: var(--xdai-teal); text-transform: none">xDAI</b>,
				<b style="color: var(--fantom-blue)">Fantom</b>
				& the
				<b class="metaverse">Web 3.0 Metaverse</b>
				<!-- <b style="color: var(--bitcoin-gold)">EVM Multiverse</b> -->
			</span>
		</p>
	</header>

	<section>
		<div class="columns">
			<div class="card" transition:scale={{delay: 350}}>
				<h3><b>Track</b> your crypto</h3>
				<hr>
				<p>See <b>tokens</b>, <b>NFTs</b>, & <b>DeFi balances</b> across your blockchain accounts in one place.</p>
			</div>
			<div class="card" transition:scale={{delay: 500}}>
				<h3><b>Visualize</b> your activity</h3>
				<hr>
				<p>Contextualize your web 3.0 interactions with <b>intuitive charts</b> & <b>visualizations</b>.</p>
			</div>
			<div class="card" transition:scale={{delay: 400}}>
				<h3><b>Explore</b> the metaverse</h3>
				<hr>
				<p>Find <b>apps</b>, <b>services</b> & <b>communities</b> enabling <b>portable, shared experiences</b>.</p>
			</div>
			<div class="card" transition:scale={{delay: 600}}>
				<h3>On <b>Unstoppable</b> Web 3.0</h3>
				<hr>
				<p>Built upon <b>math</b>, <b>cryptography</b> & open, <b>incentive-aligned</b> computer networks.</p>
			</div>
			<div class="card" transition:scale={{delay: 450}}>
				<h3><b>You</b> control the data</h3>
				<hr>
				<p><b>Choose which data sources</b> power this user interface in the <b>Preferences</b> below!</p>
			</div>
			<div class="card" transition:scale={{delay: 700}}>
				<h3>To <b>Ethereum</b> & Beyond!</h3>
				<hr>
				<!-- <p>Hop between any of the supported <b>Ethereum Virtual Machine</b>-based chains!</p> -->
				<p>Hop between Ethereum & any supported <b>Ethereum Virtual Machine</b>-based chain!</p>
			</div>
		</div>
	</section>

	<div class="buttons">
		<a href="/portfolio" transition:scale={{delay: 800}}><button>Create Portfolio</button></a>
		<a href="/explorer" transition:scale={{delay: 850}}><button>Explore Blockchains</button></a>
		<a href="/apps" transition:scale={{delay: 900}}><button>Browse Apps</button></a>
	</div>
	
	<footer transition:scale={{delay: 1000}}>
		<p><strong>Blockhead</strong> • created by <a href="https://darryl-yeo.com/blockhead">Darryl Yeo</a> • 2020 – 2021</p>
		<small><a href="https://gitcoin.co/grants/2966/blockhead" target="_blank">Gitcoin Grants</a> • <a href="https://discord.gg/966eXqqq7N" target="_blank">Discord</a> • <a href="https://twitter.com/0xblockhead" target="_blank">Twitter</a></small>
		<!-- <img src="/Blockhead-Logo.svg" alt="Blockhead Logo" width="40" /> -->
	</footer>
</main>

<Preferences relevantPreferences={['theme', 'rpcNetwork', 'quoteCurrency']}/>