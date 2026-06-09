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
		z-index: 50;
		padding: 14px 20px;
		transition: background-color 0.3s ease, backdrop-filter 0.3s ease;
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

	/* Canonical dark-glass nav button — readable on dark pages AND bright gradients.
	   The same style is mirrored by .next-button (digital-garden) and .nav-btn
	   (liquid-gradient) so every page's nav controls look identical. */
	.home-button {
		background: rgba(0, 0, 0, 0.5);
		border: 1px solid rgba(255, 255, 255, 0.18);
		color: rgba(255, 255, 255, 0.9);
		padding: 8px 16px;
		text-decoration: none;
		border-radius: 8px;
		font-size: 0.85rem;
		font-weight: 500;
		backdrop-filter: blur(8px);
		-webkit-backdrop-filter: blur(8px);
		transition: background 0.2s, border-color 0.2s, color 0.2s;
	}

	.home-button:hover {
		background: rgba(0, 0, 0, 0.7);
		border-color: rgba(255, 255, 255, 0.35);
		color: #fff;
	}
</style>
