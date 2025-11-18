<script>
	import StickyNav from '$lib/components/Nav.svelte';
	import { page } from '$app/stores';

	// 💡 Svelte 5 uses $props() for component props.
	// In SvelteKit's root layout, the content is usually passed via <slot/>,
	// but if you are using the new Svelte 5 runestone structure for *all* components,
	// you typically define props (though not strictly necessary for the root layout slot).
	// The main content is accessed via a named slot (e.g., 'children') in the props.

	// Assuming 'children' is the default slot content passed down:
	const { children } = $props();

	// Note: The $page store import is correct.
</script>

{#if $page.url.pathname !== '/'}
	<StickyNav />
{/if}

<svelte:head></svelte:head>

<main>
	{@render children()}
</main>
