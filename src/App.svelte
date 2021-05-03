<script>
  import Settings from "./Settings.svelte";
  import Bookmarks from "./Bookmarks.svelte";
  import SearchBar from "./SearchBar.svelte";
  import Clock from "./Clock.svelte";
  import Name from "./Name.svelte";

  import { scale } from "svelte/transition";

  let showAttributions = true;
</script>

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
  {#if showAttributions}
    <div class="attributions" transition:scale>
      <p>
        Icon by <a href="https://freeicons.io/profile/3484">BECRIS</a> on
        <a href="https://freeicons.io">freeicons.io</a>
      </p>
      <button
        on:click={() => {
          showAttributions = !showAttributions;
        }}>X</button
      >
    </div>
  {/if}
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

  .attributions {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 15px 30px;
    margin-bottom: 10px;
    background-color: var(--bg-alt);
    color: var(--text-color);
    width: 90%;
    border-radius: 5px;
  }

  .attributions a {
    color: #00adb5;
  }

  @media screen and (max-width: 480px) {
    main {
      width: 95%;
    }

    .attributions {
      padding: 15px;
    }
  }
</style>
