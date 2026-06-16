<script>
	const plDescription = `For years, the standard setup for a powerlifting coach looked like this: a Google Sheet with a custom-built program, a Google Drive folder full of lifting videos, and a constant back-and-forth with athletes to keep both in sync. It works — barely — but it's slow, fragile, and not built for the sport.

PrimedLifter replaces all of that in one place.

The core of the app is the program builder. Coaches create complex RPE-based programs — the kind that require calculating training maxes, setting up backdown sets, managing volume across a training block — and the app handles all the maths automatically. Training maxes auto-update based on logged performance. Backdowns calculate off the top set. RPE targets stay accurate as athletes get stronger. No more manual formula chains in a spreadsheet.

Athletes log their sessions directly in the app and attach links to their lifting videos. Coaches review technique without ever leaving the platform — no Drive tab, no tracking down links in WhatsApp, no "did you send the video yet?" Every session, every lift, every note is in one place.

Key features:
— RPE-based program builder with auto-calculating training maxes and backdown sets
— Full Tuchscherer RPE chart for accurate intensity targeting
— Session logging with video link attachment
— Athlete dashboard for coaches — see every athlete's week at a glance
— Lift history and progress tracking (squat, bench, deadlift)
— Meet management for competition prep
— Calendar view for block planning
— Offline-capable PWA — works in the gym without signal

Stack: SvelteKit (Svelte 5), Supabase (Postgres + Auth), Vite, Vitest, PWA service worker. Live at primedlifter.app — actively used by real coaches and athletes.`;

	const plFeatures = [
		'RPE program builder',
		'Auto training maxes',
		'Backdown set calc',
		'Tuchscherer chart',
		'Session + video logs',
		'Athlete dashboard',
		'Meet management',
		'Calendar planning',
		'Offline PWA'
	];

	const plStack = ['SvelteKit', 'Svelte 5', 'Supabase', 'Vite', 'PWA'];

	const projects = [
		{
			id: 'intogolf',
			num: '02',
			title: 'IntoGolf — Internship',
			subtitle: 'V3, V3 Pro & Website',
			tag: 'Internship',
			shortDescription:
				'Frontend intern at a SaaS platform for golf professionals across the Netherlands. Shipped across three products — an Astro marketing site, the V3 coaching app (Vue + Element Plus / Laravel), and the V3 Pro platform (Quasar/Vue) — focused on scheduling & planning, CRM-style customer views, and offline PWA support.',
			description: `IntoGolf is a SaaS company whose platform is used by golf professionals and academies across the Netherlands to manage lessons, availability, bookings, and student relationships. I joined as a frontend intern and worked across three parts of the product simultaneously.

The public website (intogolf.nl) is built in Astro — static-first pages with inlined critical CSS and optimised fonts. My work here focused on performance and structure: fast-loading marketing pages that stay easy for the team to maintain.

V3 is the core coaching app — a Vue + Element Plus frontend inside a Laravel application (Laravel Mix, with Pusher / Laravel Echo for real-time updates). Coming in mid-development meant getting up to speed fast on a large existing codebase. I worked across scheduling and planning — calendar and planner views (FullCalendar), lesson/clinic/event creation, tee-time booking, and coach availability — and built CRM-style customer views: a 360° customer profile with notes and history.

V3 Pro is the expanded Quasar/Vue platform for larger academies. Here I focused on the offline PWA layer — a service worker with update handling, push notifications, and connection-status detection — so coaches can keep working without signal out on the course.

Stack: Astro (website) · Vue + Element Plus, Laravel Mix, Pusher, FullCalendar (V3) · Quasar/Vue, Workbox PWA, push notifications (V3 Pro).`,
			liveLink: 'https://intogolf.nl',
			readmeLink: '',
			logo: '/intogolf-logo.svg'
		},
		{
			id: 'funda',
			num: '03',
			title: 'Funda Listing Detail Page',
			subtitle: 'SSR · Progressive Enhancement',
			tag: 'School project',
			shortDescription:
				"Rebuilt Funda's Listing Detail Page in three weeks — semantic HTML first, CSS second, JS strictly last. AVIF image optimisation via <picture> for real-world load-time gains on a listing with dozens of photos.",
			description: `The assignment: rebuild Funda's Listing Detail Page (LDP) from scratch in three weeks, using Server-Side Rendering and Liquid templating.

Progressive enhancement was the guiding principle throughout. The page is built to work at every layer — semantic HTML first, then CSS layout, then JS enhancements on top. Nothing breaks if JavaScript doesn't load.

On the server, Liquid templating handles the page structure modularly. House images are served through the <picture> element with AVIF conversion where the browser supports it, falling back gracefully to JPEG. On a listing page with dozens of photos, that makes a real difference to load time.

Stack: HTML, CSS, JavaScript, Node.js, Liquid templating, SSR, AVIF image optimisation.`,
			liveLink: 'https://proof-of-concept-1-6ez0.onrender.com/',
			readmeLink: 'https://github.com/Sebastiaan-hva/proof-of-concept'
		},
		{
			id: 'embassy',
			num: '04',
			title: 'Embassy of the Free Mind',
			subtitle: 'SvelteKit · Headless CMS',
			tag: 'School project',
			shortDescription:
				"Full website redesign for an Amsterdam foundation in SvelteKit. Reusable components, a focus on accessibility, and GSAP-driven motion in a content-first layout that reflects the foundation's ethos of open inquiry.",
			description: `Full redesign for the Embassy of the Free Mind using SvelteKit. The EFM is an Amsterdam foundation dedicated to freedom of thought and open inquiry — the design reflects that with a clean, content-first layout.

Focus areas: component reusability, accessibility, and GSAP-driven animation for a polished, content-first experience.`,
			liveLink: 'https://embassyofthefreemind.dev.fdnd.nl/',
			readmeLink: ''
		},
		{
			id: 'squad',
			num: '05',
			title: 'Squad Page Yearbook',
			subtitle: 'FDND Collaboration',
			tag: 'School project',
			shortDescription:
				"End-of-year collaborative project with my FDND squad. Custom animations, profile cards, shared memory wall — coordinated across branches without breaking each other's work.",
			description: `A collaborative end-of-year project with my FDND squad. Each team member contributed their own section — the result is a digital yearbook with custom animations, profile cards, and a shared memory wall. A fun project that pushed us to coordinate across branches without breaking each other's work.`,
			liveLink: '',
			readmeLink: ''
		}
	];
