<script>
	import ProjectCard from '$lib/components/ProjectCard.svelte';

	// Svelte 5: No need for $state if this array is static constants.
	// If you fetch this data, you would use let projects = $state([...]);
	const projects = [
		{
			title: 'Funda Listing Detail Page',
			shortDescription:
				'A responsive reconstruction of the Funda LDP using pure HTML/CSS/JS and Liquid templating.',
			description: `This project involved a three-week challenge to build a new Listing Detail Page (LDP) for the Dutch real estate platform, Funda. 
            
            The technical implementation leveraged Server-Side Scripting (SSR) and Liquid templating. The server handled data routing and used Liquid for modular page structure. Critically, the SSR process dynamically loaded house images, optimizing delivery using the <picture> element to convert images to the AVIF format where possible.`,
			readmeLink: 'https://github.com/Sebastiaan-hva/proof-of-concept',
			liveLink: 'https://proof-of-concept-1-6ez0.onrender.com/',
			imageSrc: '/funda-mobile.png',
			mediaContent: []
		},
		{
			title: 'Embassy of the Free Mind',
			shortDescription: 'Redesign for the EFM built in SvelteKit with a focus on accessibility.',
			description:
				'Full redesign for the Embassy of the Free Mind using SvelteKit. Focused on component reusability, accessibility (WCAG), and connecting to a headless CMS.',
			readmeLink: '#',
			liveLink: 'https://efm-j2.netlify.app/',
			imageSrc: '/embassy.png',
			mediaContent: []
		},
		{
			title: 'Squad Page Yearbook',
			shortDescription: 'A digital yearbook for our dev squad.',
			description:
				'A collaborative project creating a digital yearbook. Features include custom animations, profile cards, and a shared memory wall.',
			readmeLink: '#',
			liveLink: '#',
			imageSrc: '/book.png',
			mediaContent: []
		},
		{
			title: 'Interactive Dashboard',
			shortDescription: '.',
			description: '.',
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
			<ProjectCard {...project} index={i} />
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
		font-family: 'Playfair Display', serif;
	}

	h2 {
		margin-bottom: 30px;
		font-size: 2.5em;
	}

	.projects-grid {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 30px; /* Adds space between stacked cards */
		max-width: 1200px;
		margin: 0 auto;
	}
</style>
