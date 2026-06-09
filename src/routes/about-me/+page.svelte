<script>
	import { onMount } from 'svelte';

	const flowerPath =
		'M468.89,151.925c-19.302-19.292-43.368-32.797-69.706-39.101c-6.296-26.324-19.79-50.39-39.098-69.702 C332.278,15.317,295.313,0.003,256.002,0.003c-39.375,0.02-76.349,15.344-104.094,43.139 c-19.302,19.252-32.783,43.329-39.068,69.701c-26.378,6.285-50.444,19.77-69.696,39.063C15.344,179.66,0.015,216.634,0,256.015 c0.02,39.33,15.265,76.224,42.936,103.9c19.372,19.372,43.508,32.916,69.89,39.251c6.294,26.343,19.799,50.438,39.136,69.771 c27.77,27.775,64.714,43.07,104.026,43.06c39.31,0,76.414-15.454,104.476-43.518c19.094-19.094,32.45-43.029,38.704-69.334 c26.343-6.294,50.399-19.77,69.682-39.052c27.795-27.795,43.129-64.769,43.149-104.109 C511.97,216.654,496.665,179.7,468.89,151.925z M130.875,126.955l0.488-2.725c0.169-0.985,0.348-1.93,0.532-2.854 c4.983-24.135,16.906-46.173,34.484-63.754c23.946-23.947,55.778-37.133,89.63-37.133s65.674,13.176,89.605,37.113 c14.49,14.49,25.275,32.28,31.36,51.682c-4.036-0.338-8.124-0.497-12.191-0.507c-39.356,0.03-76.334,15.344-104.124,43.13 c-26.786,26.79-42.076,62.421-43.04,100.211l-0.024,0.378c-0.07,1.154-0.14,2.306-0.145,3.5c0,4.317,0.726,8.592,2.123,12.63 c-20.048-6.018-38.336-16.926-53.193-31.783c-23.937-23.936-37.118-55.759-37.118-89.61 C129.264,140.469,129.802,133.647,130.875,126.955z M121.403,380.103c-24.221-5.032-46.347-17.024-63.993-34.667 c-23.812-23.816-36.93-55.57-36.93-89.421c0-33.851,13.192-65.684,37.138-89.631c14.499-14.509,32.29-25.289,51.656-31.355 c-0.333,4.067-0.497,8.155-0.497,12.203c-0.005,39.31,15.309,76.274,43.12,104.089c27.045,27.039,62.964,42.344,100.937,43.061 c1.024,0.09,2.083,0.148,3.127,0.148c4.322,0.01,8.607-0.716,12.65-2.108c-6.021,20.078-16.926,38.366-31.758,53.204 c-23.932,23.926-55.769,37.112-89.65,37.112c-7.737,0-15.414-0.686-22.768-2.018L121.403,380.103z M380.682,387.721 c-0.164,0.885-0.343,1.79-0.567,2.894c-4.942,24.096-16.746,46.013-34.129,63.396c-24.185,24.185-56.147,37.502-89.998,37.511 c-33.851,0-65.654-13.166-89.552-37.063c-14.519-14.519-25.324-32.33-31.405-51.732c4.038,0.328,8.12,0.497,12.182,0.497 c39.356-0.019,76.33-15.334,104.12-43.129c26.836-26.831,42.121-62.552,43.03-100.49c0.105-1.203,0.155-2.406,0.155-3.59 c0.009-4.316-0.707-8.602-2.099-12.65c20.018,5.997,38.237,16.846,52.99,31.593c24.066,24.066,37.322,55.959,37.322,89.81 C382.731,372.406,382.04,380.133,380.682,387.721z M454.371,345.625c-14.48,14.479-32.27,25.259-51.672,31.334 c0.328-4.027,0.497-8.105,0.497-12.172c0-39.33-15.379-76.374-43.313-104.308c-26.721-26.722-62.317-41.947-100.396-42.882 c-1.158-0.069-2.316-0.138-3.525-0.138c-4.306,0.009-8.572,0.735-12.595,2.118c6.016-20.059,16.92-38.347,31.768-53.194 c23.937-23.936,55.77-37.123,89.626-37.113c7.742,0.02,15.514,0.726,23.166,2.118c0.895,0.139,1.785,0.319,2.665,0.507 c24.19,5.012,46.257,16.946,63.83,34.508c23.916,23.916,37.093,55.739,37.093,89.59S478.328,321.668,454.371,345.625z';

	// The bottom dock doubles as the page's section navigator. Each petal jumps to a
	// section and lights up while that section is in view.
	const sections = [
		{ id: 'sec-hero', label: 'Top' },
		{ id: 'sec-craft', label: 'Craft' },
		{ id: 'sec-lifting', label: 'Powerlifting' },
		{ id: 'sec-primedlifter', label: 'PrimedLifter' },
		{ id: 'sec-intogolf', label: 'IntoGolf' },
		{ id: 'sec-toolkit', label: 'Toolkit' },
		{ id: 'sec-contact', label: 'Contact' }
	];

	let activeId = 'sec-hero';

	/** @param {string} id */
	function scrollToSection(id) {
		if (typeof document === 'undefined') return;
		document.getElementById(id)?.scrollIntoView({ behavior: 'smooth', block: 'start' });
	}

	onMount(() => {
		const observer = new IntersectionObserver(
			(entries) => {
				for (const entry of entries) {
					if (entry.isIntersecting) activeId = entry.target.id;
				}
			},
			// Trigger when a section reaches the vertical middle of the viewport.
			{ rootMargin: '-45% 0px -50% 0px', threshold: 0 }
		);
		for (const s of sections) {
			const el = document.getElementById(s.id);
			if (el) observer.observe(el);
		}
		return () => observer.disconnect();
	});

	const skills = [
		'SvelteKit', 'Svelte 5', 'Astro', 'React', 'Vue / Quasar',
		'TypeScript', 'JavaScript', 'HTML & CSS', 'Node.js', '11ty',
		'Supabase', 'Firebase', 'PWA / Service Workers',
		'Vite', 'Vitest', 'Playwright', 'Figma', 'Git'
	];

	// Email split to avoid scraping
	const emailUser = 'sebastiaanhagoort';
	const emailDomain = 'outlook.com';
	const email = `${emailUser}@${emailDomain}`;
