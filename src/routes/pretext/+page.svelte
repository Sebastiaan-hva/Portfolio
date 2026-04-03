<script>
	import { onMount } from 'svelte';
	// prepare() does the expensive work once: segments the text into grapheme clusters
	// using Intl.Segmenter, then measures each segment's pixel width via an off-screen
	// canvas (CanvasRenderingContext2D.measureText). No DOM elements are touched.
	// layout() is the cheap part: pure arithmetic on those cached widths.
	import { prepare, layout } from '@chenglou/pretext';

	// These three values must match the actual CSS applied to .text-column exactly.
	// If they drift, pretext's line count won't match what the browser renders.
	const FONT_SIZE = 19;
	const LINE_HEIGHT = Math.round(FONT_SIZE * 1.75); // px, not a unitless ratio
	const FONT_STRING = `${FONT_SIZE}px "Playfair Display", serif`; // CSS font shorthand

	const quotes = [
		{
			character: 'Gintoki Sakata',
			series: 'Gintama',
			text: "There's no short-cut to becoming strong. But I'll tell you what does make you strong — it's the people standing beside you. The moment you swear to yourself that you'll protect someone, that you'll fight for them, that you'd throw your life away before letting them get hurt — that's when you stop being a child and start being something more. Not a hero. Just someone who made a choice and lived up to it."
		},
		{
			character: 'Itachi Uchiha',
			series: 'Naruto Shippuden',
			text: "People live their lives bound by what they accept as correct and true. That's how they define reality. But what does it mean to be correct or true? Those are merely vague concepts. Their reality may all be a mirage. Can we consider them to simply be living in their own world, shaped by their own beliefs? You and I live in completely different worlds. But perhaps there exists a place where those worlds intersect — not through agreement, but through understanding."
		},
		{
			character: 'Roy Mustang',
			series: 'Fullmetal Alchemist: Brotherhood',
			text: "They call me the Hero of Ishval. I've been given medals for what I did. But a medal is just a coin with a pretty picture on it. Somewhere, a woman is crying for her son. Somewhere, children have lost their father. The State Alchemist program gave us a license to kill — and we used it. I used it. If I'm to spend the rest of my life making that right, then so be it. That's not punishment. That's the only thing I have left that means anything."
		},
		{
			character: 'Nagato',
			series: 'Naruto Shippuden',
			text: "Do you understand pain a little now? If you don't share someone's pain, you can never understand them. But just because you understand them doesn't mean you can come to an agreement. That's the truth. I see now that the circumstances of one's birth are irrelevant. It is what you do with the gift of life that determines who you are. And every person who has ever lived has been shaped by loss — because pain is the only thing that truly belongs to us."
		},
		{
			character: 'Kamina',
			series: 'Tengen Toppa Gurren Lagann',
			text: "Who the hell do you think I am? I'm Kamina of Team Gurren! Listen up, Simon — never forget. Just believe in yourself. Not in the you who believes in me. Not the me who believes in you. Believe in the you who believes in yourself. A man fights when he has something to protect. A real man's spirit never breaks — it bends, it bends until it touches the ground, and then it launches you straight through the heavens."
		},
		{
			character: 'Erwin Smith',
			series: 'Attack on Titan',
			text: "We're all going to die. Our only options are how and why. I believe that what we're doing now is a meaningful choice. The difference between us and the titans is that we choose. We choose to be afraid. We choose to fight. We choose to sacrifice. If we're wrong — if there's nothing beyond these walls worth dying for — then we die for nothing. But if we're right, even for a moment, then we leave behind something real. Something that says we were here and we fought and we were human."
		},
		{
			character: 'Edward Elric',
			series: 'Fullmetal Alchemist: Brotherhood',
			text: "A lesson without pain is meaningless. That's because no one can gain without sacrificing something. But by enduring that pain and overcoming it, he shall obtain a powerful, unmatched heart. Alchemy's first law of Equivalent Exchange demands a sacrifice of equal value. But human souls don't work that way — they grow, they stretch, they outlast the pain that shaped them. That's the one thing alchemy could never teach me, and the one thing I had to learn on my own."
		},
		{
			character: 'Spike Spiegel',
			series: 'Cowboy Bebop',
			text: "I'm not going there to die. I'm going to find out if I'm really alive. I've been in a dream, and that dream has been going on for so long I can't tell what's real anymore. Three years ago I died. Everything that happened after that isn't really life — it's just the leftover. But you know what? Even leftovers can mean something. Even a man who's already dead can have one last thing worth doing. I just have to go find out what it is."
		},
		{
			character: 'Meruem',
			series: 'Hunter × Hunter',
			text: "I was born in darkness. I knew nothing but power, and power told me everything I needed to know. The weak exist to serve the strong — that was the world I understood. And then I met someone who had nothing — no strength, no power, no value by any measure I had ever used — and she beat me. Every single time. What does that mean? What is it that I was missing all along? I want to know. For the first time in my existence, there is something in this world I want more than victory."
		},
		{
			character: 'Guts',
			series: 'Berserk',
			text: "In this world, is the destiny of mankind controlled by some transcendental entity or law? Is it like the hand of God hovering above? At least it is true that man has no control, even over his own will. You act. You fight. You bleed. And somewhere behind it all, something watches and laughs — or maybe it doesn't care at all. Maybe that's the worst part. That nothing is watching. That you're responsible for everything, and there's no one to blame but yourself, and no one to save you but yourself."
		}
	];

	let activeIndex = 0;
	let columnWidth = 440; // controlled by the range slider; drives both CSS width and layout()
	const MIN_WIDTH = 180;
	const MAX_WIDTH = 660;

	/** @type {any} - opaque object returned by prepare(); holds cached segment widths */
	let prepared = null;
	let lineCount = 0; // how many lines pretext computed at the current columnWidth
	let textHeight = 0; // lineCount × LINE_HEIGHT in px
	let measureMs = 0; // how long layout() took — usually sub-millisecond
	let fontsReady = false; // gate so we don't measure before the font is loaded
	let metricsFlash = false; // triggers the brief border-color flash on the metrics box

	async function initialize() {
		// Wait for all fonts declared in <svelte:head> to finish loading.
		// This is critical: prepare() calls canvas.measureText() internally, and if
		// the custom font hasn't loaded yet, the browser falls back to a system font,
		// producing wrong widths that won't match the rendered text.
		await document.fonts.ready;
		fontsReady = true;
		preparePassage();
	}

	function preparePassage() {
		if (!fontsReady) return;
		// prepare() is the slow step (~5–20 ms for a paragraph).
		// It walks every grapheme cluster in the string, measures its pixel width on
		// an OffscreenCanvas using the exact same font string we pass in, and stores
		// those widths in a compact array. It also notes where word/line-break
		// opportunities exist (spaces, hyphens, etc.) via the Unicode line-breaking
		// algorithm so layout() knows where it's allowed to wrap.
		prepared = prepare(quotes[activeIndex].text, FONT_STRING);
		runLayout();
	}

	function runLayout() {
		if (!prepared) return;
		const t0 = performance.now();
		// layout() is the fast step — pure JS arithmetic, no DOM or canvas reads.
		// It simulates a greedy line-wrap: starting from the first segment, it
		// accumulates widths until the running total would exceed maxWidth, then
		// "wraps" by incrementing lineCount and resetting the accumulator.
		// height = lineCount × lineHeight. That's it — no layout engine involved.
		const result = layout(prepared, columnWidth, LINE_HEIGHT);
		measureMs = performance.now() - t0;
		lineCount = result.lineCount;
		textHeight = result.height; // lineCount * LINE_HEIGHT
		triggerFlash();
	}

	function triggerFlash() {
		// Double rAF trick: setting false then true in the same tick wouldn't toggle
		// the CSS class because Svelte batches DOM updates. The rAF defers the
		// re-assignment to the next paint frame so the class change is actually seen.
		metricsFlash = false;
		requestAnimationFrame(() => {
			metricsFlash = true;
			setTimeout(() => (metricsFlash = false), 400);
		});
	}

	function randomize() {
		// Pick any index except the current one so the quote always visibly changes.
		let next;
		do {
			next = Math.floor(Math.random() * quotes.length);
		} while (next === activeIndex && quotes.length > 1);
		activeIndex = next;
		preparePassage(); // re-run the expensive prepare() for the new text
	}

	onMount(initialize);

	// Reactive statement: re-runs layout() whenever columnWidth changes (e.g. slider
	// drag). prepare() is NOT re-called here — the cached widths are still valid
	// because the text and font haven't changed, only the available width.
	$: if (prepared && columnWidth) runLayout();
