<script>
  import Settings from "./Settings.svelte";
  import Bookmarks from "./Bookmarks.svelte";
  import SearchBar from "./SearchBar.svelte";
  import Clock from "./Clock.svelte";
  import Name from "./Name.svelte";
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
</main>

<style>
  /* dark mode / light mode styles */
  :root {
    --bg-color: #eee;
    --text-color: #222831;
    --accent-color: #ccc;
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
    /* place-items: center; */
    flex-direction: column;
    justify-content: center;
    align-items: center;
    /* padding: 0px 30px; */
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
    /* align-items: center; */
    justify-content: space-around;
    width: 100%;
    /* color: #eee; */
  }

  @media screen and (max-width: 480px) {
    main {
      /* border: 1px solid red; */
      width: 95%;
    }
  }
</style>
