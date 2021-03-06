<script>
  import { onMount } from "svelte";
  import { scale } from "svelte/transition";

  import NewBookmark from "./NewBookmark.svelte";

  let bookmarksArray = [];
  let categories = [];
  let newBookmarkFormOpen = false;
  let verboseTabs;

  let bookmarkName = "";
  let url = "";
  let category = "";
  let description = "";

  let updatingBookmark = false;

  // define the db
  const db = new Dexie("bookmarks");
  db.version(1).stores({
    bookmarks: "bookmarkName,url,category,description",
  });

  const settingsDB = new Dexie("settings");
  settingsDB.version(1).stores({
    settings: "settingName,settingValue",
  });

  onMount(async () => {
    await getBookmarks();
    await retrieveCategories();
    await selectFirstCategory();
    await getTabSettings();
  });

  // on new bookmark created reload list of bookmarks
  const handleNewBookmark = async (event) => {
    await getBookmarks();
    await retrieveCategories();
    newBookmarkFormOpen = false;
    selectFirstCategory();
  };

  const handleCloseNewBookmarkForm = (event) => {
    newBookmarkFormOpen = false;
  };

  const getBookmarks = async () => {
    await db.bookmarks
      .each()
      .then(function () {
        // Then when data is stored, read from it
        db.bookmarks.toArray(async (bookmarks) => {
          bookmarksArray = bookmarks;
        });
        return db.bookmarks.get("duckduckgo");
      })
      .then(function (bookmark) {
        // Display the result
        // console.log("Bookmark name: " + bookmark.bookmarkName);
      })
      .catch(function (error) {
        // Finally don't forget to catch any error
        // that could have happened anywhere in the
        // code blocks above.
        alert("Ooops: " + error);
      });
  };

  const deleteBookmark = (bookmarkName) => {
    db.bookmarks
      .where("bookmarkName")
      .equals(bookmarkName)
      .modify((value, ref) => {
        delete ref.value;
      })
      .then(() => {
        handleNewBookmark();
      })
      .catch((err) => {
        alert("There was an error: " + err);
      });
  };

  const openNewBookmarkModal = () => {
    bookmarkName = "";
    url = "";
    category = "";
    description = "";

    updatingBookmark = false;
    newBookmarkFormOpen = !newBookmarkFormOpen;
  };

  const openUpdateBookmark = async (oldBookmarkName) => {
    const bookmark = await db.bookmarks.get({
      bookmarkName: oldBookmarkName,
    });

    bookmarkName = bookmark.bookmarkName;
    url = bookmark.url;
    category = bookmark.category;
    description = bookmark.description;

    newBookmarkFormOpen = !newBookmarkFormOpen;
    updatingBookmark = true;
    console.dir(bookmark);
  };

  const retrieveCategories = () => {
    categories = [];
    bookmarksArray.forEach((bookmark) => {
      if (!categories.includes(bookmark.category)) {
        categories.push(bookmark.category);
      }
    });
  };

  const getTabSettings = async () => {
    await settingsDB.settings
      .get("verboseTabs")
      .then((setting) => {
        verboseTabs = setting.settingValue;
      })
      .catch((err) => {
        console.log(err);
      });
  };

  const showCategory = (category) => {
    const categories = document
      .getElementById("categories")
      .getElementsByClassName("category");

    for (let index = 0; index < categories.length; index++) {
      const categoryDiv = categories[index];
      if (categoryDiv.id === category) {
        categoryDiv.style.display = "flex";
      } else {
        categoryDiv.style.display = "none";
      }
    }
  };

  const selectFirstCategory = () => {
    const categories = document
      .getElementById("categories")
      .getElementsByClassName("category");

    for (let index = 0; index < categories.length; index++) {
      const categoryDiv = categories[index];
      if (index === 0) {
        categoryDiv.style.display = "flex";
      } else {
        categoryDiv.style.display = "none";
      }
    }
  };
</script>

