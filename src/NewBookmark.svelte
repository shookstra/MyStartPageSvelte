<script>
  import { createEventDispatcher } from "svelte";
  import { slide } from "svelte/transition";

  export let bookmarkName = "";
  export let url = "";
  export let category = "";
  export let description = "";
  export let oldBookmarkName = "";
  export let updatingBookmark;

  let showURLHint = false;

  let errorMessages = [];

  const db = new Dexie("bookmarks");
  db.version(1).stores({
    bookmarks: "bookmarkName,url,category,description",
  });

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

    if (errorMessages.length == 0) {
      if (updatingBookmark) {
        updateBookmark();
      } else {
        createBookmark();
      }
    }
  };

  const createBookmark = async () => {
    db.bookmarks
      .put({
        bookmarkName: bookmarkName,
        url: url,
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

  const updateBookmark = async () => {
    await db.bookmarks
      .update(oldBookmarkName, {
        bookmarkName: bookmarkName,
        url: url,
        category: category,
        description: description,
      })
      .then(() => {
        console.log(`${oldBookmarkName} was renamed to ${bookmarkName}`);
        createNewBookmarkDispatcher();
      })
      .catch((err) => {
        alert("There was an error: " + err);
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
  <input
    type="text"
    name="bookmarkName"
    bind:value={bookmarkName}
    placeholder="Google"
  />
</div>
<div class="new-bookmark-section">
  <label for="url">URL</label>
  {#if showURLHint}
    <p class="url-tip" transition:slide>
      Make sure to include the protocol (e.g. "https" and "http"), subdomain
      (e.g. "www"), and top level domain (e.g. ".com" or ".org")
    </p>
  {/if}
  <input
    type="text"
    name="url"
    bind:value={url}
    on:focus={() => {
      showURLHint = true;
    }}
    on:focusout={() => {
      showURLHint = false;
    }}
    placeholder="https://www.google.com"
  />
</div>
<div class="new-bookmark-section">
  <label for="category">Category</label>
  <input
    type="text"
    name="category"
    bind:value={category}
    placeholder="Search Engines"
  />
</div>
<div class="new-bookmark-section">
  <label for="description">Description (optional)</label>
  <input
    type="text"
    name="description"
    bind:value={description}
    placeholder="Search the web"
  />
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

<style>
  h2 {
    font-weight: normal;
    margin-bottom: 25px;
  }

  input[type="text"] {
    width: 100%;
  }

  label {
    padding: 3px 0px;
    font-weight: bold;
    font-size: 1.15rem;
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

  .url-tip {
    background-color: var(--bg-color);
    margin: 0px 5px 10px 0px;
    padding: 10px;
    border-radius: 5px;
    box-shadow: 0px 0px 5px 0px rgba(0, 0, 0, 0.25);
  }
  @media screen and (max-width: 630px) {
  }
</style>