</script>

<svelte:head>
	<title>Pretext</title>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="" />
	<link
		href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital@0;1&family=Poppins:wght@400;600&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<div class="page">
	<header class="page-header">
		<div class="header-left">
			<span class="eyebrow">@chenglou/pretext</span>
			<h1 class="page-title">Pretext</h1>
			<p class="page-subtitle">text layout · without the DOM</p>
		</div>
		<div class="header-right">
			<p class="header-desc">
				Drag the slider. <em>Pretext</em> measures line count and height
				without touching the DOM — no reflow, no flicker.
			</p>
		</div>
	</header>

	<main class="stage">
		<div class="passage-title">
			{quotes[activeIndex].character}
			<span class="passage-series">— {quotes[activeIndex].series}</span>
		</div>

		<div class="column-wrap">
			<div class="column-outer" style="width: {columnWidth}px;">
				<div
					class="text-column"
					style="width: {columnWidth}px; font-size: {FONT_SIZE}px; line-height: {LINE_HEIGHT}px;"
				>
					<span class="drop-cap">{quotes[activeIndex].text[0]}</span>{quotes[activeIndex].text.slice(1)}
				</div>

				<div class="ruler" style="height: {textHeight}px;">
					{#each { length: lineCount } as _, i}
						<div class="tick" style="top: {i * LINE_HEIGHT + LINE_HEIGHT - 1}px;"></div>
					{/each}
					<div class="ruler-bracket ruler-bracket--top"></div>
					<div class="ruler-bracket ruler-bracket--bot" style="top: {textHeight}px;"></div>
					<div class="ruler-label" style="top: {textHeight / 2}px;">
						{Math.round(textHeight)}px
					</div>
				</div>
			</div>
		</div>

		<div class="controls-row">
			<span class="ctrl-label">Width</span>
			<input
				class="width-slider"
				type="range"
				min={MIN_WIDTH}
				max={MAX_WIDTH}
				bind:value={columnWidth}
			/>
			<span class="ctrl-val">{columnWidth}px</span>
		</div>

		<div class="metrics" class:flash={metricsFlash}>
			<div class="metric">
				<span class="metric-val">{lineCount}</span>
				<span class="metric-label">lines</span>
			</div>
			<span class="metric-dot">·</span>
			<div class="metric">
				<span class="metric-val">{Math.round(textHeight)}</span>
				<span class="metric-label">px</span>
			</div>
			<span class="metric-dot">·</span>
			<div class="metric">
				<span class="metric-val">{measureMs < 0.1 ? '< 0.1' : measureMs.toFixed(2)}</span>
				<span class="metric-label">ms measured</span>
			</div>
		</div>
	</main>

	<footer class="page-footer">
		<button class="randomize-btn" on:click={randomize}>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				width="13"
				height="13"
				viewBox="0 0 24 24"
				fill="none"
				stroke="currentColor"
				stroke-width="2.5"
				stroke-linecap="round"
				stroke-linejoin="round"
			>
				<polyline points="16 3 21 3 21 8" />
				<line x1="4" y1="20" x2="21" y2="3" />
				<polyline points="21 16 21 21 16 21" />
				<line x1="15" y1="15" x2="21" y2="21" />
			</svg>
			Randomize
		</button>

		<a
			class="lib-link"
			href="https://chenglou.me/pretext/"
			target="_blank"
			rel="noreferrer noopener"
		>
			chenglou.me/pretext ↗
		</a>
	</footer>
</div>

<style>
	:global(body) {
		margin: 0;
		padding: 0;
		background: #08080f;
	}

	.page {
		min-height: 100vh;
		background: #08080f;
		color: #e0dbd0;
		font-family: 'Poppins', sans-serif;
		display: flex;
		flex-direction: column;
		padding: 80px 60px 40px;
		box-sizing: border-box;
	}

	/* ── Header ─────────────────────────────────────────── */
	.page-header {
		display: flex;
		justify-content: space-between;
		align-items: flex-end;
		margin-bottom: 60px;
	}

	.eyebrow {
		font-size: 11px;
		letter-spacing: 0.12em;
		text-transform: uppercase;
		color: #c9a26b;
		display: block;
		margin-bottom: 6px;
		font-family: 'Poppins', sans-serif;
	}

	.page-title {
		font-family: 'Playfair Display', serif;
		font-size: clamp(48px, 7vw, 96px);
		font-weight: 700;
		margin: 0 0 4px;
		line-height: 1;
		color: #e8e3d8;
		letter-spacing: -0.02em;
	}

	.page-subtitle {
		margin: 0;
		font-size: 13px;
		color: #6e6866;
		letter-spacing: 0.08em;
	}

	.header-right {
		max-width: 320px;
		text-align: right;
	}

	.header-desc {
		margin: 0;
		font-size: 13px;
		line-height: 1.65;
		color: #6e6866;
	}

	.header-desc em {
		color: #c9a26b;
		font-style: normal;
	}

	/* ── Stage ─────────────────────────────────────────── */
	.stage {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 32px;
	}

	.passage-title {
		font-family: 'Poppins', sans-serif;
		font-size: 12px;
		letter-spacing: 0.1em;
		color: #6e6866;
		text-align: center;
	}

	.passage-series {
		color: #3e3c3a;
	}

	.column-wrap {
		display: flex;
		justify-content: center;
	}

	.column-outer {
		position: relative;
	}

	.text-column {
		font-family: 'Playfair Display', serif;
		color: #e0dbd0;
		word-break: normal;
		overflow-wrap: break-word;
		hyphens: auto;
		text-align: justify;
		position: relative;
	}

	.drop-cap {
		float: left;
		font-size: 4.6em;
		line-height: 0.78;
		margin: 0.05em 0.08em 0 0;
		color: #c9a26b;
		font-family: 'Playfair Display', serif;
		font-weight: 700;
	}

	/* ── Ruler ─────────────────────────────────────────── */
	.ruler {
		position: absolute;
		left: calc(100% + 14px);
		top: 0;
		width: 20px;
	}

	.tick {
		position: absolute;
		left: 0;
		width: 6px;
		height: 1px;
		background: #c9a26b;
		opacity: 0.35;
	}

	.ruler-bracket {
		position: absolute;
		left: 0;
		width: 12px;
		height: 1px;
		background: #c9a26b;
		opacity: 0.6;
	}

	.ruler-bracket--top {
		top: 0;
	}

	.ruler-label {
		position: absolute;
		left: 16px;
		transform: translateY(-50%);
		font-size: 10px;
		color: #c9a26b;
		opacity: 0.7;
		white-space: nowrap;
		font-family: 'Poppins', sans-serif;
		letter-spacing: 0.04em;
	}

	/* ── Controls ─────────────────────────────────────── */
	.controls-row {
		display: flex;
		align-items: center;
		gap: 14px;
		font-size: 12px;
		color: #6e6866;
	}

	.ctrl-label {
		letter-spacing: 0.08em;
		text-transform: uppercase;
		font-size: 10px;
	}

	.width-slider {
		-webkit-appearance: none;
		appearance: none;
		width: 240px;
		height: 2px;
		background: #222228;
		outline: none;
		border-radius: 1px;
		cursor: pointer;
	}

	.width-slider::-webkit-slider-thumb {
		-webkit-appearance: none;
		appearance: none;
		width: 14px;
		height: 14px;
		border-radius: 50%;
		background: #c9a26b;
		cursor: pointer;
		transition: transform 0.15s;
	}

	.width-slider::-webkit-slider-thumb:hover {
		transform: scale(1.25);
	}

	.width-slider::-moz-range-thumb {
		width: 14px;
		height: 14px;
		border: none;
		border-radius: 50%;
		background: #c9a26b;
		cursor: pointer;
	}

	.ctrl-val {
		font-size: 11px;
		color: #6e6866;
		min-width: 48px;
		font-variant-numeric: tabular-nums;
	}

	/* ── Metrics ─────────────────────────────────────── */
	.metrics {
		display: flex;
		align-items: center;
		gap: 20px;
		padding: 14px 28px;
		border: 1px solid #1a1a22;
		border-radius: 6px;
		background: #0c0c15;
		transition: border-color 0.3s, background 0.3s;
	}

	.metrics.flash {
		border-color: #c9a26b44;
		background: #0f0d0a;
	}

	.metric {
		display: flex;
		align-items: baseline;
		gap: 5px;
	}

	.metric-val {
		font-size: 22px;
		font-weight: 600;
		color: #e0dbd0;
		font-variant-numeric: tabular-nums;
		font-family: 'Poppins', sans-serif;
		line-height: 1;
	}

	.metric-label {
		font-size: 10px;
		letter-spacing: 0.08em;
		text-transform: uppercase;
		color: #6e6866;
	}

	.metric-dot {
		color: #2a2a32;
		font-size: 18px;
	}

	/* ── Footer ─────────────────────────────────────── */
	.page-footer {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-top: 48px;
		padding-top: 24px;
		border-top: 1px solid #14141c;
	}

	.randomize-btn {
		display: flex;
		align-items: center;
		gap: 8px;
		background: none;
		border: 1px solid #222228;
		color: #6e6866;
		font-family: 'Poppins', sans-serif;
		font-size: 12px;
		letter-spacing: 0.06em;
		padding: 8px 18px;
		border-radius: 4px;
		cursor: pointer;
		transition: border-color 0.2s, color 0.2s;
	}

	.randomize-btn:hover {
		border-color: #c9a26b66;
		color: #c9a26b;
	}

	.lib-link {
		font-size: 11px;
		letter-spacing: 0.08em;
		color: #3e3c3a;
		text-decoration: none;
		transition: color 0.2s;
	}

	.lib-link:hover {
		color: #c9a26b;
	}
</style>
