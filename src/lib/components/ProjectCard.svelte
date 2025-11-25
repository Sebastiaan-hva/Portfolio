<script>
	/**
	 * PROJECT CARD - SVELTE 5
	 * Features:
	 * 1. Native Popover API for details
	 * 2. Scroll-Driven Animations (CSS view() timeline)
	 */

	let {
		title = '',
		shortDescription = '',
		description = '',
		readmeLink = '',
		liveLink = '',
		imageSrc = '',
		mediaContent = [],
		index = 0
	} = $props();

	// Create a unique ID for the popover connection
	const popoverId = `popover-${index}`;
</script>

<article class="project-card">
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

				{#if readmeLink && readmeLink !== '#'}
					<a href={readmeLink} target="_blank" class="link-button readme">GitHub</a>
				{/if}

				{#if liveLink && liveLink !== '#'}
					<a href={liveLink} target="_blank" class="link-button live">Live Demo</a>
				{/if}
			</div>
		</div>
	</div>

	<div id={popoverId} popover class="native-popover">
		<div class="popover-content">
			<div class="popover-header">
				<h3>{title}</h3>
				<button class="close-btn" popovertarget={popoverId} popovertargetaction="hide"> × </button>
			</div>

			<div class="popover-body">
				<p class="long-description">{description}</p>

				{#if mediaContent.length > 0}
					<div class="media-preview">
						<p>Media items: {mediaContent.length}</p>
					</div>
				{/if}
			</div>
		</div>
	</div>
</article>

<style>
	/* --- SCROLL DRIVEN ANIMATION --- */
	@keyframes appear {
		from {
			opacity: 0;
			transform: translateY(100px) scale(0.9);
			filter: blur(10px);
		}
		to {
			opacity: 1;
			transform: translateY(0) scale(1);
			filter: blur(0);
		}
	}

	.project-card {
		width: 100%;
		margin: 40px 0; /* Add spacing so we have room to scroll */

		/* The Animation Magic */
		animation: appear linear both;

		/* view() = Track this element relative to the viewport.
		   entry 10% = Start animation when top of card enters 10% of viewport.
		   cover 30% = Finish animation when card has covered 30% of viewport.
		*/
		animation-timeline: view();
		animation-range: entry 10% cover 30%;
	}

	/* --- CARD LAYOUT --- */
	.project-card-inner {
		display: flex;
		background-color: #1a1a1a;
		border-radius: 12px;
		overflow: hidden;
		box-shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
		border: 1px solid #333;
		transition:
			transform 0.3s ease,
			box-shadow 0.3s ease;
	}

	.project-card-inner:hover {
		transform: translateY(-5px);
		box-shadow: 0 10px 20px rgba(0, 0, 0, 0.5);
		border-color: #555;
	}

	.project-image-column {
		flex: 1;
		min-width: 250px;
		background: #000;
		position: relative;
	}

	.project-image {
		width: 100%;
		height: 300px;
		object-fit: cover;
		display: block;
	}

	.project-image-placeholder {
		padding: 40px;
		text-align: center;
		color: #555;
	}

	.project-content-column {
		flex: 2;
		padding: 30px;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		text-align: left;
	}

	.project-title-desktop {
		margin-top: 0;
		color: #fff;
		font-size: 1.8rem;
		margin-bottom: 0.5rem;
	}

	.project-short-description {
		color: #bbb;
		margin-bottom: 25px;
		line-height: 1.6;
		font-size: 1.1rem;
	}

	.links {
		display: flex;
		gap: 12px;
		flex-wrap: wrap;
	}

	.link-button {
		padding: 10px 20px;
		border-radius: 6px;
		text-decoration: none;
		font-weight: 600;
		cursor: pointer;
		border: none;
		font-size: 0.95em;
		transition: all 0.2s;
	}

	.link-button:hover {
		filter: brightness(1.2);
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

	/* --- POPOVER STYLES --- */
	[popover] {
		margin: auto;
		padding: 0;
		border: none;
		border-radius: 12px;
		background: #1a1a1a;
		color: #f0f0f0;
		box-shadow: 0 20px 60px rgba(0, 0, 0, 0.8);
		width: 90%;
		max-width: 650px;
		border: 1px solid #444;

		/* 1. THE EXIT STATE (When closing) */
		opacity: 0;
		transform: scale(0.95); /* Shrink slightly in place, don't zoom to 0 */

		/* 2. THE EXIT ANIMATION */
		/* 'allow-discrete' is CRITICAL. It keeps the element in the DOM 
		   long enough for the fade-out to finish before setting display:none */
		transition:
			opacity 0.3s ease-in,
			transform 0.3s ease-in,
			display 0.3s ease-in allow-discrete,
			overlay 0.3s ease-in allow-discrete;
	}

	/* 3. THE OPEN STATE (Active) */
	[popover]:popover-open {
		opacity: 1;
		transform: scale(1);
	}

	/* 4. THE ENTRY ANIMATION (Starting Point) */
	@starting-style {
		[popover]:popover-open {
			opacity: 0;
			transform: scale(0.95); /* Start slightly smaller */
		}
	}

	/* --- BACKDROP STYLES --- */
	[popover]::backdrop {
		background-color: rgba(0, 0, 0, 0.6);
		backdrop-filter: blur(4px);

		/* Exit State */
		opacity: 0;

		/* Animate the backdrop exit too */
		transition:
			opacity 0.3s ease-in,
			display 0.3s ease-in allow-discrete,
			overlay 0.3s ease-in allow-discrete;
	}

	[popover]:popover-open::backdrop {
		opacity: 1;
	}

	@starting-style {
		[popover]:popover-open::backdrop {
			opacity: 0;
		}
	}

	/* ... rest of your popover content styles ... */

	[popover]::backdrop {
		background-color: rgba(0, 0, 0, 0.6);
		backdrop-filter: blur(4px);
		opacity: 0;
		transition: opacity 0.3s allow-discrete;
	}

	[popover]:popover-open::backdrop {
		opacity: 1;
	}

	.popover-content {
		padding: 30px;
	}

	.popover-header {
		display: flex;
		justify-content: space-between;
		align-items: flex-start;
		margin-bottom: 20px;
		border-bottom: 1px solid #333;
		padding-bottom: 15px;
	}

	.popover-header h3 {
		margin: 0;
		font-size: 1.8em;
		color: white;
	}

	.close-btn {
		background: none;
		border: none;
		color: #888;
		font-size: 2rem;
		line-height: 0.5;
		cursor: pointer;
		padding: 5px;
	}
	.close-btn:hover {
		color: #fff;
	}

	.long-description {
		white-space: pre-line;
		line-height: 1.8;
		color: #ddd;
	}

	/* --- MOBILE STYLES --- */
	.project-title-mobile {
		display: none;
	}

	@media (max-width: 768px) {
		.project-card-inner {
			flex-direction: column;
		}
		.project-image-column {
			height: 200px;
		}
		.project-title-desktop {
			display: none;
		}
		.project-title-mobile {
			display: block;
			color: white;
			text-align: center;
			padding: 15px;
			margin: 0;
			background: #111;
		}
	}
</style>
