<script>
	/**
	 * SVELTE 5 VERSION
	 * We use standard JS variables here. If this list needed to change
	 * dynamically, we would use: let items = $state([...]);
	 */

	// 1. The flower path from your previous request
	const flowerPath =
		'M468.89,151.925c-19.302-19.292-43.368-32.797-69.706-39.101c-6.296-26.324-19.79-50.39-39.098-69.702 C332.278,15.317,295.313,0.003,256.002,0.003c-39.375,0.02-76.349,15.344-104.094,43.139 c-19.302,19.252-32.783,43.329-39.068,69.701c-26.378,6.285-50.444,19.77-69.696,39.063C15.344,179.66,0.015,216.634,0,256.015 c0.02,39.33,15.265,76.224,42.936,103.9c19.372,19.372,43.508,32.916,69.89,39.251c6.294,26.343,19.799,50.438,39.136,69.771 c27.77,27.775,64.714,43.07,104.026,43.06c39.31,0,76.414-15.454,104.476-43.518c19.094-19.094,32.45-43.029,38.704-69.334 c26.343-6.294,50.399-19.77,69.682-39.052c27.795-27.795,43.129-64.769,43.149-104.109 C511.97,216.654,496.665,179.7,468.89,151.925z M130.875,126.955l0.488-2.725c0.169-0.985,0.348-1.93,0.532-2.854 c4.983-24.135,16.906-46.173,34.484-63.754c23.946-23.947,55.778-37.133,89.63-37.133s65.674,13.176,89.605,37.113 c14.49,14.49,25.275,32.28,31.36,51.682c-4.036-0.338-8.124-0.497-12.191-0.507c-39.356,0.03-76.334,15.344-104.124,43.13 c-26.786,26.79-42.076,62.421-43.04,100.211l-0.024,0.378c-0.07,1.154-0.14,2.306-0.145,3.5c0,4.317,0.726,8.592,2.123,12.63 c-20.048-6.018-38.336-16.926-53.193-31.783c-23.937-23.936-37.118-55.759-37.118-89.61 C129.264,140.469,129.802,133.647,130.875,126.955z M121.403,380.103c-24.221-5.032-46.347-17.024-63.993-34.667 c-23.812-23.816-36.93-55.57-36.93-89.421c0-33.851,13.192-65.684,37.138-89.631c14.499-14.509,32.29-25.289,51.656-31.355 c-0.333,4.067-0.497,8.155-0.497,12.203c-0.005,39.31,15.309,76.274,43.12,104.089c27.045,27.039,62.964,42.344,100.937,43.061 c1.024,0.09,2.083,0.148,3.127,0.148c4.322,0.01,8.607-0.716,12.65-2.108c-6.021,20.078-16.926,38.366-31.758,53.204 c-23.932,23.926-55.769,37.112-89.65,37.112c-7.737,0-15.414-0.686-22.768-2.018L121.403,380.103z M380.682,387.721 c-0.164,0.885-0.343,1.79-0.567,2.894c-4.942,24.096-16.746,46.013-34.129,63.396c-24.185,24.185-56.147,37.502-89.998,37.511 c-33.851,0-65.654-13.166-89.552-37.063c-14.519-14.519-25.324-32.33-31.405-51.732c4.038,0.328,8.12,0.497,12.182,0.497 c39.356-0.019,76.33-15.334,104.12-43.129c26.836-26.831,42.121-62.552,43.03-100.49c0.105-1.203,0.155-2.406,0.155-3.59 c0.009-4.316-0.707-8.602-2.099-12.65c20.018,5.997,38.237,16.846,52.99,31.593c24.066,24.066,37.322,55.959,37.322,89.81 C382.731,372.406,382.04,380.133,380.682,387.721z M454.371,345.625c-14.48,14.479-32.27,25.259-51.672,31.334 c0.328-4.027,0.497-8.105,0.497-12.172c0-39.33-15.379-76.374-43.313-104.308c-26.721-26.722-62.317-41.947-100.396-42.882 c-1.158-0.069-2.316-0.138-3.525-0.138c-4.306,0.009-8.572,0.735-12.595,2.118c6.016-20.059,16.92-38.347,31.768-53.194 c23.937-23.936,55.77-37.123,89.626-37.113c7.742,0.02,15.514,0.726,23.166,2.118c0.895,0.139,1.785,0.319,2.665,0.507 c24.19,5.012,46.257,16.946,63.83,34.508c23.916,23.916,37.093,55.739,37.093,89.59S478.328,321.668,454.371,345.625z';

	// 2. Create 16 items
	const items = Array.from({ length: 16 }, (_, i) => i);