</script>

<div class="projects-page">
	<!-- ── PAGE HEADER ───────────────────────────────────────────────────────── -->
	<header class="page-header">
		<span class="eyebrow">Selected work</span>
		<h1 class="page-title">Projects</h1>
	</header>

	<!-- ── 01 PRIMEDLIFTER HERO ──────────────────────────────────────────────── -->
	<section class="pl-hero">
		<div class="pl-inner">
			<div class="pl-left">
				<div class="pl-meta-row">
					<span class="pl-num">01</span>
					<span class="pl-badge">Personal project</span>
				</div>

				<h2 class="pl-title">PrimedLifter</h2>

				<p class="pl-tagline">The Google Sheets workflow, rebuilt as a real coaching app.</p>

				<ul class="pl-features" role="list">
					{#each plFeatures as feat}
						<li>{feat}</li>
					{/each}
				</ul>

				<div class="pl-stack">
					{#each plStack as s, i}
						<span>{s}</span>{#if i < plStack.length - 1}<span class="sep" aria-hidden="true">·</span
							>{/if}
					{/each}
				</div>

				<div class="pl-cta">
					<button class="pl-btn-detail" popovertarget="pl-popover">
						Details
						<svg viewBox="0 0 16 16" fill="none" width="13" height="13" aria-hidden="true">
							<path
								d="M6 12l4-4-4-4"
								stroke="currentColor"
								stroke-width="1.5"
								stroke-linecap="round"
								stroke-linejoin="round"
							/>
						</svg>
					</button>
					<a
						href="https://primedlifter.app"
						target="_blank"
						rel="noopener noreferrer"
						class="pl-btn-live"
					>
						primedlifter.app
						<svg viewBox="0 0 16 16" fill="none" width="12" height="12" aria-hidden="true">
							<path
								d="M3 13L13 3M13 3H7M13 3v6"
								stroke="currentColor"
								stroke-width="1.5"
								stroke-linecap="round"
								stroke-linejoin="round"
							/>
						</svg>
					</a>
				</div>
			</div>

			<div class="pl-right" aria-hidden="true">
				<img src="/primedlifter-logo.svg" alt="" class="pl-logo" />
			</div>
		</div>
	</section>

	<!-- PrimedLifter detail popover -->
	<div id="pl-popover" popover class="detail-popover">
		<div class="popover-content">
			<div class="popover-hdr">
				<h3>PrimedLifter</h3>
				<button class="close-btn" popovertarget="pl-popover" popovertargetaction="hide">×</button>
			</div>
			<p class="popover-body">{plDescription}</p>
			<div class="popover-footer">
				<a
					href="https://primedlifter.app"
					target="_blank"
					rel="noopener noreferrer"
					class="popover-link"
				>
					primedlifter.app ↗
				</a>
			</div>
		</div>
	</div>

	<!-- ── 02–05 OTHER WORK LIST ─────────────────────────────────────────────── -->
	<section class="other-section">
		<div class="other-header">
			<span class="other-eyebrow">More work</span>
		</div>

		{#each projects as proj, i}
			<article class="proj-row" style="--i: {i}">
				<span class="proj-num">{proj.num}</span>

				<div class="proj-body">
					<div class="proj-top">
						{#if proj.logo}
							<img src={proj.logo} alt="{proj.title} logo" class="proj-logo" />
						{/if}
						<h3 class="proj-title">{proj.title}</h3>
						<span class="proj-subtitle">{proj.subtitle}</span>
						<span class="proj-tag">{proj.tag}</span>
					</div>

					<p class="proj-desc">{proj.shortDescription}</p>

					<div class="proj-links">
						<button class="proj-btn" popovertarget="proj-{proj.id}">Details</button>
						{#if proj.readmeLink}
							<a
								href={proj.readmeLink}
								target="_blank"
								rel="noopener noreferrer"
								class="proj-btn ghost">GitHub</a
							>
						{/if}
						{#if proj.liveLink}
							<a
								href={proj.liveLink}
								target="_blank"
								rel="noopener noreferrer"
								class="proj-btn ghost">Live ↗</a
							>
						{/if}
					</div>
				</div>
			</article>

			<div id="proj-{proj.id}" popover class="detail-popover">
				<div class="popover-content">
					<div class="popover-hdr">
						<h3>{proj.title}</h3>
						<button class="close-btn" popovertarget="proj-{proj.id}" popovertargetaction="hide"
							>×</button
						>
					</div>
					<p class="popover-body">{proj.description}</p>
					{#if proj.readmeLink || proj.liveLink}
						<div class="popover-footer">
							{#if proj.readmeLink}
								<a
									href={proj.readmeLink}
									target="_blank"
									rel="noopener noreferrer"
									class="popover-link">GitHub ↗</a
								>
							{/if}
							{#if proj.liveLink}
								<a
									href={proj.liveLink}
									target="_blank"
									rel="noopener noreferrer"
									class="popover-link">Live site ↗</a
								>
							{/if}
						</div>
					{/if}
				</div>
			</div>
		{/each}
	</section>
</div>

<style>
	:global(body) {
		margin: 0;
		padding: 0;
		background: #0e0e0e;
	}

	.projects-page {
		padding: 0 clamp(24px, 5vw, 80px) 120px;
		background: #0e0e0e;
		min-height: 100vh;
		color: white;
		font-family: 'Helvetica Neue', system-ui, sans-serif;
	}

	/* ── Header ──────────────────────────────────────────────────────────────── */
	.page-header {
		padding: 72px 0 48px;
		border-bottom: 1px solid rgba(255, 255, 255, 0.06);
	}

	.eyebrow {
		display: block;
		font-size: 0.68rem;
		letter-spacing: 0.15em;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.28);
		margin-bottom: 10px;
	}

	.page-title {
		font-family: 'Playfair Display', serif;
		font-size: clamp(3rem, 8vw, 6rem);
		font-weight: 700;
		margin: 0;
		line-height: 1;
		letter-spacing: -0.02em;
		color: #fff;
	}

	/* ── PrimedLifter Hero ───────────────────────────────────────────────────── */
	.pl-hero {
		display: flex;
		align-items: flex-start;
		border-bottom: 1px solid rgba(255, 255, 255, 0.06);
		padding: 56px 0 72px;
	}

	.pl-inner {
		width: 100%;
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 60px;
		align-items: center;
	}

	.pl-left {
		display: flex;
		flex-direction: column;
		gap: 22px;
	}

	.pl-meta-row {
		display: flex;
		align-items: center;
		gap: 12px;
	}

	.pl-num {
		font-size: 0.65rem;
		letter-spacing: 0.12em;
		color: rgba(255, 255, 255, 0.18);
	}

	.pl-badge {
		display: inline-block;
		padding: 3px 10px;
		border-radius: 100px;
		border: 1px solid rgba(255, 255, 255, 0.12);
		color: rgba(255, 255, 255, 0.35);
		font-size: 0.62rem;
		letter-spacing: 0.08em;
		text-transform: uppercase;
	}

	.pl-title {
		font-family: 'Playfair Display', serif;
		font-size: clamp(2.4rem, 4.5vw, 4.5rem);
		font-weight: 700;
		margin: 0;
		line-height: 1.05;
		letter-spacing: -0.02em;
		color: #fff;
	}

	.pl-tagline {
		font-size: clamp(0.95rem, 1.4vw, 1.15rem);
		color: rgba(255, 255, 255, 0.52);
		margin: 0;
		line-height: 1.55;
		max-width: 420px;
	}

	/* Feature chips */
	.pl-features {
		list-style: none;
		padding: 0;
		margin: 0;
		display: flex;
		flex-wrap: wrap;
		gap: 7px;
	}

	.pl-features li {
		padding: 5px 13px;
		border-radius: 100px;
		background: rgba(255, 255, 255, 0.04);
		border: 1px solid rgba(255, 255, 255, 0.09);
		font-size: 0.76rem;
		color: rgba(255, 255, 255, 0.58);
		letter-spacing: 0.01em;
		transition:
			border-color 0.2s,
			color 0.2s;
	}

	.pl-features li:hover {
		border-color: rgba(255, 255, 255, 0.2);
		color: rgba(255, 255, 255, 0.8);
	}

	/* Stack */
	.pl-stack {
		display: flex;
		flex-wrap: wrap;
		align-items: center;
		gap: 6px;
		font-size: 0.68rem;
		letter-spacing: 0.06em;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.22);
	}

	.sep {
		color: rgba(255, 255, 255, 0.1);
	}

	/* CTA buttons */
	.pl-cta {
		display: flex;
		gap: 10px;
		flex-wrap: wrap;
	}

	.pl-btn-detail,
	.pl-btn-live {
		display: inline-flex;
		align-items: center;
		gap: 6px;
		padding: 10px 22px;
		border-radius: 100px;
		font-size: 0.875rem;
		font-weight: 600;
		cursor: pointer;
		text-decoration: none;
		transition: all 0.2s ease;
		font-family: 'Helvetica Neue', sans-serif;
		letter-spacing: 0.01em;
	}

	.pl-btn-detail {
		background: rgba(255, 255, 255, 0.07);
		color: rgba(255, 255, 255, 0.78);
		border: 1px solid rgba(255, 255, 255, 0.11);
	}

	.pl-btn-detail:hover {
		background: rgba(255, 255, 255, 0.13);
		color: #fff;
		border-color: rgba(255, 255, 255, 0.22);
		transform: translateY(-2px);
	}

	.pl-btn-live {
		background: transparent;
		color: rgba(255, 255, 255, 0.82);
		border: 1px solid rgba(255, 255, 255, 0.32);
	}

	.pl-btn-live:hover {
		background: rgba(255, 255, 255, 0.07);
		border-color: rgba(255, 255, 255, 0.6);
		color: #fff;
		transform: translateY(-2px);
	}

	/* Logo */
	.pl-right {
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.pl-logo {
		width: 100%;
		max-width: 300px;
		height: auto;
		opacity: 0.88;
		filter: drop-shadow(0 0 48px rgba(255, 255, 255, 0.07));
		animation: logo-drift 8s ease-in-out infinite;
	}

	@keyframes logo-drift {
		0%,
		100% {
			transform: translateY(0px) rotate(0deg);
		}
		50% {
			transform: translateY(-10px) rotate(1deg);
		}
	}

	/* ── Other Work List ─────────────────────────────────────────────────────── */
	.other-section {
		padding-top: 80px;
	}

	.other-header {
		padding-bottom: 0;
		margin-bottom: 0;
	}

	.other-eyebrow {
		font-size: 0.68rem;
		letter-spacing: 0.15em;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.28);
	}

	.proj-row {
		display: grid;
		grid-template-columns: 40px 1fr;
		gap: 20px;
		padding: 36px 0;
		border-top: 1px solid rgba(255, 255, 255, 0.06);
		border-radius: 6px;
		transition: background 0.2s;
		animation: row-in linear both;
		animation-timeline: view();
		animation-range: entry 5% cover 25%;
	}

	.proj-row:hover {
		background: rgba(255, 255, 255, 0.02);
	}

	@keyframes row-in {
		from {
			opacity: 0;
			transform: translateY(24px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	.proj-num {
		font-size: 0.62rem;
		letter-spacing: 0.1em;
		color: rgba(255, 255, 255, 0.16);
		padding-top: 5px;
		font-variant-numeric: tabular-nums;
	}

	.proj-body {
		display: flex;
		flex-direction: column;
		gap: 10px;
	}

	.proj-top {
		display: flex;
		align-items: baseline;
		flex-wrap: wrap;
		gap: 10px 12px;
	}

	.proj-logo {
		width: 26px;
		height: 26px;
		flex-shrink: 0;
		align-self: center;
		opacity: 0.9;
	}

	.proj-title {
		font-family: 'Playfair Display', serif;
		font-size: clamp(1.2rem, 2vw, 1.5rem);
		font-weight: 700;
		color: #fff;
		margin: 0;
		line-height: 1.2;
	}

	.proj-subtitle {
		font-size: 0.72rem;
		color: rgba(255, 255, 255, 0.28);
		letter-spacing: 0.03em;
	}

	.proj-tag {
		margin-left: auto;
		font-size: 0.6rem;
		letter-spacing: 0.08em;
		text-transform: uppercase;
		color: rgba(255, 255, 255, 0.22);
		padding: 2px 8px;
		border-radius: 100px;
		border: 1px solid rgba(255, 255, 255, 0.08);
	}

	.proj-desc {
		font-size: 0.875rem;
		color: rgba(255, 255, 255, 0.46);
		margin: 0;
		line-height: 1.7;
		max-width: 640px;
	}

	.proj-links {
		display: flex;
		gap: 8px;
		flex-wrap: wrap;
		padding-top: 2px;
	}

	.proj-btn {
		display: inline-flex;
		align-items: center;
		gap: 5px;
		padding: 6px 15px;
		border-radius: 100px;
		font-size: 0.775rem;
		font-weight: 600;
		cursor: pointer;
		text-decoration: none;
		transition: all 0.15s ease;
		font-family: 'Helvetica Neue', sans-serif;
		background: rgba(255, 255, 255, 0.06);
		color: rgba(255, 255, 255, 0.65);
		border: 1px solid rgba(255, 255, 255, 0.1);
		letter-spacing: 0.01em;
	}

	.proj-btn:hover {
		background: rgba(255, 255, 255, 0.1);
		color: #fff;
		border-color: rgba(255, 255, 255, 0.2);
		transform: translateY(-1px);
	}

	.proj-btn.ghost {
		background: transparent;
		color: rgba(255, 255, 255, 0.38);
		border-color: rgba(255, 255, 255, 0.07);
	}

	.proj-btn.ghost:hover {
		background: rgba(255, 255, 255, 0.05);
		color: rgba(255, 255, 255, 0.75);
		border-color: rgba(255, 255, 255, 0.16);
	}

	/* ── Detail Popovers (shared) ────────────────────────────────────────────── */
	[popover] {
		margin: auto;
		padding: 0;
		border: none;
		border-radius: 16px;
		background: #181818;
		color: #f0f0f0;
		box-shadow: 0 24px 64px rgba(0, 0, 0, 0.85);
		width: 90%;
		max-width: 680px;
		max-height: 80vh;
		overflow-y: auto;
		border: 1px solid rgba(255, 255, 255, 0.08);
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
		background: rgba(0, 0, 0, 0.7);
		backdrop-filter: blur(8px);
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
		padding: 28px 32px 32px;
	}

	.popover-hdr {
		display: flex;
		justify-content: space-between;
		align-items: flex-start;
		margin-bottom: 20px;
		padding-bottom: 16px;
		border-bottom: 1px solid rgba(255, 255, 255, 0.07);
	}

	.popover-hdr h3 {
		margin: 0;
		font-family: 'Playfair Display', serif;
		font-size: 1.5rem;
		color: #fff;
		line-height: 1.2;
	}

	.close-btn {
		background: rgba(255, 255, 255, 0.05);
		border: 1px solid rgba(255, 255, 255, 0.09);
		color: rgba(255, 255, 255, 0.5);
		font-size: 1.2rem;
		line-height: 1;
		cursor: pointer;
		padding: 5px 10px;
		border-radius: 8px;
		transition: all 0.15s;
		flex-shrink: 0;
	}

	.close-btn:hover {
		background: rgba(255, 255, 255, 0.1);
		color: #fff;
	}

	.popover-body {
		white-space: pre-line;
		line-height: 1.85;
		color: rgba(255, 255, 255, 0.65);
		font-size: 0.92rem;
		margin: 0;
	}

	.popover-footer {
		margin-top: 24px;
		padding-top: 16px;
		border-top: 1px solid rgba(255, 255, 255, 0.07);
		display: flex;
		gap: 10px;
	}

	.popover-link {
		font-size: 0.82rem;
		font-weight: 600;
		color: rgba(255, 255, 255, 0.45);
		text-decoration: none;
		padding: 7px 16px;
		border: 1px solid rgba(255, 255, 255, 0.12);
		border-radius: 100px;
		transition: all 0.15s;
		font-family: 'Helvetica Neue', sans-serif;
	}

	.popover-link:hover {
		color: #fff;
		border-color: rgba(255, 255, 255, 0.3);
		background: rgba(255, 255, 255, 0.06);
	}

	/* ── Responsive ──────────────────────────────────────────────────────────── */
	@media (max-width: 720px) {
		.pl-inner {
			grid-template-columns: 1fr;
			gap: 24px;
		}

		.pl-hero {
			min-height: unset;
			padding: 40px 0 60px;
			align-items: flex-start;
		}

		.pl-right {
			order: -1;
			padding: 0;
		}

		.pl-logo {
			max-width: 140px;
		}

		.proj-top {
			flex-direction: column;
			gap: 4px;
		}

		.proj-tag {
			margin-left: 0;
		}
	}
</style>