</script>

<div class="page-wrapper">
	<div class="scroll-progress-bar"></div>

	<main class="page-content">

		<!-- HERO -->
		<section class="s-hero" id="sec-hero">
			<span class="hero-eyebrow">Frontend Developer · Netherlands</span>
			<h1 class="hero-name">Sebastiaan<br>Hagoort</h1>
			<p class="hero-sub">I build interfaces that feel fast, look intentional, and work on every screen.</p>
			<span class="scroll-hint" aria-hidden="true">scroll</span>
		</section>

		<!-- DIVIDER -->
		<div class="divider" aria-hidden="true"><span>///</span></div>

		<!-- CRAFT -->
		<section class="s-body s-craft" id="sec-craft">
			<p>
				My main stack is <strong>SvelteKit</strong> and <strong>TypeScript</strong>, but I've shipped
				production code across Astro, React, Quasar/Vue, and vanilla JS. I care about the things most
				users never notice until they're broken — scroll performance, animation timing, layout shifts,
				accessibility.
			</p>
		</section>

		<!-- POWERLIFTING PULLQUOTE -->
		<section class="s-pullquote" id="sec-lifting">
			<div class="pullquote-inner">
				<span class="pullquote-mark">"</span>
				<blockquote>
					Outside of code, I'm a competitive powerlifter and a coach. Powerlifting is about as
					data-driven as sport gets — every kilo, every percentage of your max, every training block
					has a measurable outcome.
				</blockquote>
			</div>
		</section>

		<!-- PRIMEDLIFTER -->
		<section class="s-body s-project-callout" id="sec-primedlifter">
			<div class="callout-label">Personal project</div>
			<p>
				I got tired of spreadsheets and built <strong>PrimedLifter</strong> — a PWA that replaces the
				Google Sheets + Drive workflow coaches are still stuck with. It handles RPE calculations,
				auto-updating training maxes, backdown sets, athlete management, and video review in one place.
				Built with SvelteKit and Supabase. Running in production, used with real athletes.
			</p>
		</section>

		<!-- INTOGOLF -->
		<section class="s-body s-right" id="sec-intogolf">
			<div class="callout-label">Internship</div>
			<p>
				On the professional side: I interned at <strong>IntoGolf</strong>, a SaaS company whose
				platform is used by golf academies across the Netherlands. I worked across the public website
				(Astro + React), the V3 coaching app (Quasar/Vue + Firebase), and V3 Pro — lesson management,
				CRM, canvas-based video annotation, Mollie payments, and offline PWA. Real product, real users,
				real constraints.
			</p>
		</section>

		<!-- SKILLS -->
		<section class="s-skills" id="sec-toolkit">
			<p class="skills-label">Toolkit</p>
			<ul class="skills-list" role="list">
				{#each skills as skill}
					<li class="skill-chip">{skill}</li>
				{/each}
			</ul>
		</section>

		<!-- CTA -->
		<section class="s-cta" id="sec-contact">
			<p class="cta-text">
				Looking for a team where I can own meaningful frontend work —<br>
				not just implement designs, but help shape them.
			</p>
			<a href="mailto:{email}" class="cta-link">
				{email}
				<svg viewBox="0 0 16 16" fill="none" class="cta-arrow"><path d="M3 13L13 3M13 3H7M13 3v6" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></svg>
			</a>
		</section>

	</main>

	<!-- DOCK — section navigator -->
	<nav class="dock-wrapper-fixed" aria-label="Section navigation">
		<div class="dock-position-container">
			<div class="dock-glass-panel">
				{#each sections as s}
					<button
						class="dock-item"
						class:active={activeId === s.id}
						on:click={() => scrollToSection(s.id)}
						aria-label="Go to {s.label}"
						aria-current={activeId === s.id ? 'true' : undefined}
					>
						<span class="dock-tip">{s.label}</span>
						<svg viewBox="0 0 512 512" class="flower-icon">
							<path fill="currentColor" d={flowerPath} />
						</svg>
						<span class="dock-dot" aria-hidden="true"></span>
					</button>
				{/each}
			</div>
		</div>
	</nav>
</div>

<style>
	/* =============================================
	   RESET & WRAPPER
	   ============================================= */
	:global(body) {
		margin: 0;
		padding: 0;
		overflow-x: hidden;
	}

	.page-wrapper {
		min-height: 100vh;
		width: 100%;       /* was 100vw — that includes scrollbar width and causes horizontal overflow */
		overflow-x: hidden;
		background-color: #0f0000;
		color: #fff;
		font-family: 'Helvetica Neue', sans-serif;
		position: relative;
	}

	.page-content {
		padding: 0 20px 220px;
		max-width: 780px;
		margin: 0 auto;
	}

	/* =============================================
	   SCROLL PROGRESS BAR
	   ============================================= */
	.scroll-progress-bar {
		position: fixed;
		top: 0;
		right: 0;
		width: 3px;
		height: 100vh;
		transform-origin: top;
		transform: scaleY(0);
		background-color: rgba(255, 200, 200, 0.5);
		z-index: 10000;
		animation: scrollProgress linear;
		animation-timeline: scroll();
	}

	@keyframes scrollProgress {
		from { transform: scaleY(0); }
		to   { transform: scaleY(1); }
	}

	/* =============================================
	   SHARED ANIMATION KEYFRAMES
	   ============================================= */
	@keyframes fade-up {
		from { opacity: 0; transform: translateY(40px); }
		to   { opacity: 1; transform: translateY(0); }
	}

	@keyframes fade-left {
		from { opacity: 0; transform: translateX(-32px); }
		to   { opacity: 1; transform: translateX(0); }
	}

	@keyframes fade-right {
		from { opacity: 0; transform: translateX(32px); }
		to   { opacity: 1; transform: translateX(0); }
	}

	@keyframes scale-up {
		from { opacity: 0; transform: scale(0.95); }
		to   { opacity: 1; transform: scale(1); }
	}

	/* =============================================
	   HERO
	   ============================================= */
	/* Hero is above the fold — view() timeline won't fire on load, use regular animation */
	@keyframes hero-entrance {
		from { opacity: 0; transform: translateY(28px); }
		to   { opacity: 1; transform: translateY(0); }
	}

	.s-hero {
		position: relative;
		padding: 120px 0 80px;
		min-height: 62vh;
		text-align: left;
		animation: hero-entrance 0.9s cubic-bezier(0.16, 1, 0.3, 1) both;
		animation-delay: 0.05s;
	}

	.hero-eyebrow {
		display: block;
		font-size: 0.8rem;
		letter-spacing: 0.15em;
		text-transform: uppercase;
		color: rgba(255, 180, 180, 0.6);
		margin-bottom: 20px;
	}

	/* The name dissolves — blurs, fades and lifts — as you scroll past the hero.
	   Scroll-driven (not time-driven), so it tracks the scrollbar both ways. */
	@keyframes name-dissolve {
		to {
			opacity: 0;
			filter: blur(16px);
			transform: translateY(-26px) scale(0.97);
			letter-spacing: 0.06em;
		}
	}

	.hero-name {
		font-family: 'Playfair Display', serif;
		font-size: clamp(2.4rem, 7vw, 5rem);
		font-weight: 700;
		line-height: 1.05;
		margin: 0 0 28px;
		color: #fff;
		text-wrap: balance;
		letter-spacing: -0.02em;
		animation: name-dissolve linear both;
		animation-timeline: scroll(root);
		animation-range: 0 340px;
	}

	.hero-sub {
		font-size: clamp(1rem, 2.5vw, 1.25rem);
		color: rgba(255, 200, 200, 0.7);
		line-height: 1.6;
		max-width: 520px;
		margin: 0;
		animation: name-dissolve linear both;
		animation-timeline: scroll(root);
		animation-range: 40px 300px;
	}

	/* small "scroll" cue that fades out as you start scrolling */
	.scroll-hint {
		position: absolute;
		left: 2px;
		bottom: 24px;
		font-size: 0.62rem;
		letter-spacing: 0.4em;
		text-transform: uppercase;
		color: rgba(255, 180, 180, 0.4);
		animation: scroll-hint-fade linear both;
		animation-timeline: scroll(root);
		animation-range: 0 160px;
	}

	.scroll-hint::after {
		content: '';
		display: block;
		width: 1px;
		height: 28px;
		margin: 8px 0 0;
		background: linear-gradient(rgba(255, 180, 180, 0.5), transparent);
	}

	@keyframes scroll-hint-fade {
		to { opacity: 0; transform: translateY(10px); }
	}

	/* =============================================
	   DIVIDER
	   ============================================= */
	.divider {
		padding: 0 0 60px;
		color: rgba(255, 255, 255, 0.08);
		font-size: 1.2rem;
		letter-spacing: 0.4em;
		user-select: none;

		animation: fade-up linear both;
		animation-timeline: view();
		animation-range: entry 5% cover 30%;
	}

	/* =============================================
	   BODY SECTIONS
	   ============================================= */
	/* Consistent left "spine" for every body section, each marked with a node dot —
	   reads as a deliberate timeline rather than a stack of bordered blocks. */
	.s-body {
		position: relative;
		padding: 4px 0 64px 28px;
		border-left: 1px solid rgba(255, 255, 255, 0.1);
	}

	.s-body::before {
		content: '';
		position: absolute;
		left: -5px;
		top: 6px;
		width: 9px;
		height: 9px;
		border-radius: 50%;
		background: rgba(255, 120, 120, 0.55);
		box-shadow: 0 0 0 4px rgba(255, 120, 120, 0.1);
	}

	.s-body p {
		font-size: clamp(1rem, 2vw, 1.15rem);
		line-height: 1.75;
		color: rgba(255, 210, 210, 0.85);
		margin: 0;
		text-wrap: pretty;
	}

	.s-body strong {
		color: #fff;
		font-weight: 600;
	}

	.s-craft {
		animation: fade-left linear both;
		animation-timeline: view();
		animation-range: entry 5% cover 30%;
	}

	.s-project-callout {
		animation: fade-left linear both;
		animation-timeline: view();
		animation-range: entry 8% cover 35%;
	}

	.s-right {
		animation: fade-left linear both;
		animation-timeline: view();
		animation-range: entry 8% cover 35%;
	}

	.callout-label {
		font-size: 0.7rem;
		letter-spacing: 0.12em;
		text-transform: uppercase;
		color: rgba(255, 150, 150, 0.5);
		margin-bottom: 10px;
	}

	/* =============================================
	   PULLQUOTE
	   ============================================= */
	.s-pullquote {
		padding: 0 0 72px;

		animation: scale-up linear both;
		animation-timeline: view();
		animation-range: entry 5% cover 30%;
	}

	.pullquote-inner {
		position: relative;
		padding: 32px 32px 32px 56px;
		border-top: 1px solid rgba(255, 255, 255, 0.12);
		border-bottom: 1px solid rgba(255, 255, 255, 0.12);
	}

	.pullquote-mark {
		position: absolute;
		top: 16px;
		left: 16px;
		font-family: 'Playfair Display', serif;
		font-size: 5rem;
		line-height: 1;
		color: rgba(255, 100, 100, 0.2);
		user-select: none;
		pointer-events: none;
	}

	blockquote {
		margin: 0;
		font-family: 'Playfair Display', serif;
		font-size: clamp(1.1rem, 2.5vw, 1.4rem);
		line-height: 1.65;
		color: rgba(255, 220, 220, 0.9);
		font-style: italic;
	}

	/* =============================================
	   SKILLS
	   ============================================= */
	.s-skills {
		padding: 0 0 72px;

		animation: fade-up linear both;
		animation-timeline: view();
		animation-range: entry 8% cover 35%;
	}

	.skills-label {
		font-size: 0.7rem;
		letter-spacing: 0.15em;
		text-transform: uppercase;
		color: rgba(255, 150, 150, 0.5);
		margin: 0 0 16px;
	}

	.skills-list {
		list-style: none;
		margin: 0;
		padding: 0;
		display: flex;
		flex-wrap: wrap;
		gap: 8px;
	}

	.skill-chip {
		padding: 5px 14px;
		border-radius: 100px;
		border: 1px solid rgba(255, 255, 255, 0.12);
		font-size: 0.82rem;
		color: rgba(255, 200, 200, 0.7);
		transition: border-color 0.2s, color 0.2s;
		cursor: default;
	}

	.skill-chip:hover {
		border-color: rgba(255, 200, 200, 0.4);
		color: #fff;
	}

	/* =============================================
	   CTA
	   ============================================= */
	.s-cta {
		padding: 0 0 40px;
		text-align: center;

		animation: fade-up linear both;
		animation-timeline: view();
		animation-range: entry 10% cover 40%;
	}

	.cta-text {
		font-size: clamp(1rem, 2vw, 1.2rem);
		line-height: 1.7;
		color: rgba(255, 200, 200, 0.7);
		margin: 0 0 28px;
	}

	.cta-link {
		display: inline-flex;
		align-items: center;
		gap: 8px;
		color: #fff;
		text-decoration: none;
		font-size: 1rem;
		font-weight: 500;
		border-bottom: 1px solid rgba(255, 255, 255, 0.25);
		padding-bottom: 2px;
		transition: border-color 0.2s, color 0.2s;
	}

	.cta-link:hover {
		border-color: rgba(255, 255, 255, 0.7);
		color: #fff;
	}

	.cta-arrow {
		width: 14px;
		height: 14px;
		flex-shrink: 0;
	}

	/* =============================================
	   DOCK — WRAPPER
	   ============================================= */
	.dock-wrapper-fixed {
		position: fixed;
		bottom: 0;
		left: 0;
		width: 100vw;
		height: 0;
		display: flex;
		justify-content: center;
		z-index: 9999;
	}

	.dock-position-container {
		position: absolute;
		bottom: 24px;
		width: 100%;
		display: flex;
		justify-content: center;
		padding: 0 12px;
		box-sizing: border-box;
	}

	/* =============================================
	   DOCK — GLASS PANEL
	   ============================================= */
	.dock-glass-panel {
		display: flex;
		align-items: flex-end;
		justify-content: center;
		height: 70px;
		padding: 0 8px 20px;
		max-width: calc(100vw - 24px);
		/* visible so magnified petals + tooltips can rise above the panel */
		overflow: visible;

		background: rgba(255, 255, 255, 0.06);
		border-radius: 20px;
		border: 1px solid rgba(255, 255, 255, 0.1);
		box-shadow: 0 12px 32px rgba(0, 0, 0, 0.5);
		backdrop-filter: blur(14px);

		animation: dockBorderDance linear;
		animation-timeline: scroll();
	}

	@keyframes dockBorderDance {
		0%   { border-color: rgba(255, 255, 255, 0.08); }
		50%  { border-color: rgba(200, 100, 100, 0.35); }
		100% { border-color: rgba(255, 180, 180, 0.6); }
	}

	/* =============================================
	   DOCK — ITEMS (responsive sizes via clamp)
	   ============================================= */
	.dock-item {
		appearance: none;
		background: none;
		border: none;
		padding: 0;
		cursor: pointer;
		position: relative;
		width: clamp(28px, 4vw, 44px);
		height: clamp(28px, 4vw, 44px);
		flex: 0 0 clamp(28px, 4vw, 44px);
		margin: 0 clamp(3px, 0.6vw, 6px);
		display: flex;
		align-items: center;
		justify-content: center;
		transition:
			flex-basis 0.2s cubic-bezier(0.25, 0.46, 0.45, 0.94),
			width 0.2s cubic-bezier(0.25, 0.46, 0.45, 0.94),
			height 0.2s cubic-bezier(0.25, 0.46, 0.45, 0.94),
			transform 0.2s cubic-bezier(0.25, 0.46, 0.45, 0.94);
	}

	.flower-icon {
		width: 100%;
		height: 100%;
		color: rgba(255, 255, 255, 0.4);
		filter: drop-shadow(0 1px 3px rgba(0, 0, 0, 0.5));
		pointer-events: none;
		transition: color 0.2s, transform 0.2s;
	}

	/* Active section: petal lights up + a dot appears beneath it */
	.dock-item.active .flower-icon {
		color: rgba(255, 150, 150, 0.95);
		filter: drop-shadow(0 0 6px rgba(255, 120, 120, 0.5));
	}

	.dock-dot {
		position: absolute;
		bottom: 4px;
		left: 50%;
		width: 3px;
		height: 3px;
		border-radius: 50%;
		background: rgba(255, 150, 150, 0.95);
		transform: translateX(-50%) scale(0);
		transition: transform 0.2s ease;
		pointer-events: none;
	}

	.dock-item.active .dock-dot {
		transform: translateX(-50%) scale(1);
	}

	/* Tooltip label above the petal on hover */
	.dock-tip {
		position: absolute;
		bottom: calc(100% + 6px);
		left: 50%;
		transform: translateX(-50%) translateY(4px);
		padding: 4px 9px;
		border-radius: 6px;
		background: rgba(20, 5, 5, 0.92);
		border: 1px solid rgba(255, 255, 255, 0.12);
		color: rgba(255, 220, 220, 0.95);
		font-size: 0.62rem;
		letter-spacing: 0.06em;
		white-space: nowrap;
		opacity: 0;
		pointer-events: none;
		transition: opacity 0.15s ease, transform 0.15s ease;
	}

	.dock-item::after {
		content: '';
		position: absolute;
		inset: -12px;
		background: transparent;
	}

	/* Hover magnification — only on devices that support hover */
	@media (hover: hover) {
		.dock-item:hover {
			flex-basis: clamp(44px, 7vw, 80px);
			width: clamp(44px, 7vw, 80px);
			height: clamp(44px, 7vw, 80px);
			transform: translateY(-18px);
			z-index: 10;
		}

		.dock-item:hover .flower-icon { color: #fff; }

		.dock-item:hover .dock-tip {
			opacity: 1;
			transform: translateX(-50%) translateY(0);
		}

		.dock-item:hover + .dock-item,
		.dock-item:has(+ .dock-item:hover) {
			flex-basis: clamp(36px, 5.5vw, 62px);
			width: clamp(36px, 5.5vw, 62px);
			height: clamp(36px, 5.5vw, 62px);
			transform: translateY(-9px);
		}

		.dock-item:hover + .dock-item + .dock-item,
		.dock-item:has(+ .dock-item + .dock-item:hover) {
			flex-basis: clamp(30px, 4.5vw, 50px);
			width: clamp(30px, 4.5vw, 50px);
			height: clamp(30px, 4.5vw, 50px);
			transform: translateY(-4px);
		}
	}
</style>
