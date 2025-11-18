<script>
	import { quintOut } from 'svelte/easing';
	import { fly } from 'svelte/transition';

	// Props - Standard JavaScript export declarations
	export let title = '';
	export let description = '';
	export let readmeLink = '';
	export let liveLink = '';
	export let imageSrc = '';
	export let mediaContent = [];
	export let index = 0;

	// State for the image details popover
	let detailsOpen = false;
	let selectedMedia = mediaContent.length > 0 ? mediaContent[0] : null;

	function openMedia(media) {
		selectedMedia = media;
	}

	// Determine animation delay based on index for staggered load-in
	const delay = index * 100; // 100ms delay per card
</script>

<div
	class="project-card"
	style="--delay: {delay}ms;"
	in:fly={{ y: 50, duration: 500, delay: delay, easing: quintOut }}
>
	<div class="project-card-inner">
		<div class="project-image-column">
			<h3 class="project-title-mobile">{title}</h3>
			{#if imageSrc}
				<img src={imageSrc} alt="{title} preview" class="project-image" />
			{:else}
				<div class="project-image-placeholder">No Image Available</div>
			{/if}
		</div>

		<div class="project-content-column">
			<h3 class="project-title-desktop">{title}</h3>

			<p class="project-description">
				{description}
			</p>

			<div class="links">
				<a href={readmeLink} target="_blank" class="link-button readme"> View Readme </a>
				<a href={liveLink} target="_blank" class="link-button live"> View Live </a>
				{#if mediaContent.length > 0}
					<button class="link-button media-button" on:click={() => (detailsOpen = !detailsOpen)}>
						{detailsOpen ? 'Hide Media' : 'Show Media'}
					</button>
				{/if}
			</div>
		</div>
	</div>

	{#if mediaContent.length > 0}
		<div class="project-details-popover" class:open={detailsOpen} transition:slide>
			<div class="media-tabs">
				{#each mediaContent as media}
					<button
						class="media-tab"
						class:active={media === selectedMedia}
						on:click={() => openMedia(media)}
					>
						{media.type === 'image' ? 'Image' : 'Video'}
					</button>
				{/each}
			</div>

			<div class="media-viewer">
				{#if selectedMedia && selectedMedia.type === 'image'}
					<img src={selectedMedia.src} alt="Project Media" class="media-display-image" />
				{:else if selectedMedia && selectedMedia.type === 'video'}
					<iframe
						title="Project Video"
						src={selectedMedia.src}
						frameborder="0"
						allow="autoplay; encrypted-media"
						allowfullscreen
						class="media-display-video"
					></iframe>
				{/if}
			</div>
		</div>
	{/if}
</div>

<style>
	/* --- Component Layout --- */
	.project-card {
		width: 100%; /* Full width of its parent */
		margin: 20px 0;
		padding: 0 10px;
		color: #f0f0f0;
		/* Animation: Svelte transition handles initial opacity and delay */
	}

	.project-card-inner {
		display: flex;
		background-color: #1a1a1a;
		border-radius: 8px;
		overflow: hidden;
		box-shadow: 0 5px 15px rgba(0, 0, 0, 0.4);
		max-width: 1200px;
		margin: 0 auto; /* Center the card */
	}

	.project-image-column {
		flex: 1;
		min-width: 250px;
		position: relative;
		background-color: #0d0d0d;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
	}

	.project-image {
		width: 100%;
		height: auto;
		object-fit: cover;
	}

	.project-image-placeholder {
		padding: 20px;
		text-align: center;
		opacity: 0.5;
	}

	.project-content-column {
		flex: 2; /* Takes 2/3 of the space */
		padding: 30px;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
	}

	.project-title-desktop {
		font-size: 2.2em;
		margin-top: 0;
		margin-bottom: 20px;
		text-align: left;
	}

	.project-title-mobile {
		display: none; /* Hide on desktop */
	}

	.project-description {
		font-size: 1.4em;
		line-height: 1.6;
		color: #bbb;
		flex-grow: 1;
		margin-bottom: 20px;

		padding-right: 0;
		text-align: left; /* FIX: Align description text to the left */
	}

	.links {
		display: flex;
		gap: 15px;
		margin-top: 15px;
	}

	.link-button {
		padding: 10px 20px;
		border-radius: 6px;
		text-decoration: none;
		font-weight: bold;
		transition:
			background-color 0.2s,
			transform 0.1s;
		text-align: center;
		cursor: pointer;
	}

	.readme {
		background-color: #555;
		color: white;
	}
	.live {
		background-color: #007bff;
		color: white;
	}
	.media-button {
		background-color: #8800ff;
		color: white;
	}

	.link-button:hover {
		transform: translateY(-2px);
	}

	/* --- Details Popover Section --- */
	.project-details-popover {
		max-width: 1200px;
		margin: 10px auto 20px auto;
		background-color: #000000;
		border-radius: 8px;
		padding: 20px;
		box-shadow: 0 5px 15px rgba(0, 0, 0, 0.5);
		display: none; /* Svelte transition will manage visibility */
		flex-direction: column;
		gap: 15px;
	}

	.project-details-popover.open {
		display: flex; /* Show when 'open' class is applied */
	}

	.media-tabs {
		display: flex;
		gap: 10px;
		border-bottom: 1px solid #333;
		padding-bottom: 10px;
	}

	.media-tab {
		background: none;
		border: none;
		color: #bbb;
		padding: 8px 15px;
		cursor: pointer;
		border-radius: 4px;
		transition:
			color 0.2s,
			background-color 0.2s;
	}

	.media-tab:hover,
	.media-tab.active {
		color: white;
		background-color: #2a2a2a;
	}

	.media-viewer {
		width: 100%;
		aspect-ratio: 16 / 9; /* Standard video ratio */
		background-color: #111;
		border-radius: 4px;
		overflow: hidden;
	}

	.media-display-image,
	.media-display-video {
		width: 100%;
		height: 100%;
		object-fit: contain; /* Ensures content fits without cropping */
	}

	/* --- Mobile Adaptation --- */
	@media (max-width: 768px) {
		.project-card-inner {
			flex-direction: column; /* Stacks vertically */
		}

		.project-content-column {
			order: 2;
			padding: 30px;
			display: flex;
			flex-direction: column;
			justify-content: space-between;
		}

		.project-title-desktop {
			display: none; /* Hide desktop title */
		}

		.project-title-mobile {
			display: block; /* Show mobile title */
			font-size: 1.8em;
			padding: 15px;
			text-align: center;
		}

		.links {
			flex-direction: column;
			gap: 10px;
		}
	}
</style>
