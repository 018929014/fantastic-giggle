<svelte:head>
  <title>Travel Wishlist Home</title>
  <meta name="description" content="Welcome to your personal travel wishlist." />
</svelte:head>

<div class="home-container">
  <h1>Welcome to Your Travel Wishlist</h1>
  <p>
    Keep track of all the amazing places you want to visit around the globe.
  </p>
  <p>
    <a href="/places" class="cta-button">View Your Places</a>
  </p>
</div>

<style>
  .home-container {
    text-align: center;
    padding-top: 4rem;
    padding-bottom: 4rem;
  }

  h1 {
    font-size: 2.5rem;
    color: #2c3e50;
    margin-bottom: 1rem;
    font-weight: 300;
  }

  p {
    font-size: 1.2rem;
    color: #555;
    margin-bottom: 2rem;
    max-width: 600px;
    margin-left: auto;
    margin-right: auto;
  }

  .cta-button {
    display: inline-block;
    padding: 0.8rem 1.8rem;
    background-color: var(--primary-color, #3498db);
    color: white;
    border-radius: 5px;
    text-decoration: none;
    font-size: 1.1rem;
    font-weight: 500;
    transition: background-color 0.2s ease, transform 0.2s ease;
  }

  .cta-button:hover {
    background-color: #2980b9; /* Darken primary color */
    text-decoration: none;
    transform: translateY(-2px);
  }
</style>
