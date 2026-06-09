<script>
	import { onMount } from 'svelte';
	// prepareWithSegments() does the expensive work once: it segments the text into
	// grapheme clusters via Intl.Segmenter and measures each segment's pixel width on
	// an off-screen canvas (measureText). No DOM is touched. layoutNextLine() is the
	// cheap part — given a starting cursor and a max width it returns ONE line's text
	// + the cursor to continue from, using pure arithmetic on the cached widths.
	//
	// By calling layoutNextLine() per line and feeding a NARROWER width to the lines
	// that overlap the floating element's vertical band, we get real "text wraps
	// around a floating image" behaviour — and because that band is driven by an
	// animated vertical position, the element drifts top-to-bottom and the text
	// genuinely reflows around it. CSS float can't do this (a float pins to the top);
	// pretext computes the whole thing off-screen.
	import { prepareWithSegments, layout, layoutNextLine } from '@chenglou/pretext';

	// Must match the CSS applied to the rendered lines exactly, or pretext's line
	// breaks won't line up with what the browser paints.
	const FONT_SIZE = 19;
	const LINE_HEIGHT = Math.round(FONT_SIZE * 1.75); // px, not a unitless ratio
	const FONT_STRING = `${FONT_SIZE}px "Playfair Display", serif`;

	// ── Stage geometry ───────────────────────────────────────────────────────
	const FLOAT_W = 150; // floating element box
	const FLOAT_H = 150;
	const FLOAT_GAP = 24; // breathing room between text and the float
	const MIN_W = 360;
	const MAX_W = 760;

	// Column width is user-controllable via the slider. narrowW is the width of the
	// lines that sit beside the floating orb (used by the template; the layout pass
	// recomputes the same value locally to stay independent of reactive ordering).
	let columnWidth = 560;
	$: narrowW = Math.max(80, columnWidth - FLOAT_W - FLOAT_GAP);

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
		},
		{
			character: 'Griffith',
			series: 'Berserk',
			text: "Every man has a reason to live. Some need a comrade beside them. Some need money, some chase knowledge, some search for the meaning of their own existence — and for a few, none of that is ever enough. They want something more. My dream is to have a kingdom of my own. I don't yet know its shape, or where its borders will fall, only that I will reach it by my own hand and stand at its summit on my own terms, owing nothing to anyone. And a friend? I'm not certain my idea of one matches yours. To me, a friend is someone who would never depend on another's dream — someone who would not be ruled by anyone, who has their own reason to live, their own dream, and who would risk everything to protect it. If such a person ever stood in my path, even the one closest to me, I would not call him a follower. I would call him my equal, and my rival, and only then could I truly call him my friend."
		},
		{
			character: 'Madara Uchiha',
			series: 'Naruto Shippuden',
			text: "Wake up to reality. Nothing ever goes as planned in this accursed world. The longer you live, the more you realize that the only things that truly exist in this reality are merely pain, suffering, and futility. Listen — everywhere you look in this world, wherever there is light, there will always be shadows to be found. As long as there is a concept of victors, the vanquished will also exist. The selfish desire to protect peace ignites wars, and hatred is born to protect the things we love. These are nexuses, cause and effect bound together with no beginning and no end. The shinobi who took everything from me believed they were righteous. The leaders who sent them believed the same. And in the name of that righteousness, more were buried, and more swore revenge. That is the chain that strangles this world. I will break it — even if I must become the single villain that the whole of history remembers, so that everyone else may finally dream in peace."
		}
	];

	let activeIndex = 0;

	/** @type {any} - opaque handle from prepareWithSegments(); holds cached widths + segments */
	let prepared = null;
	let fontsReady = false;

	// Rendered output
	/** @type {{ text: string, narrow: boolean }[]} */
	let wrappedLines = [];
	let lineCount = 0;
	let narrowCount = 0;
	let textHeight = 0;
	let measureMs = 0;

	// ── Motion + interaction state ─────────────────────────────────────────────
	let floatY = 0; // top of the floating orb (px)
	let baseTextHeight = 0; // full-width text height — stable drift range
	let paused = false; // play/pause the auto-drift
	let dragging = false; // user is dragging the orb
	let _animT = 0;
	const SPEED = 0.006; // slow drift
	/** @type {number | null} */
	let _animFrame = null;
	let _mounted = false;
	// Only re-wrap when the float crosses into a new line band.
	let _bandStart = -1;
	let _bandEnd = -1;
	/** @type {HTMLElement | undefined} */
	let stageEl;
	let _dragOffset = 0;

	async function initialize() {
		await document.fonts.ready;
		fontsReady = true;
		preparePassage();
		_startAnim();
	}

	function preparePassage() {
		if (!fontsReady) return;
		prepared = prepareWithSegments(quotes[activeIndex].text, FONT_STRING);
		_animT = 0;
		floatY = 0;
		applyLayout();
	}

	// Recompute the stable height + the float band from the current floatY and
	// columnWidth, then re-wrap. Runs on quote change, slider change and manual moves.
	function applyLayout() {
		if (!prepared) return;
		baseTextHeight = layout(prepared, columnWidth, LINE_HEIGHT).height;
		const maxY = Math.max(LINE_HEIGHT, baseTextHeight - FLOAT_H);
		floatY = Math.min(Math.max(0, floatY), maxY);
		_bandStart = Math.floor(floatY / LINE_HEIGHT);
		_bandEnd = Math.ceil((floatY + FLOAT_H) / LINE_HEIGHT);
		recomputeLines();
	}

	// Re-wrap whenever the slider changes the column width. (Function call, so Svelte
	// only tracks `prepared` and `columnWidth` as dependencies — no feedback loop.)
	$: if (prepared && columnWidth) applyLayout();

	// Walk the passage one line at a time. Lines whose vertical position overlaps the
	// orb's band get the narrow width; everything else gets the full column width.
	function recomputeLines() {
		if (!prepared) return;
		const narrow_w = Math.max(80, columnWidth - FLOAT_W - FLOAT_GAP);
		const t0 = performance.now();
		/** @type {{ text: string, narrow: boolean }[]} */
		const lines = [];
		let cursor = { segmentIndex: 0, graphemeIndex: 0 };
		let i = 0;
		let narrow = 0;
		while (i < 600) {
			const inBand = i >= _bandStart && i < _bandEnd;
			const w = inBand ? narrow_w : columnWidth;
			const line = layoutNextLine(prepared, cursor, w);
			if (!line) break;
			lines.push({ text: line.text, narrow: inBand });
			if (inBand) narrow++;
			cursor = line.end;
			i++;
		}
		measureMs = performance.now() - t0;
		wrappedLines = lines;
		lineCount = lines.length;
		narrowCount = narrow;
		textHeight = lineCount * LINE_HEIGHT;
	}

	function updateBand() {
		const ns = Math.floor(floatY / LINE_HEIGHT);
		const ne = Math.ceil((floatY + FLOAT_H) / LINE_HEIGHT);
		if (ns !== _bandStart || ne !== _bandEnd) {
			_bandStart = ns;
			_bandEnd = ne;
			recomputeLines();
		}
	}

	function _startAnim() {
		function step() {
			if (!_mounted) return;
			if (!paused && !dragging) {
				_animT += SPEED;
				// Ease 0 → max → 0: drift down the passage, then back up.
				const maxY = Math.max(LINE_HEIGHT, baseTextHeight - FLOAT_H);
				floatY = (0.5 - 0.5 * Math.cos(_animT)) * maxY;
				updateBand();
			}
			_animFrame = requestAnimationFrame(step);
		}
		_animFrame = requestAnimationFrame(step);
	}

	function togglePause() {
		paused = !paused;
		if (!paused) syncAnimToFloat(); // resume drifting from the orb's current spot
	}

	// Invert (0.5 - 0.5·cos t) so play continues from wherever the orb currently sits.
	function syncAnimToFloat() {
		const maxY = Math.max(LINE_HEIGHT, baseTextHeight - FLOAT_H);
		const r = Math.min(1, Math.max(0, floatY / maxY));
		_animT = Math.acos(1 - 2 * r);
	}

	// ── Drag the orb ─────────────────────────────────────────────────────────
	/** @param {PointerEvent} e */
	function onOrbPointerDown(e) {
		dragging = true;
		paused = true;
		const target = /** @type {HTMLElement} */ (e.currentTarget);
		_dragOffset = e.clientY - target.getBoundingClientRect().top;
		target.setPointerCapture(e.pointerId);
		e.preventDefault();
	}

	/** @param {PointerEvent} e */
	function onOrbPointerMove(e) {
		if (!dragging || !stageEl) return;
		const rect = stageEl.getBoundingClientRect();
		const maxY = Math.max(0, stageEl.offsetHeight - FLOAT_H);
		floatY = Math.min(maxY, Math.max(0, e.clientY - rect.top - _dragOffset));
		updateBand();
	}

	/** @param {PointerEvent} e */
	function onOrbPointerUp(e) {
		dragging = false;
		try {
			/** @type {HTMLElement} */ (e.currentTarget).releasePointerCapture(e.pointerId);
		} catch (_) {
			/* ignore */
		}
	}

	/** @param {KeyboardEvent} e */
	function onOrbKeyDown(e) {
		if (e.key === 'ArrowUp' || e.key === 'ArrowDown') {
			paused = true;
			const maxY = Math.max(0, (stageEl?.offsetHeight ?? baseTextHeight) - FLOAT_H);
			const delta = (e.key === 'ArrowUp' ? -1 : 1) * LINE_HEIGHT;
			floatY = Math.min(maxY, Math.max(0, floatY + delta));
			updateBand();
			e.preventDefault();
		} else if (e.key === ' ' || e.key === 'Enter') {
			togglePause();
			e.preventDefault();
		}
	}

	function randomize() {
		let next;
		do {
			next = Math.floor(Math.random() * quotes.length);
		} while (next === activeIndex && quotes.length > 1);
		activeIndex = next;
		preparePassage();
	}

	onMount(() => {
		_mounted = true;
		initialize();
		return () => {
			_mounted = false;
			if (_animFrame) cancelAnimationFrame(_animFrame);
		};
	});
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
				The orb drifts down the passage and the text wraps around it — like a
				floated image, except CSS <em>can't</em> position a float vertically.
				<em>Pretext</em> lays out each line off-screen, feeding a narrower width to
				the lines the orb overlaps. <em>Drag</em> the orb, <em>pause</em> it, or
				<em>resize</em> the column — it re-wraps every frame.
			</p>
		</div>
	</header>

	<main class="stage">
		<div class="passage-title">
			{quotes[activeIndex].character}
			<span class="passage-series">— {quotes[activeIndex].series}</span>
		</div>

		<div class="stage-scroll">
			<div class="text-stage" bind:this={stageEl} style="width: {columnWidth}px; height: {textHeight}px;">
				<!-- live line-number gutter -->
				<div class="line-gutter" aria-hidden="true">
					{#each wrappedLines as _line, i}
						<div class="line-num" style="top: {i * LINE_HEIGHT}px;">{i + 1}</div>
					{/each}
				</div>

				<!-- each line is positioned + sized exactly where pretext computed it -->
				{#each wrappedLines as line, i}
					<div
						class="tline"
						class:narrow={line.narrow}
						style="top: {i * LINE_HEIGHT}px; width: {line.narrow ? narrowW : columnWidth}px; height: {LINE_HEIGHT}px; font-size: {FONT_SIZE}px; line-height: {LINE_HEIGHT}px;"
					>{line.text}</div>
				{/each}

				<!-- the floating orb — drifts top to bottom, draggable, text flows around it -->
				<div
					class="floatbox"
					class:dragging
					style="top: {floatY}px; width: {FLOAT_W}px; height: {FLOAT_H}px;"
					role="slider"
					tabindex="0"
					aria-label="Drag to move the orb through the text"
					aria-valuemin="0"
					aria-valuemax={Math.max(0, Math.round(textHeight - FLOAT_H))}
					aria-valuenow={Math.round(floatY)}
					on:pointerdown={onOrbPointerDown}
					on:pointermove={onOrbPointerMove}
					on:pointerup={onOrbPointerUp}
					on:pointercancel={onOrbPointerUp}
					on:keydown={onOrbKeyDown}
				>
					<div class="floatbox-label">
						<span class="fb-name">{quotes[activeIndex].character}</span>
						<span class="fb-series">{quotes[activeIndex].series}</span>
					</div>
				</div>
			</div>
		</div>

		<div class="controls">
			<button class="ctrl-btn" on:click={togglePause} aria-pressed={paused}>
				{#if paused}
					<svg viewBox="0 0 24 24" width="12" height="12" fill="currentColor" aria-hidden="true"><path d="M8 5v14l11-7z" /></svg>
					Play
				{:else}
					<svg viewBox="0 0 24 24" width="12" height="12" fill="currentColor" aria-hidden="true"><path d="M6 5h4v14H6zM14 5h4v14h-4z" /></svg>
					Pause
				{/if}
			</button>

			<div class="ctrl-slider">
				<span class="ctrl-label">Width</span>
				<input
					type="range"
					min={MIN_W}
					max={MAX_W}
					step="2"
					bind:value={columnWidth}
					aria-label="Column width"
				/>
				<span class="ctrl-val">{columnWidth}px</span>
			</div>

			<span class="ctrl-hint" aria-hidden="true">drag the orb ↕</span>
		</div>

		<div class="metrics">
			<div class="metric">
				<span class="metric-val">{lineCount}</span>
				<span class="metric-label">lines</span>
			</div>
			<span class="metric-dot">·</span>
			<div class="metric">
				<span class="metric-val">{narrowCount}</span>
				<span class="metric-label">wrapped</span>
			</div>
			<span class="metric-dot">·</span>
			<div class="metric">
				<span class="metric-val">{Math.round(textHeight)}</span>
				<span class="metric-label">px tall</span>
			</div>
			<span class="metric-dot">·</span>
			<div class="metric">
				<span class="metric-val">{measureMs < 0.1 ? '< 0.1' : measureMs.toFixed(2)}</span>
				<span class="metric-label">ms / re-wrap</span>
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
		margin-bottom: 48px;
		gap: 40px;
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
		max-width: 360px;
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

	.stage-scroll {
		max-width: 100%;
		display: flex;
		justify-content: center;
	}

	.text-stage {
		position: relative;
		flex-shrink: 0;
	}

	/* Each rendered line. Left-aligned natural width (matches pretext's measure). */
	.tline {
		position: absolute;
		left: 0;
		font-family: 'Playfair Display', serif;
		color: #e0dbd0;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: clip;
	}

	/* ── Line gutter ──────────────────────────────────── */
	.line-gutter {
		position: absolute;
		right: calc(100% + 24px);
		top: 0;
		width: 24px;
		height: 100%;
		pointer-events: none;
	}

	.line-num {
		position: absolute;
		right: 0;
		font-family: 'Poppins', sans-serif;
		font-size: 9px;
		color: rgba(201, 162, 107, 0.28);
		line-height: 1;
		height: 9px;
		transform: translateY(-50%);
		margin-top: 16px;
		text-align: right;
		letter-spacing: 0.04em;
		transition: top 0.25s ease;
	}

	/* ── Floating element ─────────────────────────────── */
	.floatbox {
		position: absolute;
		right: 0;
		display: flex;
		align-items: center;
		justify-content: center;
		background:
			radial-gradient(circle at 32% 28%, rgba(255, 226, 170, 0.95), rgba(201, 162, 107, 0.9) 42%, rgba(120, 88, 40, 0.85) 100%);
		box-shadow:
			0 0 0 1px rgba(201, 162, 107, 0.4),
			0 14px 50px rgba(201, 162, 107, 0.28),
			inset 0 0 30px rgba(255, 240, 210, 0.25);
		/* organic blob that gently morphs as it drifts */
		border-radius: 47% 53% 44% 56% / 55% 48% 52% 45%;
		animation: blob-morph 9s ease-in-out infinite, blob-spin 26s linear infinite;
		/* float moves vertically via inline `top`; transition smooths the per-frame steps */
		transition: top 0.06s linear;
		will-change: top, border-radius, transform;
		cursor: grab;
		touch-action: none; /* let pointer-drag work on touch without scrolling */
	}

	.floatbox:focus-visible {
		outline: 2px solid rgba(255, 240, 210, 0.9);
		outline-offset: 3px;
	}

	.floatbox.dragging {
		cursor: grabbing;
		transition: none; /* follow the pointer with no lag */
	}

	/* Full character name + series, counter-rotating so it stays upright as the
	   blob spins around it. */
	.floatbox-label {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 4px;
		padding: 0 16px;
		text-align: center;
		pointer-events: none;
		user-select: none;
		animation: blob-spin-rev 26s linear infinite;
	}

	.fb-name {
		font-family: 'Playfair Display', serif;
		font-weight: 700;
		font-size: 17px;
		line-height: 1.15;
		color: rgba(38, 24, 6, 0.92);
		text-shadow: 0 1px 1px rgba(255, 240, 210, 0.5);
	}

	.fb-series {
		font-family: 'Poppins', sans-serif;
		font-size: 8px;
		letter-spacing: 0.12em;
		text-transform: uppercase;
		color: rgba(60, 40, 12, 0.72);
	}

	@keyframes blob-morph {
		0%,
		100% {
			border-radius: 47% 53% 44% 56% / 55% 48% 52% 45%;
		}
		33% {
			border-radius: 58% 42% 60% 40% / 42% 58% 42% 58%;
		}
		66% {
			border-radius: 40% 60% 50% 50% / 60% 40% 55% 45%;
		}
	}

	@keyframes blob-spin {
		to {
			transform: rotate(360deg);
		}
	}

	@keyframes blob-spin-rev {
		to {
			transform: rotate(-360deg);
		}
	}

	/* ── Controls ─────────────────────────────────────── */
	.controls {
		display: flex;
		align-items: center;
		gap: 22px;
		flex-wrap: wrap;
		justify-content: center;
	}

	.ctrl-btn {
		display: inline-flex;
		align-items: center;
		gap: 7px;
		background: rgba(201, 162, 107, 0.1);
		border: 1px solid rgba(201, 162, 107, 0.32);
		color: #e2c79a;
		font-family: 'Poppins', sans-serif;
		font-size: 12px;
		letter-spacing: 0.04em;
		padding: 8px 16px;
		border-radius: 6px;
		cursor: pointer;
		transition: background 0.2s, border-color 0.2s, color 0.2s;
		min-width: 88px;
		justify-content: center;
	}

	.ctrl-btn:hover {
		background: rgba(201, 162, 107, 0.2);
		border-color: rgba(201, 162, 107, 0.55);
		color: #fff;
	}

	.ctrl-slider {
		display: flex;
		align-items: center;
		gap: 10px;
		font-family: 'Poppins', sans-serif;
	}

	.ctrl-label {
		font-size: 10px;
		letter-spacing: 0.1em;
		text-transform: uppercase;
		color: #6e6866;
	}

	.ctrl-val {
		font-size: 11px;
		color: #8a8480;
		font-variant-numeric: tabular-nums;
		min-width: 44px;
	}

	.ctrl-slider input[type='range'] {
		-webkit-appearance: none;
		appearance: none;
		width: 160px;
		height: 3px;
		border-radius: 3px;
		background: #25252e;
		outline: none;
		cursor: pointer;
	}

	.ctrl-slider input[type='range']::-webkit-slider-thumb {
		-webkit-appearance: none;
		appearance: none;
		width: 14px;
		height: 14px;
		border-radius: 50%;
		background: #c9a26b;
		border: none;
		cursor: pointer;
		box-shadow: 0 0 0 3px rgba(201, 162, 107, 0.18);
	}

	.ctrl-slider input[type='range']::-moz-range-thumb {
		width: 14px;
		height: 14px;
		border-radius: 50%;
		background: #c9a26b;
		border: none;
		cursor: pointer;
	}

	.ctrl-hint {
		font-size: 10px;
		letter-spacing: 0.1em;
		text-transform: uppercase;
		color: #4a4644;
	}

	/* ── Metrics ─────────────────────────────────────── */
	.metrics {
		display: flex;
		align-items: center;
		gap: 18px;
		padding: 14px 28px;
		border: 1px solid #1a1a22;
		border-radius: 6px;
		background: #0c0c15;
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
		min-width: 1ch;
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

	@media (max-width: 700px) {
		.page {
			padding: 72px 24px 32px;
		}
		.page-header {
			flex-direction: column;
			align-items: flex-start;
			gap: 16px;
		}
		.header-right {
			text-align: left;
		}
		.stage-scroll {
			overflow-x: auto;
			justify-content: flex-start;
			width: 100%;
			padding-left: 36px;
		}
	}
</style>