<!-- new bookmark form -->
{#if newBookmarkFormOpen}
  <div transition:scale class="new-bookmark">
    <NewBookmark
      on:newBookmark={handleNewBookmark}
      on:closeNewBookmarkForm={handleCloseNewBookmarkForm}
      {bookmarkName}
      {url}
      {category}
      {description}
      oldBookmarkName={bookmarkName}
      {updatingBookmark}
    />
  </div>
{/if}

<div class="category-buttons">
  {#each categories as category, index}
    {#if verboseTabs}
      <button class="category-button" on:click={showCategory(category)}>
        {category}
      </button>
    {:else}
      <button class="category-button" on:click={showCategory(category)}>
        {index + 1}
      </button>
    {/if}
  {/each}
</div>
<!-- categorize each bookmark into columns -->
<div class="categories" id="categories">
  {#each categories as category}
    <div class="category" transition:scale id={category}>
      <h2>{category}</h2>
      {#each bookmarksArray as bookmark}
        {#if bookmark.category === category}
          <div class="bookmark" transition:scale>
            <a
              href={bookmark.url}
              target="_blank"
              title={bookmark.description}
              rel="noopener noreferrer"
            >
              {#if bookmark.bookmarkName.length > 20}
                {bookmark.bookmarkName.substring(0, 20) + "..."}
              {:else}
                {bookmark.bookmarkName}
              {/if}
            </a>
            <div class="bookmark-button-section">
              <button
                on:click={openUpdateBookmark(bookmark.bookmarkName)}
                class="bookmark-button"
                alt="Edit {bookmark.bookmarkName}"
                title="Edit {bookmark.bookmarkName}">✏️</button
              >
              <button
                on:click={deleteBookmark(bookmark.bookmarkName)}
                class="bookmark-button"
                alt="Delete {bookmark.bookmarkName}"
                title="Delete {bookmark.bookmarkName}">🗑️</button
              >
            </div>
          </div>
        {/if}
      {/each}
    </div>
  {/each}
</div>

<button
  class="new-bookmark-button"
  on:click={() => openNewBookmarkModal("new")}
  title="Create new bookmark">➕</button
>
<div class="debug-section">
  <h2 class="debug-section-header">Debug Section</h2>
  <p class="debug-section-text">
    This is a work in progress. Expect some bugs ☺️
  </p>
  <button on:click={getBookmarks}>Retrieve All Bookmarks</button>
  <button on:click={retrieveCategories}>Categorize</button>
  <p>Updating bookmark: {updatingBookmark}</p>
</div>

<style>
  a {
    color: #00adb5;
    text-decoration: underline;
  }

  .bookmark {
    /* background-color: #393e46;
    padding: 10px; */
    color: #eee;
    margin: 5px 0px;
    border-radius: 5px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 300px;
    /* border: 1px solid red; */
  }

  .bookmark a {
    text-decoration: none;
  }
  .bookmark a:hover,
  .bookmark a:focus {
    text-decoration: underline;
  }

  .new-bookmark {
    width: 600px;
    min-height: 580px;
    margin: 0 auto;
  }

  .new-bookmark-button {
    position: fixed;
    bottom: 30px;
    right: 25px;
    font-size: 32px;
    border-radius: 50px;
    padding: 5px 13px;
    z-index: 1;
    transition: background-color 250ms ease;
    box-shadow: 0px 0px 5px 0px rgba(0, 0, 0, 0.35);
  }

  .new-bookmark-button:hover,
  .new-bookmark-button:focus {
    background-color: #06d6a0;
  }

  .categories {
    width: 100%;
    margin-bottom: 10px;
  }

  .category {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 10px 0px;
    background-color: var(--bg-alt);
    color: var(--text-color);
    border-radius: 5px;
    text-align: center;
    min-width: 100%;
    box-shadow: 0px 0px 15px 0px rgba(0, 0, 0, 0.25);
    min-height: 250px;
  }

  .debug-section {
    color: var(--text-color);
    padding: 15px;
    background-color: var(--bg-alt);
    border-radius: 5px;
    /* margin-top: 150px; */
    width: 300px;
    position: absolute;
    bottom: 20px;
    left: 20px;
    box-shadow: 0px 0px 15px 0px rgba(0, 0, 0, 0.35);
  }

  .debug-section-header {
    padding: 10px 0px 5px 0px;
  }

  .debug-section-text {
    margin-bottom: 10px;
  }

  .bookmark-button-section {
    display: flex;
  }

  .bookmark-button {
    margin: 0px 1px;
    padding: 10px 15px;
    font-size: 18px;
    transition: background-color 250ms ease;
    background-color: transparent;
    color: #eee;
  }

  .bookmark-button:hover {
    background-color: #ff2e63;
  }

  .category-buttons {
    display: flex;
    align-items: center;
    margin: 10px 0px 10px 0px;
    width: 100%;
    overflow-x: auto;
    padding: 5px 0px 10px 0px;
    z-index: 0;
  }

  .category-button {
    padding: 5px 15px;
    height: 100%;
    margin: 0px 3px;
    box-shadow: 0px 0px 5px 0px rgba(0, 0, 0, 0.25);
    white-space: nowrap;
    flex: 1;
  }

  @media screen and (max-width: 630px) {
    .category-buttons {
      align-items: center;
      justify-content: space-between;
      margin: 20px 0px 10px 0px;
      /* border: 1px solid red; */
    }

    .debug-section {
      position: relative;
      padding: 15px;
      width: 90%;
      bottom: 0px;
      left: initial;
      margin-bottom: 10px;
      margin-top: 10px;
    }

    .category {
      width: auto;
    }

    .new-bookmark {
      top: 0;
      right: 0;
      left: 0;
      bottom: 0;
      /* width: 600px; */
      width: 95%;
      min-height: 100vh;
      margin: 0 auto;
      background-color: var(--bg-alt);
      border: 3px solid var(--button-color);
      z-index: 2;
    }

    .category-buttons {
      padding: 5px 0px 15px 0px;
    }
  }
</style>
