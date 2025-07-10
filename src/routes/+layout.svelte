<script lang="ts">
  import 'normalize.css';
  import './global.css'; // We'll create this for app-wide styles
</script>

<nav>
  <a href="/">Home</a>
  <a href="/places">Places to Visit</a>
</nav>

<main>
  <slot />
</main>

<style>
  :root {
    --primary-color: #3498db;
    --secondary-color: #2ecc71;
    --background-color: #f4f6f8;
    --text-color: #333;
    --light-gray: #ecf0f1;
    --dark-gray: #7f8c8d;
    --font-body: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
    /* Ensure global.css handles base body styles like background-color, color, line-height */
  }

  nav {
    background-color: #fff;
    padding: 1rem 2rem;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    display: flex;
    gap: 1rem;
    align-items: center;
  }

  nav a {
    text-decoration: none;
    color: var(--primary-color);
    font-weight: 500;
    padding: 0.5rem 0;
    position: relative;
  }

  nav a::after {
    content: '';
    position: absolute;
    bottom: 0;
    left: 0;
    width: 0;
    height: 2px;
    background-color: var(--primary-color);
    transition: width 0.3s ease;
  }

  nav a:hover::after,
  nav a.active::after { /* You might need SvelteKit's page store for active class */
    width: 100%;
  }

  main {
    padding: 1rem;
  }
</style>
