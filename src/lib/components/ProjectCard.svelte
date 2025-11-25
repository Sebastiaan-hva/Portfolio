<script>
	import { onMount } from 'svelte';

	// Svelte 5: Props using $props() rune
	let {
		title = '',
		shortDescription = '',
		description = '', // The long description for the popover
		readmeLink = '',
		liveLink = '',
		imageSrc = '',
		mediaContent = [],
		index = 0
	} = $props();

	// Unique ID for this card's popover
	const popoverId = `popover-${index}`;

	// JS Fallback for staggered loading
	// If CSS animations fail or you prefer JS control, this fades them in.
	let isVisible = $state(false);

	onMount(() => {
		// Arrow notation fallback as requested
		const staggerFadeIn = () => {
			setTimeout(() => {
				isVisible = true;
			}, index * 75); // 150ms delay per card
		};

		staggerFadeIn();
	});
</script>

<article class="project-card" class:visible={isVisible} style="--index: {index};">
	<div class="project-card-inner">
		<div class="project-image-column">
			<h3 class="project-title-mobile">{title}</h3>
			{#if imageSrc}
				<img src={imageSrc} alt="{title} preview" class="project-image" />
			{:else}
				<div class="project-image-placeholder">No Image</div>
			{/if}
		</div>

		<div class="project-content-column">
			<h3 class="project-title-desktop">{title}</h3>

			<p class="project-short-description">
				{shortDescription}
			</p>

			<div class="links">
				<button class="link-button details-btn" popovertarget={popoverId}> Read More </button>

				<a href={readmeLink} target="_blank" class="link-button readme">GitHub</a>
				<a href={liveLink} target="_blank" class="link-button live">Live Demo</a>
			</div>
		</div>
	</div>

	<div id={popoverId} popover class="native-popover">
		<div class="popover-content">
			<div class="popover-header">
				<h3>{title}</h3>
				<button class="close-btn" popovertarget={popoverId} popovertargetaction="hide">×</button>
			</div>

			<div class="popover-body">
				<p class="long-description">{description}</p>

				{#if mediaContent.length > 0}
					<div class="media-preview">
						<p>Media content available ({mediaContent.length} items)</p>
					</div>
				{/if}
			</div>
		</div>
	</div>
</article>

<style>
	/* --- STAGGERED ANIMATION --- */
	.project-card {
		width: 100%;
		opacity: 0; /* Hidden by default */
		transform: translateY(20px);

		/* 1. Define the animation 
           2. Use calc() with the var(--index) passed from Svelte for the delay
        */
		animation: fadeInUp 0.6s ease-out forwards;
		animation-delay: calc(var(--index) * 150ms);
	}

	/* Fallback: If animation fails or is overridden, JS class handles it */
	.project-card.visible {
		opacity: 1;
		transform: translateY(0);
		transition:
			opacity 0.6s ease-out,
			transform 0.6s ease-out;
	}

	@keyframes fadeInUp {
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	/* --- POPOVER STYLES --- */

	/* The actual popover container */
	[popover] {
		margin: auto; /* Centers it in the viewport */
		padding: 0;
		border: none;
		border-radius: 12px;
		background: #1a1a1a;
		color: #f0f0f0;
		box-shadow: 0 10px 40px rgba(0, 0, 0, 0.5);
		width: 90%;
		max-width: 600px;
		opacity: 0;
		transform: scale(0.9);
		transition:
			opacity 0.3s allow-discrete,
			transform 0.3s allow-discrete,
			display 0.3s allow-discrete;
	}

	/* Open State */
	[popover]:popover-open {
		opacity: 1;
		transform: scale(1);
	}

	/* The Backdrop (dimmed background) */
	[popover]::backdrop {
		background-color: rgba(0, 0, 0, 0.7);
		backdrop-filter: blur(4px);
		opacity: 0;
		transition:
			opacity 0.3s allow-discrete,
			display 0.3s allow-discrete;
	}

	[popover]:popover-open::backdrop {
		opacity: 1;
	}

	/* Starting Style for Entry Animation (Newer CSS feature, enhances popover animation) */
	@starting-style {
		[popover]:popover-open {
			opacity: 0;
			transform: scale(0.9);
		}
		[popover]:popover-open::backdrop {
			opacity: 0;
		}
	}

	.popover-content {
		padding: 30px;
		text-align: left;
	}

	.popover-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 20px;
		border-bottom: 1px solid #333;
		padding-bottom: 15px;
	}

	.popover-header h3 {
		margin: 0;
		font-size: 1.8em;
		color: #fff;
	}

	.close-btn {
		background: none;
		border: none;
		color: #888;
		font-size: 2em;
		cursor: pointer;
		line-height: 0.5;
		padding: 10px;
	}

	.close-btn:hover {
		color: #fff;
	}

	.long-description {
		font-size: 1.1em;
		line-height: 1.6;
		color: #ddd;
		white-space: pre-line; /* Preserves line breaks in description */
	}

	/* --- CARD LAYOUT (Simpler version) --- */
	.project-card-inner {
		display: flex;
		background-color: #1a1a1a;
		border-radius: 8px;
		overflow: hidden;
		box-shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
	}

	.project-image-column {
		flex: 1;
		min-width: 200px;
		background: #000;
	}

	.project-image {
		width: 100%;
		max-height: 300px;
		object-fit: cover;
	}

	.project-content-column {
		flex: 2;
		padding: 25px;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		text-align: left;
	}

	.project-title-desktop {
		margin-top: 0;
		color: #fff;
	}

	.project-short-description {
		color: #bbb;
		margin-bottom: 20px;
	}

	.links {
		display: flex;
		gap: 10px;
		flex-wrap: wrap;
	}

	.link-button {
		padding: 8px 16px;
		border-radius: 4px;
		text-decoration: none;
		font-weight: bold;
		cursor: pointer;
		border: none;
		font-size: 0.9em;
		transition: transform 0.2s;
	}

	.link-button:hover {
		transform: translateY(-2px);
	}

	.details-btn {
		background-color: #8800ff;
		color: white;
	}
	.readme {
		background-color: #333;
		color: white;
	}
	.live {
		background-color: #007bff;
		color: white;
	}

	@media (max-width: 768px) {
		.project-card-inner {
			flex-direction: column;
		}
		.project-image-column {
			min-height: 200px;
		}
		.project-title-desktop {
			display: none;
		}
		.project-title-mobile {
			display: block;
			padding: 15px;
			margin: 0;
			color: white;
			text-align: center;
		}
	}
	@media (min-width: 769px) {
		.project-title-mobile {
			display: none;
		}
	}
</style>
