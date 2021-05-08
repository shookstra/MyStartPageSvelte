<script>
  import Settings from "./Settings.svelte";
  import Bookmarks from "./Bookmarks.svelte";
  import SearchBar from "./SearchBar.svelte";
  import Clock from "./Clock.svelte";
  import Name from "./Name.svelte";

  import { onMount } from "svelte";

  import { slide } from "svelte/transition";

  let ready = false;
  onMount(() => {
    ready = true;
  });
</script>

{#if !ready}
  <p
    class="loading-overlay"
    id="loading-overlay"
    out:slide={{ duration: 350, delay: 500 }}
  >
    Loading...
  </p>
{/if}

<main>
  <header>
    <h1>MyStartPage</h1>
  </header>
  <div class="info-section">
    <Name />
    <Clock />
  </div>
  <SearchBar />
  <Bookmarks />
  <Settings />
</main>

<style>
  /* dark mode / light mode styles */
  :root {
    --bg-color: #eee;
    --text-color: #222831;
    --accent-color: rgb(250, 250, 250);
    --bg-alt: #eee;
  }

  :global(body) {
    background: var(--bg-color);
  }

  :global(body.dark) {
    --bg-color: #222831;
    --text-color: #eee;
    --bg-alt: #393e46;
    --accent-color: #eee;
  }

  /* end */

  main {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    margin: 0 auto;
    width: 600px;
    min-height: 100vh;
  }

  header {
    color: var(--text-color);
    padding: 5px;
    text-align: center;
  }

  .info-section {
    display: flex;
    justify-content: space-around;
    width: 100%;
  }

  .loading-overlay {
    background-color: #222831;
    color: white;
    position: absolute;
    top: 0;
    right: 0;
    left: 0;
    bottom: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 3rem;
    color: #eee;
    z-index: 3;
  }

  @media screen and (max-width: 630px) {
    main {
      width: 95%;
    }
  }
</style>
