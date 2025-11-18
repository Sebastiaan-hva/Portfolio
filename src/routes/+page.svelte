<script>
	import { onMount } from 'svelte';

	// --- State Management: Ensure proper initialization for SSR ---

	let isLoading = true;

	const fullName = 'Sebastiaan Hagoort';

	const nameParts = fullName.split(' ');

	// --- Theme Logic: Now maps to 'dark' or 'light' ---

	let theme = 'dark'; // Initialize outside functions for reliable SSR

	function getSystemTheme() {
		// Must check if `window` exists, as it doesn't on the server (SSR)

		if (typeof window !== 'undefined') {
			return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
		}

		// Default to dark during SSR

		return 'dark';
	}

	onMount(() => {
		// Set initial theme after component mounts (runs client-side)

		theme = getSystemTheme();

		if (typeof window !== 'undefined') {
			const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');

			const handler = () => {
				theme = getSystemTheme();
			};

			mediaQuery.addEventListener('change', handler);

			// Cleanup function

			return () => mediaQuery.removeEventListener('change', handler);
		}
	});

	// --- Start Animation Logic ---

	// The timeout must be handled separately as it controls the isLoading state

	onMount(() => {
		setTimeout(() => {
			isLoading = false;
		}, 2500);
	});

	const links = [
		{ title: 'PROJECTS', href: '/projects', className: 'panel-projects', bgKey: 'key-projects' },

		{ title: 'ABOUT ME', href: '/about-me', className: 'panel-about-me', bgKey: 'key-about' },

		{
			title: 'DIGITAL GARDEN',

			href: '/digital-garden',

			className: 'panel-garden',

			bgKey: 'key-garden'
		}
	];
</script>

