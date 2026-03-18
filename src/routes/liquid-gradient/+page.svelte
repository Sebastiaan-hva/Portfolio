<script>
	import { onMount } from 'svelte';
	import * as THREE from 'three';

	let canvas;
	let animFrameId;
	let renderer, camera, scene, mesh, clock;
	let currentUniforms = null;
	let showPopover = false;
	let activePreset = 'liquid';

	// ── Playback controls ──────────────────────────────────────────────────────
	let speed      = 1;
	let isPaused   = false;
	let isReversed = false;
	let controlValues = {};

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
		uniform float uContrast;
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
			lum=mix(0.5,lum,uContrast);
			gl_FragColor=vec4(vec3(lum),1.);
		}
	`;

	// ── 2. CYBERSIGILISM ───────────────────────────────────────────────────────
	// Castlevania castle silhouette drawn with breathdvinity blade/arc language.
	// EVA-02 color scheme: near-black bg → deep crimson → amber → warm white.
	const cyberFrag = `
		uniform float uTime;
		uniform vec2  uResolution;
		uniform float uGlow;
		uniform float uScale;
		varying vec2 vUv;
		#define PI  3.14159265359
		#define TAU 6.28318530718

		// Tapered blade — leaf shape, zero width at both endpoints.
		// s = edge softness: ~.004 crisp, ~.055 for glow pass.
		float blade(vec2 p,vec2 a,vec2 b,float mw,float s){
			vec2 pa=p-a,ba=b-a;
			float t=clamp(dot(pa,ba)/dot(ba,ba),0.,1.);
			float w=mw*sin(t*PI);
			return smoothstep(w+s,.0,length(pa-ba*t));
		}

		// Tapered arc blade — scythe shape, zero width at both angle endpoints.
		float tArc(vec2 p,vec2 cen,float r,float a0,float a1,float mw,float s){
			vec2 q=p-cen;
			float d=abs(length(q)-r);
			float ang=atan(q.y,q.x);
			float da=mod(ang-a0,TAU);
			float span=mod(a1-a0,TAU);
			float t=clamp(da/span,0.,1.);
			float w=mw*sin(t*PI);
			return smoothstep(w+s,.0,d)*step(da,span);
		}

		// Spike — tapers from sp at base to zero at tip.
		float spike(vec2 p,vec2 tip,vec2 base,float sp,float s){
			vec2 pa=p-base,ba=tip-base;
			float t=clamp(dot(pa,ba)/dot(ba,ba),0.,1.);
			float w=sp*(1.-t);
			return smoothstep(w+s,.0,length(pa-ba*t));
		}

		void main(){
			vec2 uv=vUv;
			float ar=uResolution.x/uResolution.y;
			vec2 p=(uv-.5)*vec2(ar,1.)*2./uScale;
			float tm=uTime;
			float pulse=.5+.5*sin(tm*.40);

			float m=0.; // crisp mask
			float g=0.; // glow mask (wider softness)
			float cs=.004;  // crisp softness
			float gs=.060;  // glow softness
			vec2 mp=vec2(abs(p.x),p.y); // bilateral symmetry

			// ── CASTLE KEEP / CRUCIFIX SPINE ──────────────────────────────────
			m+=blade(p, vec2(0.,-.93),vec2(0.,.93),.023,cs);
			g+=blade(p, vec2(0.,-.93),vec2(0.,.93),.023,gs);
			// Upper horizontal arm (cross bar)
			m+=blade(p, vec2(-.76,.30),vec2(.76,.30),.019,cs);
			g+=blade(p, vec2(-.76,.30),vec2(.76,.30),.019,gs);
			// Lower reinforcing bar
			m+=blade(p, vec2(-.43,.07),vec2(.43,.07),.013,cs);

			// ── GOTHIC ARCHES (pointed entry arch) ────────────────────────────
			// Inner arch arc — each side curves up to a point at the center top
			m+=tArc(mp,vec2(.22,.30),.30,PI*.58,PI*1.06,.015,cs);
			g+=tArc(mp,vec2(.22,.30),.30,PI*.58,PI*1.06,.015,gs);
			// Outer arch — wider sweep
			m+=tArc(mp,vec2(.36,.30),.46,PI*.60,PI*1.04,.012,cs);

			// ── FLYING BUTTRESSES ─────────────────────────────────────────────
			// Primary buttress — large arc from body sweeping down-out
			m+=tArc(mp,vec2(.60,.00),.48,PI*.58,PI*1.16,.013,cs);
			g+=tArc(mp,vec2(.60,.00),.48,PI*.58,PI*1.16,.013,gs);
			// Counter-buttress
			m+=tArc(mp,vec2(.44,.16),.34,PI*.60,PI*1.20,.011,cs);
			// Outer buttress extension
			m+=tArc(mp,vec2(.70,-.18),.36,PI*.66,PI*1.12,.010,cs);

			// ── LANCET WINDOWS (pointed Gothic openings) ──────────────────────
			// Lower lancet: two opposing arcs suggest a pointed window
			m+=tArc(mp,vec2(.14,.04),.18,PI*.60,PI*1.00,.011,cs);
			m+=tArc(mp,vec2(.14,.50),.18,PI*.02,PI*.42,.011,cs);

			// ── SIDE WING BLADES (organic body curves) ────────────────────────
			m+=blade(mp,vec2(.06,.52),vec2(.60,.63),.013,cs);
			m+=blade(mp,vec2(.06,.16),vec2(.58,.06),.012,cs);
			m+=blade(mp,vec2(.08,-.10),vec2(.52,-.35),.011,cs);

			// ── OUTER SHOULDER / GARGOYLE ARCS ────────────────────────────────
			m+=tArc(mp,vec2(0.,0.),.72,-PI*.24,PI*.24,.016,cs);
			g+=tArc(mp,vec2(0.,0.),.72,-PI*.24,PI*.24,.016,gs);
			m+=blade(mp,vec2(.52,.46),vec2(.84,.24),.011,cs);
			m+=blade(mp,vec2(.50,-.34),vec2(.82,-.14),.010,cs);

			// ── BATTLEMENTS / MERLONS (top spikes) ────────────────────────────
			m+=spike(p,  vec2(.000, .98),vec2(.000, .62),.019,cs);
			m+=spike(mp, vec2(.130, .94),vec2(.090, .58),.015,cs);
			m+=spike(mp, vec2(.285, .87),vec2(.185, .56),.012,cs);
			m+=spike(mp, vec2(.455, .74),vec2(.285, .52),.010,cs);
			m+=spike(mp, vec2(.625, .58),vec2(.365, .46),.008,cs);
			// Battlement teeth between main spires
			m+=spike(mp, vec2(.068, .95),vec2(.068, .76),.009,cs);
			m+=spike(mp, vec2(.205, .88),vec2(.165, .70),.008,cs);

			// ── DUNGEON ROOT SPIKES (bottom) ──────────────────────────────────
			m+=spike(p,  vec2(.000,-.98),vec2(.000,-.55),.019,cs);
			m+=spike(mp, vec2(.155,-.92),vec2(.100,-.53),.015,cs);
			m+=spike(mp, vec2(.325,-.82),vec2(.205,-.49),.012,cs);
			m+=spike(mp, vec2(.505,-.66),vec2(.305,-.43),.010,cs);

			// ── ROSE WINDOW / CENTER CROSS ────────────────────────────────────
			m+=tArc(mp,vec2(0.,.30),.11,-PI*.5,PI*.5,.009,cs);
			m+=blade(p, vec2(0.,.20),vec2(0.,.40),.008,cs);
			m+=blade(p, vec2(-.11,.30),vec2(.11,.30),.008,cs);

			m=clamp(m,0.,1.);
			g=clamp(g*uGlow,0.,1.);

			// ── EVA-02 COLOR SCHEME ───────────────────────────────────────────
			vec3 bg       =vec3(.022,.006,.008); // near-black, red tint
			vec3 crimson  =vec3(.800,.078,.052); // deep EVA-02 red
			vec3 amber    =vec3(1.00,.450,.065); // EVA-02 orange trim
			vec3 warmWhite=vec3(1.00,.840,.640); // warm highlight

			vec3 col=bg;
			// Red glow halo behind the glyph
			col+=vec3(.55,.030,.010)*g;
			// Base crimson layer
			col=mix(col,crimson,m*.88);
			// Orange accent on brighter areas
			col=mix(col,amber,m*m*(.52+pulse*.28));
			// Warm-white highlight on peak intensity
			col=mix(col,warmWhite,m*m*m*(.32+pulse*.32));
			// Subtle central red radial glow
			float dist=length(p);
			col+=vec3(.50,.030,.010)*exp(-dist*1.5)*.10*(.7+pulse*.3);
			// Edge vignette
			col*=1.-smoothstep(.88,1.22,dist);

			gl_FragColor=vec4(col,1.);
		}
	`;

	// ── 3. BREAKCORE ──────────────────────────────────────────────────────────
	const breakFrag = `
		uniform float uTime;
		uniform vec2  uResolution;
		uniform float uBPM;
		varying vec2 vUv;
		${NOISE_GLSL}

		float hash1(float n){return fract(sin(n)*43758.5453);}

		void main(){
			vec2 uv=vUv;
			float t=uTime;

			// BPM timing — controlled by uBPM uniform
			float bpm=uBPM;
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
		uniform float uInk;
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
			float density=(inkD*.65+inkD2*.35)*uInk; // 0..1, high = dark

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
		uniform float uBloom;
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
			col+=vec3(.85,.08,.52)*bloom*.5*uBloom;

			// ─ Cyan city-light underlight ─
			float cityGlow=exp(-length(p-vec2(.0,-.5))*2.)*(.4+n2*.3);
			col+=vec3(.0,.58,.82)*cityGlow*.45*uBloom;

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
			id: 'liquid', name: 'Liquid', frag: liquidFrag,
			thumb: 'linear-gradient(135deg,#1e1e1e 0%,#6b6b6b 50%,#a6a6a6 100%)',
			controls: [
				{ id: 'uContrast', label: 'Contrast', min: 0.3, max: 2.0, step: 0.05, default: 1.0 }
			]
		},
		{
			id: 'cyber', name: 'Sigil', frag: cyberFrag,
			thumb: 'linear-gradient(#141020,#141020) center/100% 100%,linear-gradient(to top,transparent 5%,#aaa 14% 16%,transparent 20%,transparent 80%,#aaa 84% 86%,transparent 95%) center/100% 100%',
			controls: [
				{ id: 'uGlow',  label: 'Glow', min: 0,   max: 1.5, step: 0.05, default: 0.35 },
				{ id: 'uScale', label: 'Zoom', min: 0.5, max: 2.0, step: 0.05, default: 1.0  }
			]
		},
		{
			id: 'break', name: 'Breakcore', frag: breakFrag,
			thumb: 'repeating-linear-gradient(0deg,#000 0px,#000 3px,#fff 3px,#fff 4px),linear-gradient(90deg,#ff000033,#00000099)',
			controls: [
				{ id: 'uBPM', label: 'BPM', min: 0.5, max: 8.0, step: 0.5, default: 3.0 }
			]
		},
		{
			id: 'manga', name: 'Berserk', frag: mangaFrag,
			thumb: 'repeating-linear-gradient(45deg,#ccc 0px,#ccc 1px,#eee8df 1px,#eee8df 6px)',
			controls: [
				{ id: 'uInk', label: 'Ink', min: 0.4, max: 1.8, step: 0.05, default: 1.0 }
			]
		},
		{
			id: 'anime', name: 'Akira', frag: animeFrag,
			thumb: 'linear-gradient(180deg,#080024 0%,#4d0540 40%,#001f38 100%)',
			controls: [
				{ id: 'uBloom', label: 'Bloom', min: 0, max: 2.5, step: 0.1, default: 1.0 }
			]
		}
	];

	// initialise all control values from defaults
	for (const p of presets) {
		if (p.controls) for (const c of p.controls) controlValues[c.id] = c.default;
	}

	$: activePresetObj = presets.find(p => p.id === activePreset);

	// ── Build uniforms for a preset (preserves uTime, reads controlValues) ─────
	function buildUniforms(preset) {
		const u = {
			uTime:       { value: currentUniforms?.uTime?.value ?? 0 },
			uResolution: { value: new THREE.Vector2(innerWidth, innerHeight) }
		};
		if (preset.controls) {
			for (const c of preset.controls) u[c.id] = { value: controlValues[c.id] ?? c.default };
		}
		return u;
	}

	// ── Apply preset: swap material, keep same mesh ────────────────────────────
	function applyPreset(preset) {
		activePreset = preset.id;
		if (!mesh) return;
		mesh.material.dispose();
		currentUniforms = buildUniforms(preset);
		mesh.material = new THREE.ShaderMaterial({
			uniforms:       currentUniforms,
			vertexShader,
			fragmentShader: preset.frag
		});
	}

	// ── Live-update a single uniform from a control slider ────────────────────
	function onControlChange(id, val) {
		controlValues[id] = val;
		controlValues = controlValues; // trigger Svelte reactivity
		if (currentUniforms?.[id]) currentUniforms[id].value = val;
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

		currentUniforms = buildUniforms(presets[0]);

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
			const rawDelta = clock.getDelta();
			if (currentUniforms && !isPaused) {
				const d = rawDelta * speed * (isReversed ? -1 : 1);
				currentUniforms.uTime.value =
					((currentUniforms.uTime.value + d) % TWO_PI_100 + TWO_PI_100) % TWO_PI_100;
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

		<hr class="section-divider" />

		<p class="popover-title">Playback</p>
		<div class="ctrl-row">
			<button class="icon-btn" class:active={isPaused} on:click={() => (isPaused = !isPaused)} title={isPaused ? 'Play' : 'Pause'}>
				{#if isPaused}
					<svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="currentColor"><polygon points="5,3 19,12 5,21"/></svg>
				{:else}
					<svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="currentColor"><rect x="6" y="4" width="4" height="16"/><rect x="14" y="4" width="4" height="16"/></svg>
				{/if}
			</button>
			<button class="icon-btn" class:active={isReversed} on:click={() => (isReversed = !isReversed)} title="Reverse time">
				<svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="1 4 1 10 7 10"/><path d="M3.51 15a9 9 0 1 0 .49-3.41"/></svg>
			</button>
			<div class="slider-field">
				<span class="ctrl-label">Speed</span>
				<input type="range" min="0.1" max="3" step="0.05" bind:value={speed} />
				<span class="ctrl-val">{speed.toFixed(1)}×</span>
			</div>
		</div>

		{#if activePresetObj?.controls?.length}
			<hr class="section-divider" />
			<p class="popover-title">{activePresetObj.name}</p>
			{#each activePresetObj.controls as ctrl}
				<div class="slider-field">
					<span class="ctrl-label">{ctrl.label}</span>
					<input
						type="range"
						min={ctrl.min} max={ctrl.max} step={ctrl.step}
						value={controlValues[ctrl.id] ?? ctrl.default}
						on:input={(e) => onControlChange(ctrl.id, +e.target.value)}
					/>
					<span class="ctrl-val">{(controlValues[ctrl.id] ?? ctrl.default).toFixed(2)}</span>
				</div>
			{/each}
		{/if}
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

	.section-divider {
		border: none;
		border-top: 1px solid rgba(255,255,255,.08);
		margin: 12px 0;
	}

	.ctrl-row {
		display: flex;
		align-items: center;
		gap: 8px;
	}

	.icon-btn {
		width: 28px;
		height: 28px;
		flex-shrink: 0;
		border-radius: 6px;
		border: 1px solid rgba(255,255,255,.15);
		background: rgba(255,255,255,.07);
		color: rgba(255,255,255,.60);
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		transition: background .15s, color .15s, border-color .15s;
	}
	.icon-btn:hover { background: rgba(255,255,255,.15); color: rgba(255,255,255,.9); }
	.icon-btn.active {
		background: rgba(255,255,255,.18);
		color: rgba(255,255,255,1);
		border-color: rgba(255,255,255,.45);
	}

	.slider-field {
		display: flex;
		align-items: center;
		gap: 8px;
		width: 100%;
		margin-top: 6px;
	}
	.ctrl-row .slider-field { margin-top: 0; }

	.ctrl-label {
		font-size: 11px;
		color: rgba(255,255,255,.40);
		font-family: system-ui, sans-serif;
		white-space: nowrap;
		min-width: 52px;
	}

	.slider-field input[type=range] {
		flex: 1;
		accent-color: rgba(255,255,255,.75);
		cursor: pointer;
		height: 4px;
		min-width: 0;
	}

	.ctrl-val {
		font-size: 11px;
		color: rgba(255,255,255,.50);
		font-family: system-ui, monospace;
		min-width: 38px;
		text-align: right;
	}
</style>
