<script>
  import { onMount } from "svelte";
  import { slide } from "svelte/transition";

  // define the db
  const db = new Dexie("searchHistory");
  db.version(1).stores({
    searchHistory: "searchTerm",
  });

  let searchTerm = "";
  let searchHistory = [];
  // removed hiding the search history for now.
  // I may include it at a later time if I find a better implementation
  let showHistory = false;

  onMount(async () => {
    await getSearchHistory();
    document.getElementById("search-bar").focus();

    const searchBar = document.getElementById("search-bar");

    searchBar.addEventListener("keyup", (event) => {
      // Number 13 is the "Enter" key on the keyboard
      if (event.keyCode === 13) {
        search();
      }
    });
  });

  const search = async () => {
    if (searchTerm !== "") {
      await addToSearchHistory();
      window.location.assign("https://www.duckduckgo.com?q=" + searchTerm);
    }
  };

  const addToSearchHistory = async () => {
    await db.searchHistory
      .put({
        searchTerm: searchTerm,
      })
      .catch(function (err) {
        alert("Oops: " + err);
      });
  };

  const getSearchHistory = async () => {
    await db.searchHistory
      .each()
      .then(function () {
        // Then when data is stored, read from it
        db.searchHistory.toArray(async (rawHistory) => {
          searchHistory = rawHistory.reverse();
        });
      })
      .catch(function (error) {
        alert("Ooops: " + error);
      });
  };
</script>

<div class="search-bar">
  <input
    type="text"
    title="Search the internet"
    name="search-bar"
    placeholder="DuckDuckGo Search"
    id="search-bar"
    bind:value={searchTerm}
    on:focus={() => {
      showHistory = true;
    }}
    on:focusout={() => {
      showHistory = false;
    }}
  />
  <button on:click={search} class="search-button"> Search </button>
</div>

<div transition:slide class="search-history">
  <h2>Search History</h2>
  {#if searchHistory.length <= 0}
    <p>Nothing here yet 😪</p>
  {:else}
    {#each searchHistory as item, index}
      {#if index < 5}
        <a
          href="https://www.duckduckgo.com?q={item.searchTerm}"
          rel="noopener noreferrer"
        >
          {item.searchTerm}
        </a>
      {/if}
    {/each}
  {/if}
</div>

<style>
  .search-bar {
    display: flex;
    width: 100%;
  }
  .search-bar > *:nth-child(2) {
    margin-left: 5px;
  }
  input {
    width: 100%;
    box-shadow: 0px 0px 5px 0px rgba(0, 0, 0, 0.25);
  }

  .search-history {
    display: flex;
    flex-direction: column;
    background-color: var(--bg-alt);
    color: var(--text-color);
    padding: 10px 0px;
    margin: 10px 0px 5px 0px;
    border-radius: 5px;
    box-shadow: 0px 0px 5px 0px rgba(0, 0, 0, 0.25);
    width: 100%;
  }

  a,
  p,
  h2 {
    padding: 1px 10px;
  }

  a {
    color: #00adb5;
    text-decoration: none;
  }

  a:hover {
    text-decoration: underline;
  }

  button {
    box-shadow: 0px 0px 10px 0px rgba(0, 0, 0, 0.35);
  }

  @media screen and (max-width: 630px) {
    .search-bar {
      display: flex;
      flex-direction: column;
    }
    .search-bar > *:nth-child(2) {
      margin: 5px 0px 0px 0px;
    }
  }
</style>
