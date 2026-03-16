<script>
	import { onMount } from 'svelte';
	import * as THREE from 'three';

	let canvas;
	let animFrameId;
	let renderer, camera, scene, mesh, clock;
	let currentUniforms = null;
	let showPopover = false;
	let activePreset = 'liquid';

	// ── Shared vertex shader ────────────────────────────────────────────────────
	const vertexShader = `
		varying vec2 vUv;
		void main() {
			vUv = uv;
			gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
		}
	`;

	// ── Shared noise helpers (inlined into each shader that needs them) ─────────
	const NOISE_GLSL = `
		vec3 _mod289v3(vec3 x){return x-floor(x*(1./289.))*289.;}
		vec2 _mod289v2(vec2 x){return x-floor(x*(1./289.))*289.;}
		vec3 _permute(vec3 x){return _mod289v3(((x*34.)+1.)*x);}
		float snoise(vec2 v){
			const vec4 C=vec4(.211324865,.366025404,-.577350270,.024390244);
			vec2 i=floor(v+dot(v,C.yy));
			vec2 x0=v-i+dot(i,C.xx);
			vec2 i1=(x0.x>x0.y)?vec2(1,0):vec2(0,1);
			vec4 x12=x0.xyxy+C.xxzz; x12.xy-=i1;
			i=_mod289v2(i);
			vec3 p=_permute(_permute(i.y+vec3(0,i1.y,1))+i.x+vec3(0,i1.x,1));
			vec3 m=max(.5-vec3(dot(x0,x0),dot(x12.xy,x12.xy),dot(x12.zw,x12.zw)),0.);
			m=m*m; m=m*m;
			vec3 x=2.*fract(p*C.www)-1.;
			vec3 h=abs(x)-.5; vec3 ox=floor(x+.5); vec3 a0=x-ox;
			m*=1.79284291-.85373472*(a0*a0+h*h);
			vec3 g; g.x=a0.x*x0.x+h.x*x0.y; g.yz=a0.yz*x12.xz+h.yz*x12.yw;
			return 130.*dot(m,g);
		}
		float fbm(vec2 p){
			float v=0.,a=.5;
			for(int i=0;i<6;i++){v+=a*snoise(p);p=p*2.03+vec2(3.1,7.4);a*=.5;}
			return v;
		}
		float hash2(vec2 p){return fract(sin(dot(p,vec2(127.1,311.7)))*43758.5453);}
	`;

	// ── 1. LIQUID GRADIENT (original) ──────────────────────────────────────────
	const liquidFrag = `
		uniform float uTime;
		uniform vec2  uResolution;
		varying vec2 vUv;
		#define PI 3.14159265359
		${NOISE_GLSL}

		float liquidNoise(vec2 uv,float t){
			vec2 q=vec2(fbm(uv+t*.14),fbm(uv+vec2(5.2,1.3)+t*.11));
			vec2 r=vec2(fbm(uv+4.*q+vec2(1.7,9.2)+t*.09),fbm(uv+4.*q+vec2(8.3,2.8)+t*.07));
			return fbm(uv+4.*r+t*.05);
		}
		float staticGrain(vec2 uv,float t){
			vec2 p=uv*uResolution*.5;
			vec2 seed=floor(p); seed=fract(seed*vec2(.1031,.1030)); seed+=dot(seed,seed.yx+19.19);
			float n=fract((seed.x+seed.y)*seed.x);
			return sin(t*4.5+n*PI*2.)*.5+.5;
		}
		float snowstorm(vec2 uv,float t){
			vec2 p=uv*uResolution;
			float cs=6.; vec2 cell=floor(p/cs); vec2 local=fract(p/cs);
			float rand=fract(sin(dot(cell,vec2(127.1,311.7)))*43758.5453);
			return length(local-vec2(rand,fract(t*.4+rand)))<.12?1.:0.;
		}
		void main(){
			vec2 uv=vUv;
			vec2 nuv=(uv-.5)*vec2(uResolution.x/uResolution.y,1.)*2.5;
			float t=uTime;
			float liq=liquidNoise(nuv,t);
			float base=.38+liq*.10;
			float sg=staticGrain(uv,t);
			float lum=clamp(base+sg*.10-.5*.10,0.,1.);
			float snowMask=snowstorm(uv,t);
			lum=mix(lum,.58,snowMask);
			lum=lum*.40+.25;
			gl_FragColor=vec4(vec3(lum),1.);
		}
	`;

	// ── 2. CYBERSIGILISM ───────────────────────────────────────────────────────
	const cyberFrag = `
		uniform float uTime;
		uniform vec2  uResolution;
		varying vec2 vUv;
		#define PI  3.14159265359
		#define TAU 6.28318530718
		${NOISE_GLSL}

		vec2 rot2(vec2 p,float a){float s=sin(a),c=cos(a);return vec2(c*p.x-s*p.y,s*p.x+c*p.y);}
		float glow(float d,float w){return exp(-abs(d)/w);}

		// Polygon SDF
		float sdPoly(vec2 p,float r,int n){
			float a=TAU/float(n);
			float bn=mod(atan(p.y,p.x),a)-a*.5;
			return length(p)*cos(bn)-r;
		}
		// Star SDF
		float sdStar(vec2 p,float r,int n,float m){
			float an=TAU/float(n),en=PI/m;
			vec2 acs=vec2(cos(an*.5),sin(an*.5));
			vec2 ecs=vec2(cos(en),sin(en));
			float bn=mod(atan(p.y,p.x)+PI*.5,an)-an*.5;
			p=length(p)*vec2(cos(bn),abs(sin(bn)));
			p-=r*acs;
			p+=ecs*clamp(-dot(p,ecs),0.,r*acs.y/ecs.y);
			return length(p)*sign(p.x);
		}

		void main(){
			vec2 uv=vUv;
			vec2 p=(uv-.5)*vec2(uResolution.x/uResolution.y,1.)*2.2;
			float t=uTime;
			float pulse=.5+.5*sin(t*1.3);

			// Background: deep purple/black with subtle noise vein
			float bgNoise=fbm(p*.8+t*.04);
			vec3 col=vec3(.01,.005,.025)+vec3(.04,.0,.08)*bgNoise;

			// --- Rings ---
			vec3 cyan=vec3(.0,.9,.85);
			vec3 purp=vec3(.55,.0,1.);
			vec3 gold=vec3(1.,.75,.1);

			float r1=abs(length(p)-.88); col+=cyan*glow(r1,.012)*(1.+pulse*.2);
			float r2=abs(length(p)-.60); col+=purp*glow(r2,.010);
			float r3=abs(length(p)-.32); col+=gold*glow(r3,.008);
			float r4=abs(length(p)-.10); col+=cyan*glow(r4,.006);

			// --- Rotating hexagram ---
			vec2 sp1=rot2(p,t*.10);
			float star1=sdStar(sp1,.80,6,3.);
			col+=purp*glow(star1,.013)*.9;

			// --- Counter-rotating pentagon ---
			vec2 sp2=rot2(p,-t*.07);
			float star2=sdStar(sp2,.50,5,2.5);
			col+=cyan*glow(star2,.011)*.75;

			// --- Spinning triangle ---
			vec2 sp3=rot2(p,t*.18);
			float tri=sdPoly(sp3,.30,3);
			col+=gold*glow(tri,.009)*.6;

			// --- Radial spokes (12) ---
			float angle=atan(p.y,p.x);
			float radius=length(p);
			float spokeAngle=mod(angle+t*.04,TAU/12.);
			float spokeW=abs(spokeAngle-TAU/24.)/(TAU/24.);
			float spokeGlow=max(0.,1.-spokeW*18.)*smoothstep(.12,.15,radius)*smoothstep(.90,.85,radius);
			col+=gold*spokeGlow*.35;

			// --- Fine grid ---
			vec2 grid=fract(p*5.)-.5;
			float gridLine=min(abs(grid.x),abs(grid.y));
			col+=vec3(.08,.0,.18)*glow(gridLine-.49,.004);

			// --- Outer glyph circle (dashed) ---
			float outerR=abs(length(p)-1.05);
			float dashAngle=mod(atan(p.y,p.x)*24./TAU,1.);
			float dash=step(.35,dashAngle);
			col+=purp*glow(outerR,.009)*dash*.5;

			// Tone map + vignette
			float vig=1.-smoothstep(.6,1.4,length(p));
			col*=vig;
			col=col/(col+.4);
			gl_FragColor=vec4(col,1.);
		}
	`;

	// ── 3. BREAKCORE ──────────────────────────────────────────────────────────
	const breakFrag = `
		uniform float uTime;
		uniform vec2  uResolution;
		varying vec2 vUv;
		${NOISE_GLSL}

		float hash1(float n){return fract(sin(n)*43758.5453);}

		void main(){
			vec2 uv=vUv;
			float t=uTime;

			// BPM timing (~180 BPM = 3 beats/s)
			float bpm=3.;
			float beat=mod(t*bpm,1.);
			float attack=pow(max(0.,1.-beat*5.),2.); // sharp transient

			// --- Horizontal glitch displacement ---
			float scanSeed=floor(uv.y*90.+floor(t*30.)*17.3);
			float glitch=pow(hash1(scanSeed),4.)*hash1(scanSeed+.5);
			vec2 guv=uv;
			guv.x+=( hash1(scanSeed+.7)-.5)*glitch*.35;

			// --- Corruption blocks ---
			float bt=floor(t*14.);
			vec2 bc=floor(uv*vec2(9.,14.));
			float br=hash2(bc+bt);
			float isBlock=step(.90,br);
			float bshift=(hash2(bc+bt+1.)-.5)*.5;
			guv.x=mix(guv.x,fract(guv.x+bshift),isBlock);

			// --- Noise base (b&w) ---
			vec2 nCoord=floor(guv*uResolution*.5+t*vec2(7.3,3.1));
			float nr=hash2(nCoord);
			float ng=hash2(nCoord+vec2(100.,73.));
			float nb=hash2(nCoord+vec2(200.,37.));
			float bwNoise=nr*.5+ng*.3+nb*.2;
			float lum=step(.52,bwNoise);

			// Scanlines
			float scan=.6+.4*sin(uv.y*uResolution.y*3.14159);
			lum*=mix(1.,scan,.2);

			vec3 col=vec3(lum);

			// --- Chromatic aberration (red bleeds right) ---
			float aberr=.010+attack*.025;
			vec2 rCoord=floor((guv+vec2(aberr,0.))*uResolution*.5+t*vec2(7.3,3.1));
			float rLum=step(.52,hash2(rCoord));
			col.r=mix(col.r,rLum,.45+attack*.3);

			// --- Blue bleeds left ---
			vec2 bCoord=floor((guv-vec2(aberr*.5,0.))*uResolution*.5+t*vec2(7.3,3.1));
			float bLum=step(.52,hash2(bCoord+vec2(200.,37.)));
			col.b=mix(col.b,bLum,.3);

			// --- Beat: red flash & white strobe ---
			float redFlash=attack*step(.4,hash1(floor(t*bpm)+7.));
			col=mix(col,vec3(1.,.0,.02),redFlash*.5);
			float whiteStrobe=pow(max(0.,1.-beat*8.),4.)*.5;
			col=mix(col,vec3(1.),whiteStrobe);

			// --- Invert blocks ---
			vec2 inv=floor(uv*vec2(5.,8.));
			col=mix(col,1.-col,step(.96,hash2(inv+floor(t*10.)+33.)));

			// --- VHS tracking bar ---
			float barY=fract(t*.19);
			float bar=smoothstep(.035,.0,abs(uv.y-barY));
			col=mix(col,vec3(hash2(uv+t),.0,.0),bar*.8);

			gl_FragColor=vec4(col,1.);
		}
	`;

	// ── 4. SEINEN MANGA (Berserk) ──────────────────────────────────────────────
	const mangaFrag = `
		uniform float uTime;
		uniform vec2  uResolution;
		varying vec2 vUv;
		#define PI 3.14159265359
		${NOISE_GLSL}

		float valueNoise(vec2 p){
			vec2 i=floor(p),f=fract(p); f=f*f*(3.-2.*f);
			float a=hash2(i),b=hash2(i+vec2(1,0)),c=hash2(i+vec2(0,1)),d=hash2(i+vec2(1,1));
			return mix(mix(a,b,f.x),mix(c,d,f.x),f.y);
		}
		float vfbm(vec2 p){float v=0.,a=.5;for(int i=0;i<5;i++){v+=a*valueNoise(p);p=p*2.1+vec2(3.1,7.4);a*=.5;}return v;}

		// Crosshatch line: returns 0 (ink) or 1 (paper) for a given angle+freq
		float hatchLine(vec2 p,float angle,float freq){
			float s=sin(angle),c=cos(angle);
			return fract((p.x*c+p.y*s)*freq);
		}

		void main(){
			vec2 uv=vUv;
			vec2 p=uv*vec2(uResolution.x/uResolution.y,1.);
			float t=uTime;

			// Slow drift to give life
			float inkD=vfbm(p*3.2+t*.025);
			float inkD2=vfbm(p*6.5-t*.018+5.3);
			float density=inkD*.65+inkD2*.35; // 0..1, high = dark

			// Paper base
			float paper=1.-density*.45;

			// Pixel-space coords for hatching
			vec2 px=p*uResolution.y*.0028;

			// 4 hatch layers, progressively darker threshold
			float h1=smoothstep(.04,.0,abs(fract(hatchLine(px, PI*.25, 55.)-.5)-.45));
			float h2=smoothstep(.04,.0,abs(fract(hatchLine(px,-PI*.25, 55.)-.5)-.45));
			float h3=smoothstep(.04,.0,abs(fract(hatchLine(px, PI*.50, 72.)-.5)-.45));
			float h4=smoothstep(.04,.0,abs(fract(hatchLine(px, 0.,    72.)-.5)-.45));

			float hatch=h1*step(.62,density)
			           +h2*step(.48,density)*.7
			           +h3*step(.38,density)*.5
			           +h4*step(.32,density)*.35;

			float lum=paper-hatch*.9;

			// --- Speed lines from an off-center focal point ---
			vec2 center=vec2(.5*uResolution.x/uResolution.y,.52);
			vec2 dir=p-center;
			float radius=length(dir);
			float spAngle=fract(atan(dir.y,dir.x)/(2.*PI)*140.+t*.25);
			float spLine=smoothstep(.018,.0,abs(spAngle-.5)*2.-.88);
			spLine*=smoothstep(.12,.35,radius)*smoothstep(.95,.65,radius);
			spLine*=step(.55,density);
			lum-=spLine*.55;

			lum=clamp(lum,0.,1.);

			// High-contrast push (manga look)
			lum=smoothstep(.22,.78,lum);

			// Aged paper / ink colors
			vec3 paperCol=vec3(.94,.91,.86);
			vec3 inkCol=vec3(.04,.04,.06);
			vec3 col=mix(inkCol,paperCol,lum);

			// Red accent in pitch-black ink regions
			float redA=step(.92,density+inkD2*.25)*(1.-lum);
			col=mix(col,vec3(.72,.04,.07),redA*.65);

			// Subtle paper texture noise
			float paperGrain=hash2(uv*uResolution*.5)*.04-.02;
			col=clamp(col+paperGrain,0.,1.);

			gl_FragColor=vec4(col,1.);
		}
	`;

	// ── 5. 80s / 90s ANIME (Akira / Devilman OVA) ─────────────────────────────
	const animeFrag = `
		uniform float uTime;
		uniform vec2  uResolution;
		varying vec2 vUv;
		#define PI 3.14159265359
		${NOISE_GLSL}

		float valueNoise(vec2 p){
			vec2 i=floor(p),f=fract(p); f=f*f*(3.-2.*f);
			return mix(mix(hash2(i),hash2(i+vec2(1,0)),f.x),
			           mix(hash2(i+vec2(0,1)),hash2(i+vec2(1,1)),f.x),f.y);
		}
		float vfbm(vec2 p){float v=0.,a=.5;for(int i=0;i<4;i++){v+=a*valueNoise(p);p=p*2.+vec2(5.2,1.3);a*=.5;}return v;}

		void main(){
			vec2 uv=vUv;
			vec2 p=(uv-.5)*vec2(uResolution.x/uResolution.y,1.);
			float t=uTime;

			// ─ Sky gradient: midnight purple → hot magenta → dark teal ─
			vec3 skyTop   =vec3(.03,.01,.14);
			vec3 skyMid   =vec3(.30,.02,.25);
			vec3 skyBot   =vec3(.00,.12,.22);
			vec3 sky=mix(skyBot,skyMid,smoothstep(.0,.4,uv.y));
			sky=mix(sky,skyTop,smoothstep(.4,1.,uv.y));

			float n =vfbm(uv*2.+t*.05);
			float n2=vfbm(uv*4.-t*.03+3.14);
			vec3 col=sky;

			// ─ Hot-pink skyline bloom ─
			float bloom=exp(-length(p-vec2(.0,.18))*3.)*(.5+n*.3);
			col+=vec3(.85,.08,.52)*bloom*.5;

			// ─ Cyan city-light underlight ─
			float cityGlow=exp(-length(p-vec2(.0,-.5))*2.)*(.4+n2*.3);
			col+=vec3(.0,.58,.82)*cityGlow*.45;

			// ─ Perspective grid (city floor) ─
			vec2 fp=vec2(
				p.x/(uv.y*.9+.08),
				1./(uv.y*.9+.08)-t*.45
			);
			float gx=abs(fract(fp.x)-.5);
			float gy=abs(fract(fp.y)-.5);
			float gridAlpha=max(smoothstep(.49,.50,gx),smoothstep(.49,.50,gy));
			float floorMask=smoothstep(.38,.22,uv.y);
			col+=vec3(.05,.75,.90)*gridAlpha*floorMask*.55;

			// ─ Light beam sweep ─
			float beamAngle=atan(p.y,p.x-sin(t*.25)*.4);
			float beamW=exp(-abs(beamAngle-.4)*8.)*exp(-length(p)*.8);
			col+=vec3(.9,.3,.7)*beamW*.25;

			// ─ CRT scanlines ─
			float scanline=sin(uv.y*uResolution.y*PI);
			col*=mix(1.,.82+.18*scanline,.3);

			// ─ Horizontal tracking glitch lines ─
			float trackSeed=floor(uv.y*220.+floor(t*4.)*31.);
			float track=step(.975,hash2(vec2(trackSeed,floor(t*2.))));
			col=mix(col,vec3(.9,.0,.6)*.45,track*.6);

			// ─ Film grain ─
			float grain=hash2(uv+fract(t*.017))*2.-1.;
			col+=grain*.04;

			// ─ Chromatic aberration (VHS color bleed) ─
			float aberrAmt=sin(t*.6+uv.y*9.)*(.004)+.003;
			col.r+=aberrAmt*8.*(col.r-.5);
			col.b-=aberrAmt*6.*(col.b-.5);

			// ─ Slight cel-shading desaturation ─
			float lumV=dot(col,vec3(.299,.587,.114));
			col=mix(col,vec3(lumV),.12);

			col=clamp(col,0.,1.);
			gl_FragColor=vec4(col,1.);
		}
	`;

	// ── Preset definitions ─────────────────────────────────────────────────────
	const presets = [
		{
			id: 'liquid',
			name: 'Liquid',
			frag: liquidFrag,
			thumb: 'linear-gradient(135deg,#1e1e1e 0%,#6b6b6b 50%,#a6a6a6 100%)'
		},
		{
			id: 'cyber',
			name: 'Sigil',
			frag: cyberFrag,
			thumb: 'radial-gradient(ellipse at center,#8800ff55 0%,#00eadf22 40%,#0a0015 100%),linear-gradient(135deg,#0a0015 0%,#1a004a 100%)'
		},
		{
			id: 'break',
			name: 'Breakcore',
			frag: breakFrag,
			thumb: 'repeating-linear-gradient(0deg,#000 0px,#000 3px,#fff 3px,#fff 4px),linear-gradient(90deg,#ff000033,#00000099)'
		},
		{
			id: 'manga',
			name: 'Berserk',
			frag: mangaFrag,
			thumb: 'repeating-linear-gradient(45deg,#ccc 0px,#ccc 1px,#eee8df 1px,#eee8df 6px)'
		},
		{
			id: 'anime',
			name: 'Akira',
			frag: animeFrag,
			thumb: 'linear-gradient(180deg,#080024 0%,#4d0540 40%,#001f38 100%)'
		}
	];

	// ── Apply preset: swap material, keep same mesh ────────────────────────────
	function applyPreset(preset) {
		activePreset = preset.id;
		if (!mesh) return;
		mesh.material.dispose();
		currentUniforms = {
			uTime:       { value: 0 },
			uResolution: { value: new THREE.Vector2(innerWidth, innerHeight) }
		};
		mesh.material = new THREE.ShaderMaterial({
			uniforms:       currentUniforms,
			vertexShader,
			fragmentShader: preset.frag
		});
	}

	// ── Three.js setup ─────────────────────────────────────────────────────────
	onMount(() => {
		renderer = new THREE.WebGLRenderer({ canvas, antialias: false });
		renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
		renderer.setSize(innerWidth, innerHeight);

		scene  = new THREE.Scene();
		camera = new THREE.PerspectiveCamera(45, innerWidth / innerHeight, 0.1, 1000);
		camera.position.z = 50;
		clock  = new THREE.Clock();

		const getViewSize = () => {
			const h = Math.abs(camera.position.z * Math.tan(camera.fov * Math.PI / 360) * 2);
			return { width: h * camera.aspect, height: h };
		};

		const vs = getViewSize();

		currentUniforms = {
			uTime:       { value: 0 },
			uResolution: { value: new THREE.Vector2(innerWidth, innerHeight) }
		};

		const material = new THREE.ShaderMaterial({
			uniforms:       currentUniforms,
			vertexShader,
			fragmentShader: liquidFrag
		});

		const geometry = new THREE.PlaneGeometry(vs.width, vs.height);
		mesh = new THREE.Mesh(geometry, material);
		scene.add(mesh);

		const TWO_PI_100 = Math.PI * 200;

		const tick = () => {
			animFrameId = requestAnimationFrame(tick);
			if (currentUniforms) {
				currentUniforms.uTime.value =
					(currentUniforms.uTime.value + clock.getDelta()) % TWO_PI_100;
			}
			renderer.render(scene, camera);
		};
		tick();

		const onResize = () => {
			camera.aspect = innerWidth / innerHeight;
			camera.updateProjectionMatrix();
			renderer.setSize(innerWidth, innerHeight);
			if (currentUniforms) currentUniforms.uResolution.value.set(innerWidth, innerHeight);
			const v = getViewSize();
			mesh.geometry.dispose();
			mesh.geometry = new THREE.PlaneGeometry(v.width, v.height);
		};

		window.addEventListener('resize', onResize);

		return () => {
			cancelAnimationFrame(animFrameId);
			window.removeEventListener('resize', onResize);
			renderer.dispose();
		};
	});
