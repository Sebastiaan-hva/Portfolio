<script>
	import { onMount } from 'svelte';
	import { slide } from 'svelte/transition';
	import { page } from '$app/stores'; // SvelteKit store for current page

	let lastScrollY = 0;
	let isVisible = true;
	let isScrolled = false;

	function handleScroll() {
		if (typeof window === 'undefined') return;

		const currentScrollY = window.scrollY;

		// Determine if scrolling up or down
		if (currentScrollY > lastScrollY && currentScrollY > 100) {
			// Scrolling down and past a small threshold
			isVisible = false;
		} else {
			// Scrolling up or near the top
			isVisible = true;
		}

		// Determine if scrolled away from top (for blur effect)
		isScrolled = currentScrollY > 10;

		lastScrollY = currentScrollY;
	}

	onMount(() => {
		window.addEventListener('scroll', handleScroll);

		return () => {
			window.removeEventListener('scroll', handleScroll);
		};
	});
</script>

{#if $page.url.pathname !== '/'}
	<div class="sticky-nav-wrapper" class:scrolled={isScrolled} transition:slide|local>
		<nav class="nav-content">
			<a href="/" class="home-button"> Home </a>
		</nav>
	</div>
{/if}

<style>
	.sticky-nav-wrapper {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		z-index: 50; /* Above project cards, below loading screen */
		padding: 15px 20px;
		transition: background-color 0.3s ease;
	}

	/* Subtle blur effect when scrolled down */
	.sticky-nav-wrapper.scrolled {
		background-color: rgba(18, 18, 18, 0.5); /* Semi-transparent dark background */
		backdrop-filter: blur(5px);
	}

	.nav-content {
		max-width: 1200px;
		margin: 0;
		display: flex;
		justify-content: flex-start; /* Align button to the left */
	}

	.home-button {
		background: none;
		border: 2px solid #f0f0f0;
		color: #f0f0f0;
		padding: 8px 15px;
		text-decoration: none;
		border-radius: 5px;
		font-weight: bold;
		transition:
			background-color 0.2s,
			color 0.2s;
	}

	.home-button:hover {
		background-color: #f0f0f0;
		color: #121212;
	}
</style>
