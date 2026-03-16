<script>
	import { onDestroy } from 'svelte';
	import { onMount } from 'svelte';

	// --- STATE ---
	let cards = [1, 2, 3, 4, 5];
	let nextId = 6;
	let removingId = null;

	// --- FEEDBACK STATE ---
	let isClicking = false;

	// --- FLIP STATE ---
	let flippedCards = [];

	// Auto-Clicker State
	let isAuto = false;
	let autoInterval;

	// Shape State
	let shapeMode = 'arc';
	const shapes = ['arc', 'roller', 'flat'];

	// Color State
	let colorIndex = 0;
	const palettes = [
		{ name: 'System', vars: '' },
		{
			name: 'Ocean',
			vars: '--card-c1: #48cae4; --card-c2: #0077b6; --card-c3: #023e8a;',
			primary: '#0077b6'
		},
		{
			name: 'Forest',
			vars: '--card-c1: #95d5b2; --card-c2: #40916c; --card-c3: #1b4332;',
			primary: '#40916c'
		},
		{
			name: 'Crimson',
			vars: '--card-c1: #ffccd5; --card-c2: #c9184a; --card-c3: #800f2f;',
			primary: '#c9184a'
		},
		{
			name: 'Gold',
			vars: '--card-c1: #ffea00; --card-c2: #ffaa00; --card-c3: #ff7b00;',
			primary: '#ffaa00'
		}
	];

	let currentPrimaryColor = '#f25c54';

	function updateSystemColor() {
		if (palettes[colorIndex].name === 'System' && typeof window !== 'undefined') {
			const style = getComputedStyle(document.documentElement);
			currentPrimaryColor = style.getPropertyValue('--card-c2').trim() || '#f25c54';
		}
	}

	onMount(() => {
		updateSystemColor();
		window.matchMedia('(prefers-color-scheme: dark)').addListener(updateSystemColor);
	});

	$: {
		if (palettes[colorIndex].primary) {
			currentPrimaryColor = palettes[colorIndex].primary;
		} else {
			updateSystemColor();
		}
	}

	function simulateClick() {
		isClicking = true;
		setTimeout(() => {
			isClicking = false;
		}, 120);
	}

	// --- ACTIONS ---
	function addCard() {
		if (cards.length < 52) {
			cards = [...cards, nextId++];
			return true;
		}
		if (isAuto) toggleAuto();
		return false;
	}

	function removeCard() {
		if (cards.length > 0 && removingId === null) {
			const idToRemove = cards[cards.length - 1];
			removingId = idToRemove;
			setTimeout(() => {
				cards = cards.filter((c) => c !== idToRemove);
				removingId = null;
			}, 200);
		}
	}

	function toggleAuto() {
		if (isAuto) {
			clearInterval(autoInterval);
			isAuto = false;
		} else {
			isAuto = true;
			simulateClick();
			addCard();

			autoInterval = setInterval(() => {
				simulateClick();
				addCard();
			}, 600);
		}
	}

	function flipAllCards() {
		if (cards.length === 0) return;

		if (flippedCards.length > 0) {
			flippedCards = [];
			setTimeout(() => startFlipSequence(), 500);
		} else {
			startFlipSequence();
		}
	}

	function startFlipSequence() {
		const flipDelay = 50;
		let finalDelay = 0;

		cards.forEach((cardId, index) => {
			const delay = index * flipDelay;
			finalDelay = delay;

			setTimeout(() => {
				flippedCards = [...flippedCards, cardId];
			}, delay);
		});

		if (cards.length > 0) {
			finalDelay += 1000;
			setTimeout(() => {
				flippedCards = [];
			}, finalDelay);
		}
	}

	function fillDeck() {
		if (isAuto) toggleAuto();
		let newCards = [...cards];
		while (newCards.length < 52) {
			newCards.push(nextId++);
		}
		cards = newCards;
	}

	function clearDeck() {
		if (isAuto) toggleAuto();
		cards = [];
	}

	function toggleShape() {
		const currentIndex = shapes.indexOf(shapeMode);
		const nextIndex = (currentIndex + 1) % shapes.length;
		shapeMode = shapes[nextIndex];
	}

	function toggleColor() {
		colorIndex = (colorIndex + 1) % palettes.length;
	}

	onDestroy(() => {
		if (autoInterval) clearInterval(autoInterval);
		if (typeof window !== 'undefined') {
			window.matchMedia('(prefers-color-scheme: dark)').removeListener(updateSystemColor);
		}
	});
</script>

<a href="/liquid-gradient" class="next-button">Next →</a>

