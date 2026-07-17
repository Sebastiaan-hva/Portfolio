# Sebastiaan Hagoort — Personal Portfolio

An interactive, premium portfolio website showcasing personal and professional frontend engineering work. Built with **SvelteKit (Svelte 5)**, featuring state-of-the-art vanilla CSS interactions, scroll-driven animations, custom WebGL gradients, and offline PWA capability.

---

## 🎨 Key Features & Design Aesthetics

- **Interactive Accordion Landing Page**: Features dynamic hover panels representing different sections. Employs sophisticated micro-animations (e.g., custom hail-fall canvas, ripple loops, sunset gradient shifting) and system-aware dark/light theme integration.
- **Scroll-Driven Animations**: Leverages CSS `animation-timeline: scroll()` and `view()` for fluid page transitions, fading/dissolving text, and scrolling progress indicators without JS scroll-listener overhead.
- **Fluid Section Navigation (macOS-Style Dock)**: The "About Me" page includes a custom-engineered bottom dock with hover-based magnification (using CSS `flex-basis` and sibling selectors), active section tracking via `IntersectionObserver`, and smooth-scroll anchors.
- **Popover Detail Modals**: Seamless project detail views powered by the native HTML `popover` API, paired with backdrop blur filters and scale transitions.
- **WebGL Liquid Gradients**: Custom high-performance shaders utilizing `three.js` to render immersive fluid-like background graphics.

---

## 📁 Projects Featured

### 1. PrimedLifter (Lead Project)

- **Role**: Creator & Developer (Personal Project)
- **Description**: A full-featured coaching web application for powerlifters that replaces the fragmented Google Sheets + Drive workflow. Offers automated program calculations, RPE tracking (Tuchscherer chart), athlete dashboard, competition/meet planning, video review tools, and offline support.
- **Stack**: SvelteKit (Svelte 5), Supabase (Postgres + Auth), Vite, Service Worker / PWA.
- **Live App**: [primedlifter.app](https://primedlifter.app)

### 2. IntoGolf

- **Role**: Frontend Intern
- **Description**: Shipped features across three parts of a SaaS platform used by golf professionals and academies throughout the Netherlands:
  1.  _Marketing Site_: Static pages using Astro, React islands, and Keystatic CMS.
  2.  _V3 Core Coaching App_: Lesson scheduling flows and calendar views using Vue & Quasar.
  3.  _V3 Pro App_: Academies CRM, canvas swing-annotation tools, Mollie payments, offline support, and push notifications.
- **Stack**: Vue/Quasar, Firebase, Astro, React, Mollie, Service Worker / PWA.
- **Live Site**: [intogolf.nl](https://intogolf.nl)

### 3. Funda Listing Detail Page

- **Role**: Developer (School Project)
- **Description**: Rebuilt Funda's detail view in 3 weeks adhering strictly to progressive enhancement (Semantic HTML first, CSS layout second, JS strictly last). Implemented AVIF image optimization using responsive `<picture>` tags.
- **Stack**: Node.js, Liquid templating, Semantic HTML, CSS, Vanilla JS.
- **Live Demo**: [POC on Render](https://proof-of-concept-1-6ez0.onrender.com/)
- **GitHub**: [proof-of-concept](https://github.com/sebas-hagoort/proof-of-concept)

### 4. Embassy of the Free Mind

- **Role**: Developer (School Project)
- **Description**: Full website redesign for an Amsterdam foundation focusing on component reusability, headless CMS integration, and WCAG accessibility compliance.
- **Stack**: SvelteKit, Headless CMS.
- **Live Site**: [Embassy of the Free Mind Dev Portal](https://embassyofthefreemind.dev.fdnd.nl/)

### 5. Squad Page Yearbook

- **Role**: Collaborator (School Project)
- **Description**: Digital yearbook built collaboratively with a school squad featuring custom profile cards, individual section animations, and a shared memory wall.
- **Stack**: HTML, CSS, JS, Git.

---

## 🛠️ Technology Stack & Tooling

### Core Frontend

- **Framework**: [SvelteKit](https://svelte.dev/docs/kit) (using **Svelte 5** runes/syntax)
- **3D / Graphics**: [Three.js](https://threejs.org/) for high-fidelity WebGL fluid rendering
- **Rich Typography**: Playfair Display, Helvetica Neue (via Google Fonts & system defaults)
- **Layout & Styling**: Vanilla CSS (CSS Variables, Container Queries `@container`, CSS Grid, Flexbox)

### Tooling & Quality Assurance

- **Bundler & Dev Server**: [Vite](https://vite.dev/)
- **Component Documentation**: [Storybook](https://storybook.js.org/) for isolated UI building
- **End-to-End Testing**: [Playwright](https://playwright.dev/)
- **Linter & Formatter**: [ESLint](https://eslint.org/) and [Prettier](https://prettier.io/)

---

## 🚀 Getting Started

### Prerequisites

Make sure you have Node.js installed.

### 1. Installation

Clone the repository and install the dependencies:

```sh
npm install
```

### 2. Development

Start the local development server:

```sh
npm run dev

# Or start the server and automatically open it in your default browser
npm run dev -- --open
```

### 3. Production Build

Build the optimized production site and preview it locally:

```sh
# Build the application
npm run build

# Preview the built application
npm run preview
```

### 4. Storybook

Run Storybook to view and test UI components in isolation:

```sh
npm run storybook
```

### 5. Running Tests

Execute Playwright end-to-end integration tests:

```sh
npm run test
```

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).