</script>

<div class="demo-page-wrapper">
	<div class="dock-position-container">
		<div class="dock-glass-panel">
			{#each items as item}
				<button class="dock-item" aria-label="Flower {item}">
					<svg viewBox="0 0 512 512" class="flower-icon">
						<path fill="currentColor" d={flowerPath} />
					</svg>
				</button>
			{/each}
		</div>
	</div>
</div>

<style>
	/* 1. Page Layout */
	:global(body) {
		margin: 0;
		background-color: #1f0000;
		overflow-x: hidden;
	}

	.demo-page-wrapper {
		position: fixed;
		inset: 0;
		width: 100vw;
		height: 100vh;
		display: flex;
		align-items: flex-end;
		justify-content: center;
		padding-bottom: 50px;
		z-index: 9999;
	}

	/* 2. Dock Container */
	.dock-position-container {
		/* Fixed width wrapper to prevent page layout shifts */
		width: 100%;
		display: flex;
		justify-content: center; /* Ensures the dock expands from the center */
		margin-bottom: 1em;
	}

	.dock-glass-panel {
		display: flex;
		/* ALIGNMENT FIX 1: Anchor to bottom so they grow UP */
		align-items: flex-end;
		justify-content: center;

		height: 80px;

		/* ALIGNMENT FIX 2: Use padding to visually center the items.
		   (80px height - 50px item) / 2 = 15px bottom padding.
		   This makes them look centered but keeps them anchored to the floor. */
		padding: 0 10px 25px 10px;

		background: rgba(255, 255, 255, 0.08);
		border-radius: 24px;
		border: 1px solid rgba(255, 255, 255, 0.1);
		box-shadow: 0 15px 40px rgba(0, 0, 0, 0.6);
		backdrop-filter: blur(15px);

		/* WIDTH FIX: This prevents the dock from snapping */
		transition: width 0.3s ease-out;
		will-change: width;
	}

	/* 3. The Items */
	.dock-item {
		appearance: none;
		background: none;
		border: none;
		padding: 0;
		cursor: pointer;
		position: relative;

		/* Base Size */
		width: 50px;
		height: 50px;

		/* JITTER FIX: Use flex-basis for smoother flow than width */
		flex: 0 0 50px;

		margin: 0 6px; /* consistent spacing */

		display: flex;
		align-items: center; /* Icons centered inside the button */
		justify-content: center;

		/* Smooth physics */
		transition:
			flex-basis 0.2s cubic-bezier(0.25, 0.46, 0.45, 0.94),
			width 0.2s cubic-bezier(0.25, 0.46, 0.45, 0.94),
			height 0.2s cubic-bezier(0.25, 0.46, 0.45, 0.94),
			transform 0.2s cubic-bezier(0.25, 0.46, 0.45, 0.94);
	}

	.flower-icon {
		width: 100%;
		height: 100%;
		color: rgba(255, 255, 255, 0.8);
		filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.5));
		pointer-events: none;
		transition: color 0.2s;
	}

	/* Safety Net (Keeps mouse attached) */
	.dock-item::after {
		content: '';
		position: absolute;
		inset: -15px;
		background: transparent;
	}

	/* 4. Hover Effects */

	/* HOVERED ITEM */
	.dock-item:hover {
		/* Grow Width (pushes neighbors) */
		flex-basis: 90px;
		width: 90px;

		/* Grow Height (goes upwards) */
		height: 90px;

		/* Lift slightly to clear the baseline */
		transform: translateY(-20px);

		z-index: 10;
	}

	.dock-item:hover .flower-icon {
		color: #fff;
	}

	/* IMMEDIATE NEIGHBORS */
	.dock-item:hover + .dock-item,
	.dock-item:has(+ .dock-item:hover) {
		flex-basis: 70px;
		width: 70px;
		height: 70px;
		transform: translateY(-10px);
	}

	/* SECOND NEIGHBORS (Optional - adds "curve" to make it feel less jerky) */
	.dock-item:hover + .dock-item + .dock-item,
	.dock-item:has(+ .dock-item + .dock-item:hover) {
		flex-basis: 55px;
		width: 55px;
		height: 55px;
		transform: translateY(-5px);
	}

	@media (max-width: 768px) {
		.dock-position-container {
			width: 95%;
			overflow-x: auto;
		}
		/* Disable physics on mobile to prevent bugs */
		.dock-item:hover,
		.dock-item:hover + .dock-item,
		.dock-item:has(+ .dock-item:hover) {
			flex-basis: 50px;
			width: 50px;
			height: 50px;
			transform: none;
		}
	}
</style>