<main class="page-container">
	<h1 class="background-title">Sebastiaan Hagoort</h1>

	<div class="content-wrapper">
		<div class="demo-info animate-entry">
			<h2 class="animate-entry delay-1">Fanned Card Deck (Light and Dark mode)</h2>
			<p class="animate-entry delay-2">
				Controls: <strong>{cards.length}</strong> / 52 Cards
			</p>

			<div class="controls animate-entry delay-3">
				<button
					on:click={addCard}
					style="background-color: {currentPrimaryColor}; border-color: {currentPrimaryColor}"
					>Add +</button
				>
				<button
					on:click={removeCard}
					style="background-color: {currentPrimaryColor}; border-color: {currentPrimaryColor}"
					>Remove -</button
				>
				<button
					class="auto-btn"
					class:active={isAuto}
					class:clicking={isClicking}
					on:click={toggleAuto}
				>
					{isAuto ? 'Stop' : 'Auto Click'}
					{#if isAuto}
						<div class="cursor-feedback"></div>
					{/if}
				</button>

				<button
					on:click={flipAllCards}
					style="background-color: #edb066; border-color: #ffc175; font-weight: 900;"
				>
					Flip Cards
				</button>
			</div>

			<div class="controls animate-entry delay-4">
				<button
					class="secondary"
					on:click={fillDeck}
					style="background-color: {currentPrimaryColor}">Full Stack</button
				>
				<button
					class="secondary"
					on:click={clearDeck}
					style="background-color: {currentPrimaryColor}">Clear Deck</button
				>
			</div>

			<div class="controls animate-entry delay-4">
				<button
					class="secondary shape-btn"
					on:click={toggleShape}
					class:arc={shapeMode === 'arc'}
					class:roller={shapeMode === 'roller'}
					class:flat={shapeMode === 'flat'}
				>
					Shape: {shapeMode.toUpperCase()}
				</button>

				<button
					class="secondary color-btn"
					on:click={toggleColor}
					style="background-color: {currentPrimaryColor}"
				>
					Color: {palettes[colorIndex].name}
				</button>
			</div>
		</div>

		<div
			class="hand {shapeMode} animate-entry delay-4"
			style="--total: {cards.length}; {palettes[colorIndex].vars};"
		>
			{#each cards as cardId, index (cardId)}
				<div
					class="card"
					class:added={index === cards.length - 1 && removingId === null}
					class:removing={cardId === removingId}
					class:flipped={flippedCards.includes(cardId)}
					style="
                        --i: {index}; 
                        z-index: {index}; 
                        --hue: {index * 10}deg; 
                    "
				></div>
			{/each}
		</div>
	</div>
</main>

<style>
	/* 2. ISOLATED STYLES
       All :root and :global(body) styles are moved to .page-container 
    */
	.page-container {
		/* LIGHT MODE DEFAULTS */
		--bg-outer: #1d3a2d;
		--bg-inner: #3a7559;
		--text-color: #f0f0f0;
		--title-opacity: 0.1;

		--card-c1: #f7b267;
		--card-c2: #f25c54;
		--card-c3: #f79d65;

		--card-width: min(30vw, 200px);
		--card-height: min(42vw, 280px);

		/* BODY SIMULATION */
		font-family: 'Poppins', sans-serif;
		background-color: var(--bg-outer);
		background-image: radial-gradient(circle, var(--bg-inner) 0%, var(--bg-outer) 100%);
		color: var(--text-color);
		margin: 0;
		padding: 0;

		/* Ensures it fills the screen like body */
		min-height: 100vh;
		width: 100%;
		overflow-x: hidden;

		transition:
			background 0.5s ease,
			color 0.5s ease;
	}

	/* DARK MODE - scoped to container */
	@media (prefers-color-scheme: dark) {
		.page-container {
			/* DARK MODE DEFAULTS */
			--bg-outer: #0f0f0f;
			--bg-inner: #2c3e50;
			--text-color: #e0e0e0;
			--title-opacity: 0.15;

			--card-c1: #4cc9f0;
			--card-c2: #4361ee;
			--card-c3: #3a0ca3;
		}
	}

	/* Rest of the CSS remains largely the same, just scoped by Svelte automatically */

	.next-button {
		position: fixed;
		top: 15px;
		right: 20px;
		z-index: 51;
		background: none;
		border: 2px solid #f0f0f0;
		color: #f0f0f0;
		padding: 8px 15px;
		text-decoration: none;
		border-radius: 5px;
		font-weight: bold;
		font-family: 'Poppins', sans-serif;
		transition: background-color 0.2s, color 0.2s;
	}
	.next-button:hover {
		background-color: #f0f0f0;
		color: #121212;
	}

	.background-title {
		position: fixed;
		top: 50%;
		left: 0.5vw;
		transform: translateY(-50%);
		font-size: 7.5vw;
		font-weight: 900;
		text-transform: uppercase;
		margin: 0;
		pointer-events: none;
		z-index: 0;
		color: white;
		opacity: var(--title-opacity);
		overflow: hidden;
		border-right: 0.15em solid #daa520;
		white-space: nowrap;
		width: 0;
		max-width: fit-content;
		animation:
			typing 3.5s steps(30, end) forwards,
			blink-caret 0.75s step-end infinite;
	}
	@keyframes typing {
		from {
			width: 0;
		}
		to {
			width: 100%;
		}
	}
	@keyframes blink-caret {
		from,
		to {
			border-color: transparent;
		}
		50% {
			border-color: #daa520;
		}
	}

	.content-wrapper {
		position: relative;
		z-index: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 2rem;
		padding: 2rem 1rem;
		/* Ensure content wrapper doesn't collapse */
		min-height: 100vh;
	}
	.demo-info {
		max-width: 650px;
		text-align: center;
		position: relative;
		z-index: 100;
	}
	h2 {
		margin: 0 0 0.5rem 0;
	}

	.animate-entry {
		opacity: 0;
		transform: translateY(20px);
		animation: fadeInUp 0.8s ease forwards;
	}
	.delay-1 {
		animation-delay: 0.2s;
	}
	.delay-2 {
		animation-delay: 0.3s;
	}
	.delay-3 {
		animation-delay: 0.4s;
	}
	.delay-4 {
		animation-delay: 0.6s;
	}
	@keyframes fadeInUp {
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	/* --- CONTROLS --- */
	.controls {
		position: relative;
		z-index: 101;
		margin-top: 1rem;
		display: flex;
		gap: 0.5rem;
		justify-content: center;
		flex-wrap: wrap;
	}

	button {
		font-family: inherit;
		font-size: 0.9rem;
		font-weight: 600;
		padding: 0.6rem 1.2rem;
		color: white;
		border-radius: 8px;
		cursor: pointer;
		transition: all 0.2s ease;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.4);
		min-width: 80px;
		position: relative;
		background-color: #a31f1f;
		border: 2px solid #daa520;
	}

	button:hover {
		transform: translateY(-3px);
		box-shadow: 0 6px 12px rgba(0, 0, 0, 0.5);
		filter: brightness(1.2);
	}

	button:active {
		transform: translateY(-1px);
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.4);
	}

	button.auto-btn {
		background-color: #1f6ca3;
		border-color: #7cd6ff;
		overflow: hidden;
	}
	button.auto-btn.active {
		background-color: #2ea31f;
		border-color: #7cff89;
		animation: pulse 1s infinite;
	}

	/* Auto-Clicking Feedback */
	.auto-btn.clicking {
		transform: translateY(-1px) !important;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.4) !important;
		transition: none;
	}

	/* Cursor styles */
	.cursor-feedback {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background-color: rgba(255, 255, 255, 0.2);
		opacity: 0;
		pointer-events: none;

		--cursor-size: 14px;
		--cursor-color: #f0f0f0;

		background-image:
			radial-gradient(
				circle at 80% 80%,
				var(--cursor-color) 0,
				var(--cursor-color) 4px,
				transparent 4px
			),
			linear-gradient(45deg, var(--cursor-color) 0, var(--cursor-color) 2px, transparent 2px);
		background-size: var(--cursor-size) var(--cursor-size);
		background-repeat: no-repeat;

		transform: translate(-100%, -100%);
		transition:
			transform 0.05s ease-out,
			opacity 0.1s;
	}

	.auto-btn.clicking .cursor-feedback {
		animation: auto-click-move 0.6s linear infinite;
		opacity: 1;
	}

	@keyframes auto-click-move {
		0% {
			transform: translate(-100%, -100%) scale(1);
		}
		10% {
			transform: translate(-50%, -50%) scale(1);
		}
		20% {
			transform: translate(10%, 10%) scale(0.95);
			opacity: 1;
		}
		30% {
			transform: translate(10%, 10%) scale(0.95);
			opacity: 1;
		}
		40% {
			transform: translate(50%, 50%) scale(1);
			opacity: 0.5;
		}
		100% {
			transform: translate(-100%, -100%) scale(1);
			opacity: 0;
		}
	}

	/* Secondary buttons */
	.secondary {
		border: 2px solid white;
	}
	.secondary:hover {
		filter: brightness(1.2);
	}

	/* --- SHAPE BUTTON STYLING --- */
	.shape-btn {
		min-width: 140px;
		transition:
			transform 0.2s ease,
			border-radius 0.4s cubic-bezier(0.4, 0, 0.2, 1);
		background-color: #5b2c91 !important;
		border-color: #b07cff !important;
	}

	/* Shape Transformations: card shape */
	.shape-btn.arc {
		border-radius: 8px 8px 16px 16px;
		transform: rotate(-3deg);
	}
	.shape-btn.roller {
		border-radius: 50%; /* Circle */
		transform: rotate(15deg);
	}
	.shape-btn.flat {
		border-radius: 4px; /* Flatter */
		transform: rotate(0deg);
	}

	/* --- CARDS  --- */
	.hand {
		position: relative;
		width: var(--card-width);
		height: var(--card-height);
		margin-inline: auto;
		perspective: 1000px;
		pointer-events: none;
		transition: all 0.5s cubic-bezier(0.4, 0, 0.2, 1);
		margin-top: 5rem;
	}
	.hand.roller {
		margin-top: 18rem;
	}

	.card {
		pointer-events: auto;
		--center-index: calc((var(--total) - 1) / 2);
		--offset: calc(var(--i) - var(--center-index));
		--abs-offset: max(var(--offset), var(--offset) * -1);
		position: absolute;
		width: 100%;
		height: 100%;
		border-radius: 12px;
		box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
		border: 3px solid white;

		transform-style: preserve-3d;
		transition: transform 0.5s cubic-bezier(0.4, 0, 0.2, 1);
		transform-origin: bottom center;

		background-image: none;
		background-color: transparent;
	}

	/* Card Back Face */
	.card::before {
		content: '';
		position: absolute;
		width: 100%;
		height: 100%;
		border-radius: 12px;
		backface-visibility: hidden;
		transition: transform 0.5s;
		transform: translateZ(-1px);

		/* Back Pattern  */
		background-image:
			repeating-linear-gradient(
				-45deg,
				transparent,
				transparent 15px,
				var(--card-c1) 15px,
				var(--card-c1) 50%
			),
			repeating-linear-gradient(
				45deg,
				var(--card-c2),
				var(--card-c2) 15px,
				var(--card-c3) 15px,
				var(--card-c3) 50%
			);
		background-size:
			30px 30px,
			22px 22px;
		filter: hue-rotate(var(--hue));
	}

	/* Card Front Face */
	.card::after {
		content: '';
		position: absolute;
		width: 100%;
		height: 100%;
		border-radius: 12px;
		backface-visibility: hidden;
		transition: transform 0.5s;
		transform: rotateY(180deg) translateZ(1px);
		background-image: url('/Joker.png');
		background-size: cover;
		background-repeat: no-repeat;
		background-position: center;
		background-color: white;
	}

	.card.flipped::before {
		transform: rotateY(180deg) translateZ(-1px);
	}
	.card.flipped::after {
		transform: rotateY(0deg) translateZ(1px);
	}

	.hand.arc .card {
		transform: rotate(calc(8deg * var(--offset))) translateY(calc(4px * var(--abs-offset)));
	}
	.hand.roller .card {
		transform: rotate(calc(15deg * var(--offset))) translateY(calc(12px * var(--offset)));
	}
	.hand.flat .card {
		transform: rotate(calc(3deg * var(--offset))) translateY(calc(1px * var(--abs-offset)));
	}

	.hand.arc .card.flipped {
		transform: rotate(calc(8deg * var(--offset))) translateY(calc(4px * var(--abs-offset)));
	}
	.hand.roller .card.flipped {
		transform: rotate(calc(15deg * var(--offset))) translateY(calc(12px * var(--offset)));
	}
	.hand.flat .card.flipped {
		transform: rotate(calc(3deg * var(--offset))) translateY(calc(1px * var(--abs-offset)));
	}

	/* Hover logic */
	.hand:hover .card {
		transform: rotate(calc(var(--rot-scale, 8) * var(--offset))) translateY(-50px);
	}
	.hand:hover .card.flipped {
		transform: rotate(calc(var(--rot-scale, 8) * var(--offset))) translateY(-50px);
	}

	/* Animation logic */
	.card.added {
		animation: deal-in 0.5s cubic-bezier(0.2, 1, 0.4, 1);
	}
	.card.removing {
		animation: deal-out 0.2s linear forwards;
	}
	@keyframes deal-in {
		from {
			opacity: 0;
			transform: translateY(100px) rotate(0deg);
		}
	}
	@keyframes deal-out {
		to {
			opacity: 0;
			transform: translateY(-100px) rotate(-5deg);
		}
	}
</style>
