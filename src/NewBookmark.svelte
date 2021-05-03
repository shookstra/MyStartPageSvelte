<script>
  import { createEventDispatcher } from "svelte";
  import { slide } from "svelte/transition";

  let bookmarkName = "";
  let url = "";
  let category = "";
  let description = "";

  let protocol;
  let subDomain;
  let topLevelDomain;

  let errorMessages = [];

  const dispatch = createEventDispatcher();

  const createNewBookmarkDispatcher = () => {
    dispatch("newBookmark", {
      text: "inserted new bookmark",
    });
  };

  const closeNewBookmarkForm = () => {
    dispatch("closeNewBookmarkForm", {
      text: "close new bookmark form",
    });
  };

  const validateNewBookmark = () => {
    errorMessages = [];
    if (bookmarkName === "")
      errorMessages.push("Please enter a bookmark name.");
    if (url === "") errorMessages.push("Please enter a URL.");
    if (category === "") errorMessages.push("Please enter a category.");
    if (protocol === "") errorMessages.push("Please choose a protocol.");
    if (subDomain === "") errorMessages.push("Please choose a subdomain.");
    if (topLevelDomain === "")
      errorMessages.push("Please choose a top level domain.");

    if (errorMessages.length == 0) {
      createBookmark();
    }
  };

  const createBookmark = async () => {
    const db = new Dexie("bookmarks");
    db.version(1).stores({
      bookmarks: "bookmarkName,url,category,description",
    });

    console.log(protocol + subDomain + url + topLevelDomain);

    db.bookmarks
      .put({
        bookmarkName: bookmarkName,
        url: protocol + subDomain + url + topLevelDomain,
        category: category,
        description: description,
      })
      .then(function () {
        console.log(db.bookmarks);
      })
      .then(function (bookmark) {
        createNewBookmarkDispatcher();
      })
      .catch(function (err) {
        alert("Oops: " + err);
      });
  };

  const createDefaultBookmark = async () => {
    const db = new Dexie("bookmarks");
    db.version(1).stores({
      bookmarks: "bookmarkName,url,category,description",
    });

    db.bookmarks
      .put({
        bookmarkName: "duckduckgo",
        url: "https://www.duckduckgo.com",
        category: "search engines",
        description: "Search the web",
      })
      .then(function (bookmark) {
        createNewBookmarkDispatcher();
      })
      .catch(function (error) {
        alert("Ooops: " + error);
      });
  };
</script>

<h2>New Bookmark</h2>
<button class="new-bookmark-close-button" on:click={closeNewBookmarkForm}
  >×</button
>
<div class="new-bookmark-section">
  <label for="bookmarkName">Bookmark name</label>
  <input type="text" name="bookmarkName" bind:value={bookmarkName} />
</div>
<div class="new-bookmark-section">
  <label for="url">URL</label>
  <div class="url-text-section">
    <select class="url-prefix" bind:value={protocol}>
      <option>https://</option>
      <option>http://</option>
    </select>
    <select class="url-prefix" bind:value={subDomain}>
      <option>www.</option>
    </select>
    <input type="text" name="url" bind:value={url} />
    <select class="url-prefix" bind:value={topLevelDomain}>
      <option>.com</option>
      <option>.org</option>
      <option>.net</option>
    </select>
  </div>
</div>
<div class="new-bookmark-section">
  <label for="category">Category</label>
  <input type="text" name="category" bind:value={category} />
</div>
<div class="new-bookmark-section">
  <label for="description">Description (optional)</label>
  <input type="text" name="description" bind:value={description} />
</div>
{#if errorMessages.length > 0}
  <div transition:slide>
    {#each errorMessages as err}
      <p>{err}</p>
    {/each}
  </div>
{/if}
<button on:click={validateNewBookmark}>Save</button>
<h3>Debug</h3>
<button on:click={createDefaultBookmark}>Create Default</button>

<!-- <p>{protocol + subDomain + url + topLevelDomain}</p> -->
<style>
  h2 {
    font-weight: normal;
    margin-bottom: 25px;
  }

  input[type="text"] {
    width: 100%;
  }

  label {
    padding-bottom: 3px;
  }

  .new-bookmark-section {
    flex: 1;
    width: 100%;
  }

  button {
    margin-top: 15px;
    padding: 10px;
    background-color: #eee;
    font-size: 1.25rem;
    color: #eee;
    outline: 3px solid transparent;
    transition: background-color 250ms ease;
    color: black;
    background-color: #06d6a0;
  }

  button:focus {
    outline: 3px solid #eeeeee;
    background-color: #06d6a0;
  }

  .url-text-section {
    display: flex;
    align-items: stretch;
  }

  .url-prefix {
    display: inline;
    background-color: #06d6a0;
    height: 100%;
    padding: 10px;
    margin: 0px 2px;
    border-radius: 5px;
  }

  .new-bookmark-close-button {
    position: absolute;
    right: 10px;
    top: 0px;
    font-size: 32px;
    padding: 0px 20px 7px 20px;
  }

  .new-bookmark-close-button:focus {
    background-color: #ff2e63;
  }
  @media screen and (max-width: 480px) {
    .url-text-section {
      display: flex;
      flex-direction: column;
    }

    .url-prefix {
      margin: 3px 0px;
    }
  }
</style>