{#if isLoading}
	<div class="loading-screen" data-theme={theme}>
		<h1 class="initial-name">
			{#each nameParts as word}
				<span class="name-word">{word}</span>
			{/each}
		</h1>
	</div>
{:else}
	<div class="accordion-container" data-theme={theme}>
		{#each links as link}
			<a href={link.href} class="accordion-panel {link.className}">
				<div class="panel-background" style="background-image: var(--{link.bgKey}-bg);"></div>

				<div class="panel-content">
					<div class="panel-link-header">
						<span class="panel-title-alt">{link.title}</span>
					</div>

					<span class="panel-title">{link.title}</span>
				</div>
			</a>
		{/each}
	</div>
{/if}

<style>
	/* --- Theme Variables --- */

	:root {
		/* Shared Colors */

		--color-projects-tint: #ff00 00;

		--color-about-tint: #008080;

		--color-garden-tint: #0000ff;

		/* Dark Mode */

		--color-bg-dark: #121212;

		--color-text-dark: #f0f0f0;

		--color-panel-projects-dark: #525252;

		--color-panel-about-dark: #242424;

		--color-panel-garden-dark: #1b1b1b;

		/* Light Mode */

		--color-bg-light: #f0f0f0;

		--color-text-light: #121212;

		--color-panel-projects-light: rgb(238, 238, 238);

		--color-panel-about-light: #dfdfdfd0;

		--color-panel-garden-light: #cacacacb;

		/* Background Images */

		--key-projects-bg: url('/frame.png');

		--key-about-bg: url('/frame2.png');

		--key-garden-bg: url('/garden.jpg');
	}

	/* --- Global Reset and Setup --- */

	:global(html),
	:global(body) {
		height: 100%;

		margin: 0;

		padding: 0;

		box-sizing: border-box;

		font-family: 'Playfair Display', serif;

		transition:
			background-color 0.5s ease,
			color 0.5s ease;
	}

	/* Base Colors (Dark Mode Default) */

	:global(body) {
		background-color: var(--color-bg-dark);

		color: var(--color-text-dark);
	}

	/* --- Light Mode Overrides --- */

	.accordion-container[data-theme='light'] {
		background-color: var(--color-bg-light);
	}

	.accordion-container[data-theme='light'] .accordion-panel,
	.loading-screen[data-theme='light'] {
		background-color: var(--color-bg-light);

		color: var(--color-text-light);

		border-color: #ccc;
	}

	/* Panel Base Colors by Theme (Unchanged) */

	.accordion-container[data-theme='dark'] .panel-projects {
		background-color: var(--color-panel-projects-dark);
	}

	.accordion-container[data-theme='dark'] .panel-about-me {
		background-color: var(--color-panel-about-dark);
	}

	.accordion-container[data-theme='dark'] .panel-garden {
		background-color: var(--color-panel-garden-dark);
	}

	.accordion-container[data-theme='light'] .panel-projects {
		background-color: var(--color-panel-projects-light);
	}

	.accordion-container[data-theme='light'] .panel-about-me {
		background-color: var(--color-panel-about-light);
	}

	.accordion-container[data-theme='light'] .panel-garden {
		background-color: var(--color-panel-garden-light);
	}

	/* --- Start Animation Styles --- */

	.loading-screen {
		position: fixed;

		top: 0;

		left: 0;

		width: 100vw;

		height: 100vh;

		z-index: 100;

		display: flex;

		justify-content: center;

		align-items: center;

		overflow: hidden;
	}

	.loading-screen[data-theme='dark'] {
		background-color: var(--color-bg-dark);
	}

	.loading-screen[data-theme='light'] {
		background-color: var(--color-bg-light);
	}

	.initial-name {
		font-size: 6.5vw;

		font-weight: 900;

		letter-spacing: 0.1em;

		text-transform: uppercase;

		position: absolute;

		top: 0;

		transform: translateY(0%);

		animation: name-scroll-fade 2s ease-in-out forwards;

		white-space: nowrap;

		/* FIX: H1 Gradient removal/fix */

		color: var(--color-text-dark); /* Default text color */

		background-image: none; /* Remove background image/gradient */

		background-clip: unset;

		-webkit-background-clip: unset;
	}

	/* Reapply color/gradient only if desired for specific themes, e.g., in dark mode for a metallic look */

	/* .loading-screen[data-theme='dark'] .initial-name {



        color: transparent;



        background-image: linear-gradient(to right, var(--color-text-dark), #aaa);



        background-clip: text;



        -webkit-background-clip: text;



    } */

	/* Light Mode: just use the light text color */

	.loading-screen[data-theme='light'] .initial-name {
		color: var(--color-text-light);

		background-image: none;
	}

	.name-word {
		display: inline-block;

		padding: 0 0.1em;
	}

	@keyframes name-scroll-fade {
		0% {
			transform: translateY(-100vh);

			opacity: 1;
		}

		35% {
			transform: translateY(0);

			opacity: 1;
		}

		85% {
			transform: translateY(0);

			opacity: 1;
		}

		100% {
			transform: translateY(0);

			opacity: 0;
		}
	}

	/* --- Accordion Layout & Expansion --- */

	.accordion-container {
		display: flex;

		width: 100vw;

		height: 100vh;

		overflow: hidden;
	}

	.accordion-panel {
		container-type: inline-size;

		flex-grow: 1;

		height: 100%;

		text-decoration: none;

		overflow: hidden;

		transition: flex-grow 0.6s ease-in-out;

		cursor: pointer;

		position: relative;

		display: flex;

		justify-content: center;

		align-items: center;

		border-right: 2px solid var(--border-color-dark, #333);

		outline: 3px solid transparent;

		outline-offset: -3px;

		transition:
			flex-grow 0.6s ease-in-out,
			outline-color 0.2s ease;
	}

	/* Border Color by Theme (Unchanged) */

	.accordion-container[data-theme='light'] .accordion-panel {
		border-right-color: #ccc;
	}

	.accordion-container[data-theme='dark'] .accordion-panel {
		border-right-color: #333;
	}

	.accordion-container[data-theme='light'] .accordion-panel:last-child {
		border-right: none;
	}

	.accordion-container[data-theme='dark'] .accordion-panel:last-child {
		border-right: none;
	}

	/* Panel Expansion FIX: Less Drastic */

	.accordion-panel:hover,
	.accordion-panel:focus {
		flex-grow: 1.2; /* Reduced from 1.5 to 1.2 */

		outline-color: var(--color-text-dark);
	}

	.accordion-container:hover .accordion-panel,
	.accordion-container:focus-within .accordion-panel {
		flex-grow: 1;
	}

	.accordion-container:hover .accordion-panel:hover,
	.accordion-container:focus-within .accordion-panel:focus {
		flex-grow: 1.2; /* Reduced from 1.5 to 1.2 */
	}

	/* --- Hades Background Reveal Fix --- (Unchanged) */

	.panel-background {
		position: absolute;

		top: 0;

		left: 0;

		width: 100%;

		height: 100%;

		background-size: cover;

		background-position: center;

		opacity: 0;

		transform: scaleX(1.1);

		transition:
			opacity 0.5s ease-out,
			transform 0.6s ease-out;

		z-index: 1;
	}

	/* On Hover/Focus, it becomes visible and scales */

	.accordion-panel:hover .panel-background,
	.accordion-panel:focus .panel-background {
		opacity: 0.3;

		transform: scaleX(1);
	}

	/* --- Content Visibility & Theming --- */

	.panel-content {
		position: absolute;

		top: 0;

		left: 0;

		width: 100%;

		height: 100%;

		display: flex;

		flex-direction: column;

		justify-content: center;

		align-items: center;

		z-index: 2;

		pointer-events: none;
	}

	/* Top Link Header (Unchanged) */

	.panel-link-header {
		position: absolute;

		top: 20px;

		left: 20px;

		z-index: 3;

		padding: 10px 20px;

		border: 2px solid var(--color-text-dark);

		border-radius: 5px;

		text-transform: uppercase;

		font-size: 1.2rem;

		font-weight: bold;

		opacity: 0;

		transition: opacity 0.3s ease;
	}

	.accordion-container[data-theme='dark'] .panel-link-header {
		color: var(--color-text-dark);

		border-color: var(--color-text-dark);
	}

	.accordion-container[data-theme='light'] .panel-link-header {
		color: var(--color-text-light);

		border-color: var(--color-text-light);
	}

	.accordion-panel:hover .panel-link-header,
	.accordion-panel:focus .panel-link-header {
		opacity: 1;
	}

	/* Middle Title Text (The centered box) */

	.panel-title {
		font-size: 3rem;

		font-weight: bold;

		text-transform: uppercase;

		white-space: nowrap;

		padding: 20px;

		border-radius: 10px;

		transition: opacity 0.3s ease;

		max-width: 90%;
	}

	/* Middle Title Color/Background by Theme (Unchanged) */

	.accordion-container[data-theme='dark'] .panel-title {
		color: var(--color-text-dark);

		text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.9);

		background-color: rgba(0, 0, 0, 0.6);
	}

	.accordion-container[data-theme='light'] .panel-title {
		color: var(--color-text-light);

		text-shadow: 2px 2px 5px rgba(255, 255, 255, 0.9);

		background-color: rgba(255, 255, 255, 0.7);
	}

	.accordion-panel:hover .panel-title,
	.accordion-panel:focus .panel-title {
		opacity: 0;
	}

	/* --- Container Query & Mobile Adjustments FIX --- */

	/* Base text size (for compressed accordion panels) */

	.panel-title {
		font-size: 1.5rem; /* Starting point to prevent overflow at 700px-1000px wide viewports */
	}

	/* CQ 1: Medium Text - Panel is wide enough for 2rem text (e.g., when expanded or on wide screens) */

	@container (min-width: 250px) {
		.panel-title {
			font-size: 2rem;
		}
	}

	/* CQ 2: Large Text - Panel is fully expanded or on very large screens */

	@container (min-width: 400px) {
		.panel-title {
			font-size: 3rem;
		}
	}

	@media (max-width: 768px) {
		/* Mobile (Vertical Stack) */

		.accordion-container {
			flex-direction: column;

			height: auto;

			min-height: 100vh;
		}

		.accordion-panel {
			width: 100%;

			height: 33.33vh;

			border-right: none;

			border-bottom: 2px solid var(--border-color-dark, #333);
		}

		/* Ensure font size is constrained on mobile regardless of CQs (Final Size for Mobile) */

		.panel-title {
			font-size: 1.5rem;

			max-width: 80%;
		}

		.panel-link-header {
			font-size: 1rem;

			padding: 8px 15px;

			opacity: 0;
		}

		.accordion-container[data-theme='light'] .accordion-panel {
			border-bottom-color: #ccc;
		}

		.accordion-container[data-theme='dark'] .accordion-panel {
			border-bottom-color: #333;
		}

		.accordion-panel:focus {
			outline-color: transparent;
		}
	}
</style>
