<script>
	import { onMount } from 'svelte';

	// --- State Management ---
	let isLoading = true;
	const fullName = 'Sebastiaan Hagoort';
	const nameParts = fullName.split(' ');

	// --- Theme Logic ---
	let theme = 'dark';

	function getSystemTheme() {
		if (typeof window !== 'undefined') {
			return window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light';
		}
		return 'dark';
	}

	onMount(() => {
		theme = getSystemTheme();
		if (typeof window !== 'undefined') {
			const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
			const handler = () => {
				theme = getSystemTheme();
			};
			mediaQuery.addEventListener('change', handler);
			return () => mediaQuery.removeEventListener('change', handler);
		}
	});

	// --- Start Animation Logic ---
	// Only show the intro animation the first time per browser session.
	onMount(() => {
		if (sessionStorage.getItem('introSeen')) {
			isLoading = false;
			return;
		}
		setTimeout(() => {
			isLoading = false;
			sessionStorage.setItem('introSeen', '1');
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

<main class="landing-wrapper" data-theme={theme}>
	{#if isLoading}
		<div class="loading-screen">
			<h1 class="initial-name">
				{#each nameParts as word}
					<span class="name-word">{word}</span>
				{/each}
			</h1>
		</div>
	{:else}
		<div class="accordion-container">
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
</main>

<style>
	/* 2. SCOPED VARIABLES 
       Instead of :root, we define these on the wrapper class.
       These only exist when this page is mounted.
    */
	.landing-wrapper {
		/* Shared Colors */
		--color-projects-tint: #ff0000;
		--color-about-tint: #008080;
		--color-garden-tint: #0000ff;

		/* Dark Mode Defaults */
		--color-bg-dark: #121212;
		--color-text-dark: #f0f0f0;
		--color-panel-projects-dark: #3f3f3f;
		--color-panel-about-dark: #242424;
		--color-panel-garden-dark: #1b1b1b;
		--border-color-dark: #333;

		/* Light Mode Defaults */
		--color-bg-light: #f0f0f0;
		--color-text-light: #121212;
		--color-panel-projects-light: rgb(88, 88, 88);
		--color-panel-about-light: #3b3b3bd0;
		--color-panel-garden-light: rgba(37, 37, 37, 0.796);

		/* Background Images */
		--key-projects-bg: url('/frame.png');
		--key-about-bg: url('/frame2.png');
		--key-garden-bg: url('/garden.jpg');

		/* 3. SIMULATE BODY STYLES 
           We move the styles that were on :global(body) to here.
        */
		height: 100vh;
		width: 100vw;
		margin: 0;
		padding: 0;
		box-sizing: border-box;
		font-family: 'Playfair Display', serif;
		overflow: hidden; /* Critical for your accordion layout */

		/* Default to Dark Mode styles */
		background-color: var(--color-bg-dark);
		color: var(--color-text-dark);

		transition:
			background-color 0.5s ease,
			color 0.5s ease;
	}

	/* Light Mode Override on the Wrapper */
	.landing-wrapper[data-theme='light'] {
		background-color: var(--color-bg-light);
		color: var(--color-text-light);
	}

	/* --- Accordion Container --- */
	.accordion-container {
		display: flex;
		width: 100%;
		height: 100%;
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
		border-right: 2px solid;
		outline: 3px solid transparent;
		outline-offset: -3px;
		transition:
			flex-grow 0.6s ease-in-out,
			outline-color 0.2s ease;
	}

	/* Use parent data-theme to style children */
	.landing-wrapper[data-theme='light'] .accordion-panel {
		border-right-color: #ccc;
		background-color: var(--color-bg-light);
		color: var(--color-text-light);
	}

	.landing-wrapper[data-theme='dark'] .accordion-panel {
		border-right-color: #333;
	}

	.accordion-panel:last-child {
		border-right: none !important;
	}

	/* Panel Colors */
	.landing-wrapper[data-theme='dark'] .panel-projects {
		background-color: var(--color-panel-projects-dark);
	}
	.landing-wrapper[data-theme='dark'] .panel-about-me {
		background-color: var(--color-panel-about-dark);
	}
	.landing-wrapper[data-theme='dark'] .panel-garden {
		background-color: var(--color-panel-garden-dark);
	}

	.landing-wrapper[data-theme='light'] .panel-projects {
		background-color: var(--color-panel-projects-light);
	}
	.landing-wrapper[data-theme='light'] .panel-about-me {
		background-color: var(--color-panel-about-light);
	}
	.landing-wrapper[data-theme='light'] .panel-garden {
		background-color: var(--color-panel-garden-light);
	}

	/* Hover Effects */
	.accordion-panel:hover,
	.accordion-panel:focus {
		flex-grow: 1.2;
		outline-color: currentColor;
	}

	.accordion-container:hover .accordion-panel,
	.accordion-container:focus-within .accordion-panel {
		flex-grow: 1;
	}

	.accordion-container:hover .accordion-panel:hover,
	.accordion-container:focus-within .accordion-panel:focus {
		flex-grow: 1.2;
	}

	/* --- Background Image Reveal --- */
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

	.accordion-panel:hover .panel-background,
	.accordion-panel:focus .panel-background {
		opacity: 0.3;
		transform: scaleX(1);
	}

	/* --- Content --- */
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

	.panel-link-header {
		position: absolute;
		top: 20px;
		left: 20px;
		z-index: 3;
		padding: 10px 20px;
		border: 2px solid currentColor;
		border-radius: 5px;
		text-transform: uppercase;
		font-size: 1.2rem;
		font-weight: bold;
		opacity: 0;
		transition: opacity 0.3s ease;
	}

	.accordion-panel:hover .panel-link-header,
	.accordion-panel:focus .panel-link-header {
		opacity: 1;
	}

	.panel-title {
		font-size: 1.5rem;
		font-weight: bold;
		text-transform: uppercase;
		white-space: nowrap;
		padding: 20px;
		border-radius: 10px;
		transition: opacity 0.3s ease;
		max-width: 90%;
	}

	.landing-wrapper[data-theme='dark'] .panel-title {
		color: var(--color-text-dark);
		text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.9);
		background-color: rgba(0, 0, 0, 0.6);
	}

	.landing-wrapper[data-theme='light'] .panel-title {
		color: var(--color-text-light);
		text-shadow: 2px 2px 5px rgba(255, 255, 255, 0.9);
		background-color: rgba(255, 255, 255, 0.7);
	}

	.accordion-panel:hover .panel-title,
	.accordion-panel:focus .panel-title {
		opacity: 0;
	}

	/* --- Loading Screen --- */
	.loading-screen {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		z-index: 100;
		display: flex;
		justify-content: center;
		align-items: center;
		overflow: hidden;
		/* Inherit background from wrapper so it matches theme */
		background-color: inherit;
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
		color: inherit; /* Inherit form wrapper */
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

	/* --- Animations (Hail, Ripple, Sun) --- */
	/* (These animations are mostly standard and don't cause global leaks) */

	.panel-projects::before,
	.panel-projects::after,
	.panel-about-me::before,
	.panel-about-me::after,
	.panel-garden::before {
		content: '';
		position: absolute;
		top: -50%;
		left: -50%;
		width: 200%;
		height: 200%;
		pointer-events: none;
		z-index: 1;
		transition: opacity 0.3s ease;
	}

	.accordion-panel:hover::before,
	.accordion-panel:focus::before,
	.accordion-panel:hover::after,
	.accordion-panel:focus::after {
		opacity: 0 !important;
		animation: none !important;
	}

	/* Hail Fall */
	.panel-projects::before {
		background-image: radial-gradient(circle, rgba(253, 253, 253, 0.4) 1.5px, transparent 2.5px);
		background-size: 60px 60px;
		transform: rotate(20deg) scaleY(6);
		animation: hail-fall-1 1.5s linear infinite;
	}
	.panel-projects::after {
		background-image: radial-gradient(circle, rgba(165, 165, 165, 0.4) 1px, transparent 2px);
		background-size: 100px 100px;
		background-position: 50px 0;
		transform: rotate(25deg) scaleY(5);
		animation: hail-fall-2 1.5s linear infinite;
	}
	@keyframes hail-fall-1 {
		0% {
			background-position: 0 0;
		}
		100% {
			background-position: 0 60px;
		}
	}
	@keyframes hail-fall-2 {
		0% {
			background-position: 50px 0;
		}
		100% {
			background-position: 50px 100px;
		}
	}

	/* Ripples */
	.panel-about-me::before,
	.panel-about-me::after {
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		transform: none;
		background-image: radial-gradient(
			circle,
			transparent 20%,
			rgba(255, 255, 255, 0.2) 30%,
			transparent 40%
		);
		background-position: center;
		background-repeat: no-repeat;
		opacity: 0;
	}
	.panel-about-me::before {
		animation: ripple-loop 4s linear infinite;
	}
	.panel-about-me::after {
		animation: ripple-loop 4s linear infinite 2s;
	}
	@keyframes ripple-loop {
		0% {
			background-size: 0% 0%;
			opacity: 0;
		}
		15% {
			opacity: 1;
		}
		100% {
			background-size: 150% 150%;
			opacity: 0;
		}
	}

	/* Sunset */
	.panel-garden::before {
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		transform: none;
		background-image: radial-gradient(circle, rgba(255, 255, 255, 0.95) 0%, transparent 65%);
		background-repeat: no-repeat;
		background-size: 100% 60%;
		animation: sun-setting 15s linear infinite;
	}
	@keyframes sun-setting {
		0% {
			background-position: 50% -60%;
			opacity: 0.8;
		}
		10% {
			opacity: 1;
		}
		90% {
			opacity: 1;
		}
		100% {
			background-position: 50% 160%;
			opacity: 0.8;
		}
	}

	/* --- Responsive --- */
	@container (min-width: 350px) {
		.panel-title {
			font-size: 2rem;
		}
	}
	@container (min-width: 450px) {
		.panel-title {
			font-size: 3rem;
		}
	}

	@media (max-width: 768px) {
		.accordion-container {
			flex-direction: column;
		}
		.accordion-panel {
			container-type: inline-size;
			width: 100%;
			height: 33.33vh;
			border-right: none;
			border-bottom: 2px solid;
		}
		.landing-wrapper[data-theme='light'] .accordion-panel {
			border-bottom-color: #ccc;
		}
		.landing-wrapper[data-theme='dark'] .accordion-panel {
			border-bottom-color: #333;
		}

		.panel-title {
			font-size: 1.5rem;
			max-width: 80%;
		}
		.panel-link-header {
			font-size: 1rem;
			padding: 8px 15px;
			opacity: 0;
		}
		.accordion-panel:focus {
			outline-color: transparent;
		}
	}
</style>
