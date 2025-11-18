<script>
	import ProjectCard from '$lib/components/ProjectCard.svelte';

	// Project data array - Updated with placeholders to test staggered loading
	const projects = [
		{
			title: 'Funda Listing Detail Page',
			description: `This project involved a three-week challenge to build a new Listing Detail Page (LDP) for the Dutch real estate platform, Funda, using pure HTML, CSS, and JavaScript. 
            The primary goal was to reconstruct the LDP from a Figma design, focusing on a responsive, mobile-first layout. Accessibility was a core requirement, enforced through semantic HTML.
            
            The technical implementation leveraged Server-Side Scripting (SSR) and Liquid templating. The server handled data routing and used Liquid for modular page structure. Critically, the SSR process dynamically loaded house images, optimizing delivery using the <picture> element to convert images to the AVIF format where possible, thus enhancing performance and preventing layout shift. 
            Client-side features included a CSS-based carousel for image navigation and custom mobile menus.`,
			readmeLink: 'https://github.com/Sebastiaan-hva/proof-of-concept?tab=readme-ov-file',
			liveLink: 'https://proof-of-concept-1-6ez0.onrender.com/house/40/',
			imageSrc: '/funda-mobile.png',
			mediaContent: []
		},
		{
			title: 'Embassy of the Free Mind',
			description: 'Redesign for the EFM built in SvelteKit.',
			readmeLink:
				'https://github.com/fdnd-agency/embassyofthefreemind/tree/dev?tab=readme-ov-file#sv',
			liveLink: 'https://efm-j2.netlify.app/',
			imageSrc: 'embassy.png',
			mediaContent: []
		},
		{
			title: 'Squad page year book',
			description: 'A Book.',
			readmeLink: 'https://s13-squad-page.netlify.app/',
			liveLink: 'https://efm-j2.netlify.app/',
			imageSrc: '/book.png',
			mediaContent: []
		},
		{
			title: 'AAAAAAAAAAAAAAA',
			description: 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA.',
			readmeLink: '#',
			liveLink: '#',
			imageSrc: '/Joker.png',
			mediaContent: []
		},
		{
			title: 'BBBBBBBB',
			description: 'BBBBBBBBBBBBBBBBBBBBBBBBBB.',
			readmeLink: '#',
			liveLink: '#',
			imageSrc: '/Joker.png',
			mediaContent: []
		}
	];
</script>

<div class="projects-page-container">
	<h2>My Projects</h2>

	<div class="projects-grid">
		{#each projects as project, i (project.title)}
			<ProjectCard
				title={project.title}
				description={project.description}
				readmeLink={project.readmeLink}
				liveLink={project.liveLink}
				imageSrc={project.imageSrc}
				mediaContent={project.mediaContent}
				index={i}
			/>
		{/each}
	</div>
</div>

<style>
	.projects-page-container {
		padding: 40px 20px;
		background-color: #222;
		min-height: 100vh;
		color: white;
		text-align: center;
		/* Using a global font for consistency */
		font-family: 'Playfair Display', serif;
	}

	h2 {
		margin-bottom: 30px;
		font-size: 2.5em;
	}

	.projects-grid {
		/* Layout ensures cards are stacked vertically and centered */
		display: flex;
		flex-direction: column;
		align-items: center;
	}
</style>