</script>

<svelte:head>
	<title>Liquid Gradient</title>
</svelte:head>

<canvas bind:this={canvas}></canvas>

<!-- Wallpaper picker button -->
<button class="picker-btn" on:click={() => (showPopover = !showPopover)} title="Change background">
	<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none"
		stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
		<circle cx="12" cy="12" r="3"/>
		<path d="M12 1v2M12 21v2M4.22 4.22l1.42 1.42M18.36 18.36l1.42 1.42M1 12h2M21 12h2M4.22 19.78l1.42-1.42M18.36 5.64l1.42-1.42"/>
	</svg>
</button>

<!-- Popover — does NOT close when picking a preset, only closes via backdrop or button -->
{#if showPopover}
	<!-- svelte-ignore a11y-click-events-have-key-events a11y-no-static-element-interactions -->
	<div class="backdrop" on:click={() => (showPopover = false)}></div>
	<div class="popover">
		<p class="popover-title">Background</p>
		<div class="grid">
			{#each presets as preset}
				<button
					class="preset-btn"
					class:active={activePreset === preset.id}
					on:click={() => applyPreset(preset)}
					title={preset.name}
				>
					<div class="thumb" style="background:{preset.thumb}"></div>
					<span>{preset.name}</span>
				</button>
			{/each}
		</div>
	</div>
{/if}

<style>
	:global(body) { margin: 0; padding: 0; overflow: hidden; }

	canvas { display: block; width: 100vw; height: 100vh; }

	.picker-btn {
		position: fixed;
		bottom: 24px;
		right: 24px;
		z-index: 100;
		width: 44px;
		height: 44px;
		border-radius: 50%;
		border: none;
		background: rgba(255,255,255,.12);
		backdrop-filter: blur(12px);
		-webkit-backdrop-filter: blur(12px);
		color: rgba(255,255,255,.85);
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		transition: background .2s, transform .15s;
		box-shadow: 0 2px 12px rgba(0,0,0,.35);
	}
	.picker-btn:hover { background: rgba(255,255,255,.22); transform: scale(1.08); }

	.backdrop { position: fixed; inset: 0; z-index: 98; }

	.popover {
		position: fixed;
		bottom: 78px;
		right: 24px;
		z-index: 99;
		background: rgba(14,12,20,.88);
		backdrop-filter: blur(20px);
		-webkit-backdrop-filter: blur(20px);
		border: 1px solid rgba(255,255,255,.1);
		border-radius: 14px;
		padding: 16px;
		box-shadow: 0 8px 32px rgba(0,0,0,.55);
		animation: pop-in .15s ease;
	}

	@keyframes pop-in {
		from { opacity: 0; transform: translateY(8px) scale(.97); }
		to   { opacity: 1; transform: translateY(0)   scale(1);   }
	}

	.popover-title {
		margin: 0 0 12px;
		font-size: 11px;
		font-weight: 600;
		letter-spacing: .1em;
		text-transform: uppercase;
		color: rgba(255,255,255,.4);
		font-family: system-ui, sans-serif;
	}

	.grid {
		display: grid;
		grid-template-columns: repeat(5, 72px);
		gap: 10px;
	}

	.preset-btn {
		background: none;
		border: 2px solid transparent;
		border-radius: 10px;
		padding: 4px;
		cursor: pointer;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 6px;
		transition: border-color .15s, transform .12s;
	}
	.preset-btn:hover { border-color: rgba(255,255,255,.28); transform: scale(1.05); }
	.preset-btn.active { border-color: rgba(255,255,255,.75); }

	.thumb {
		width: 60px;
		height: 42px;
		border-radius: 6px;
		flex-shrink: 0;
	}

	.preset-btn span {
		font-size: 11px;
		color: rgba(255,255,255,.65);
		font-family: system-ui, sans-serif;
		white-space: nowrap;
	}
	.preset-btn.active span { color: rgba(255,255,255,1); }
</style>
