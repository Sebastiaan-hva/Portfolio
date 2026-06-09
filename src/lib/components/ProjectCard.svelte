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
		featured = false,
		compact = false,
		tag = '',
		index = 0,
		imageContain = false
	} = $props();

	// Create a unique ID for the popover connection
	const popoverId = `popover-${index}`;
</script>

<article class="project-card" class:featured class:compact>
	{#if featured}
		<div class="featured-badge">Featured Project</div>
	{/if}
	{#if tag && !featured}
		<div class="tag-label">{tag}</div>
	{/if}
	<div class="project-card-inner">
		<div class="project-image-column">
			<h3 class="project-title-mobile">{title}</h3>
			{#if imageSrc}
				<img
					src={imageSrc}
					alt="{title} preview"
					class="project-image"
					class:image-contain={imageContain}
				/>
			{:else}
				<div class="project-image-placeholder">
					<span class="placeholder-label">{title[0]}</span>
				</div>
			{/if}
		</div>

		<div class="project-content-column">
			<h3 class="project-title-desktop">{title}</h3>

			<p class="project-short-description">
				{shortDescription}
			</p>

			<div class="links">
				<button class="link-button details-btn" popovertarget={popoverId}>
					Details
					<svg class="btn-icon" viewBox="0 0 16 16" fill="none" xmlns="http://www.w3.org/2000/svg">
						<path
							d="M6 12l4-4-4-4"
							stroke="currentColor"
							stroke-width="1.5"
							stroke-linecap="round"
							stroke-linejoin="round"
						/>
					</svg>
				</button>

				{#if readmeLink && readmeLink !== '#'}
					<a href={readmeLink} target="_blank" class="link-button readme">
						<svg
							class="btn-icon-left"
							viewBox="0 0 16 16"
							fill="currentColor"
							xmlns="http://www.w3.org/2000/svg"
						>
							<path
								d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"
							/>
						</svg>
						GitHub
					</a>
				{/if}

				{#if liveLink && liveLink !== '#'}
					<a href={liveLink} target="_blank" class="link-button live">
						Live
						<svg
							class="btn-icon"
							viewBox="0 0 16 16"
							fill="none"
							xmlns="http://www.w3.org/2000/svg"
						>
							<path
								d="M3 13L13 3M13 3H7M13 3v6"
								stroke="currentColor"
								stroke-width="1.5"
								stroke-linecap="round"
								stroke-linejoin="round"
							/>
						</svg>
					</a>
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
		margin: 40px 0;
		position: relative;
		animation: appear linear both;
		animation-timeline: view();
		animation-range: entry 10% cover 30%;
	}

	/* --- TAG LABEL (non-featured cards) --- */
	.tag-label {
		display: inline-block;
		margin-bottom: 8px;
		font-family: 'Helvetica Neue', sans-serif;
		font-size: 0.7rem;
		letter-spacing: 0.1em;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.25);
	}

	/* --- FEATURED BADGE --- */
	.featured-badge {
		display: inline-block;
		margin-bottom: 12px;
		padding: 4px 14px;
		border-radius: 100px;
		background: transparent;
		border: 1px solid rgba(255, 255, 255, 0.2);
		color: #aaa;
		font-size: 0.75rem;
		font-weight: 600;
		letter-spacing: 0.08em;
		text-transform: uppercase;
	}

	/* --- CARD LAYOUT --- */
	.project-card-inner {
		display: flex;
		background-color: #1a1a1a;
		border-radius: 14px;
		overflow: hidden;
		box-shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
		border: 1px solid #333;
		transition:
			transform 0.3s ease,
			box-shadow 0.3s ease,
			border-color 0.3s ease;
	}

	.project-card-inner:hover {
		transform: translateY(-4px);
		box-shadow: 0 16px 40px rgba(0, 0, 0, 0.5);
		border-color: #555;
	}

	/* Featured card gets a slightly brighter border */
	.featured .project-card-inner {
		border-color: rgba(255, 255, 255, 0.15);
	}

	.featured .project-card-inner:hover {
		border-color: rgba(255, 255, 255, 0.35);
	}

	.project-image-column {
		flex: 1;
		min-width: 250px;
		background: #111;
		position: relative;
	}

	.project-image {
		width: 100%;
		height: 100%;
		min-height: 280px;
		object-fit: cover;
		display: block;
	}

	/* Contain mode — shows full image with dark letterbox (logos, tall screenshots) */
	.project-image.image-contain {
		object-fit: contain;
		background: #0d0d0d;
		padding: 20px;
		box-sizing: border-box;
	}

	/* Featured image area is taller */
	.featured .project-image {
		min-height: 380px;
	}

	/* Featured logo gets extra breathing room */
	.featured .project-image.image-contain {
		padding: 48px 40px;
	}

	.project-image-placeholder {
		width: 100%;
		height: 100%;
		min-height: 280px;
		display: flex;
		align-items: center;
		justify-content: center;
		background: linear-gradient(135deg, #1a1a1a, #2a2a2a);
	}

	.featured .project-image-placeholder {
		min-height: 380px;
	}

	.placeholder-label {
		font-size: 5rem;
		font-weight: 800;
		color: rgba(255, 255, 255, 0.06);
		font-family: 'Playfair Display', serif;
		user-select: none;
	}

	.project-content-column {
		flex: 2;
		padding: 32px;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		text-align: left;
	}

	.featured .project-content-column {
		padding: 40px;
	}

	.project-title-desktop {
		margin-top: 0;
		color: #fff;
		font-size: 1.8rem;
		margin-bottom: 0.5rem;
		line-height: 1.2;
	}

	.featured .project-title-desktop {
		font-size: 2.4rem;
	}

	.project-short-description {
		color: #aaa;
		margin-bottom: 28px;
		line-height: 1.65;
		font-size: 1.05rem;
		flex-grow: 1;
	}

	.featured .project-short-description {
		font-size: 1.15rem;
		color: #bbb;
	}

	/* --- BUTTONS --- */
	.links {
		display: flex;
		gap: 10px;
		flex-wrap: wrap;
		align-items: center;
	}

	.link-button {
		display: inline-flex;
		align-items: center;
		gap: 6px;
		padding: 9px 18px;
		border-radius: 100px;
		text-decoration: none;
		font-weight: 600;
		cursor: pointer;
		font-size: 0.875rem;
		letter-spacing: 0.01em;
		transition:
			background 0.2s ease,
			color 0.2s ease,
			transform 0.15s ease,
			box-shadow 0.2s ease,
			border-color 0.2s ease;
	}

	.link-button:hover {
		transform: translateY(-2px);
	}

	/* Details — solid, subtle */
	.details-btn {
		background: rgba(255, 255, 255, 0.08);
		color: #e0e0e0;
		border: 1px solid rgba(255, 255, 255, 0.12);
	}

	.details-btn:hover {
		background: rgba(255, 255, 255, 0.14);
		border-color: rgba(255, 255, 255, 0.25);
		color: #fff;
	}

	/* GitHub — outlined */
	.readme {
		background: transparent;
		color: #aaa;
		border: 1px solid rgba(255, 255, 255, 0.15);
	}

	.readme:hover {
		background: rgba(255, 255, 255, 0.06);
		color: #fff;
		border-color: rgba(255, 255, 255, 0.3);
	}

	/* Live — outlined, stands out without the jarring white fill */
	.live {
		background: transparent;
		color: rgba(255, 255, 255, 0.88);
		border: 1px solid rgba(255, 255, 255, 0.38);
	}

	.live:hover {
		background: rgba(255, 255, 255, 0.08);
		border-color: rgba(255, 255, 255, 0.7);
		color: #fff;
	}

	.btn-icon {
		width: 14px;
		height: 14px;
		flex-shrink: 0;
	}

	.btn-icon-left {
		width: 14px;
		height: 14px;
		flex-shrink: 0;
	}

	/* --- POPOVER STYLES --- */
	[popover] {
		margin: auto;
		padding: 0;
		border: none;
		border-radius: 16px;
		background: #1c1c1c;
		color: #f0f0f0;
		box-shadow: 0 20px 60px rgba(0, 0, 0, 0.8);
		width: 90%;
		max-width: 680px;
		max-height: 80vh;
		overflow-y: auto;
		border: 1px solid #2e2e2e;
		opacity: 0;
		transform: scale(0.96) translateY(8px);
		transition:
			opacity 0.25s ease,
			transform 0.25s ease,
			display 0.25s ease allow-discrete,
			overlay 0.25s ease allow-discrete;
	}

	[popover]:popover-open {
		opacity: 1;
		transform: scale(1) translateY(0);
	}

	@starting-style {
		[popover]:popover-open {
			opacity: 0;
			transform: scale(0.96) translateY(8px);
		}
	}

	[popover]::backdrop {
		background-color: rgba(0, 0, 0, 0.65);
		backdrop-filter: blur(6px);
		opacity: 0;
		transition:
			opacity 0.25s ease,
			display 0.25s ease allow-discrete,
			overlay 0.25s ease allow-discrete;
	}

	[popover]:popover-open::backdrop {
		opacity: 1;
	}

	@starting-style {
		[popover]:popover-open::backdrop {
			opacity: 0;
		}
	}

	.popover-content {
		padding: 32px;
	}

	.popover-header {
		display: flex;
		justify-content: space-between;
		align-items: flex-start;
		margin-bottom: 20px;
		border-bottom: 1px solid #2a2a2a;
		padding-bottom: 18px;
	}

	.popover-header h3 {
		margin: 0;
		font-size: 1.6rem;
		color: white;
		line-height: 1.2;
	}

	.close-btn {
		background: rgba(255, 255, 255, 0.06);
		border: 1px solid rgba(255, 255, 255, 0.1);
		color: #888;
		font-size: 1.2rem;
		line-height: 1;
		cursor: pointer;
		padding: 6px 10px;
		border-radius: 8px;
		transition: all 0.15s;
		flex-shrink: 0;
	}

	.close-btn:hover {
		background: rgba(255, 255, 255, 0.12);
		color: #fff;
		border-color: rgba(255, 255, 255, 0.2);
	}

	.long-description {
		white-space: pre-line;
		line-height: 1.85;
		color: #ccc;
		font-size: 0.95rem;
	}

	/* =============================================
	   COMPACT (GRID) LAYOUT
	   ============================================= */
	.compact .project-card-inner {
		flex-direction: column;
	}

	.compact .project-image-column {
		min-width: unset;
		width: 100%;
	}

	.compact .project-image,
	.compact .project-image-placeholder {
		min-height: 200px;
		height: 200px;
	}

	.compact .project-image.image-contain {
		padding: 16px;
	}

	.compact .project-content-column {
		padding: 24px;
	}

	.compact .project-title-desktop {
		font-size: 1.3rem;
	}

	.compact .project-short-description {
		font-size: 0.9rem;
		-webkit-line-clamp: 3;
		display: -webkit-box;
		-webkit-box-orient: vertical;
		overflow: hidden;
	}

	.compact .links {
		gap: 8px;
	}

	.compact .link-button {
		padding: 7px 14px;
		font-size: 0.8rem;
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
			min-width: unset;
		}

		.project-image,
		.project-image-placeholder {
			min-height: 200px;
		}

		.featured .project-image,
		.featured .project-image-placeholder {
			min-height: 240px;
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
			font-size: 1.2rem;
		}

		.project-content-column,
		.featured .project-content-column {
			padding: 24px;
		}

		.featured .project-title-desktop {
			font-size: 1.8rem;
		}
	}
</style>
