<script>
  import { onMount } from "svelte";
  import { scale } from "svelte/transition";

  import NewBookmark from "./NewBookmark.svelte";

  let bookmarksArray = [];
  let categories = [];
  let newBookmarkFormOpen = false;

  // define the db
  const db = new Dexie("bookmarks");
  db.version(1).stores({
    bookmarks: "bookmarkName,url,category,description",
  });

  onMount(async () => {
    await getBookmarks();
    await retrieveCategories();
    selectFirstCategory();
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

  const retrieveCategories = () => {
    categories = [];
    bookmarksArray.forEach((bookmark) => {
      if (!categories.includes(bookmark.category)) {
        categories.push(bookmark.category);
      }
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
    />
  </div>
{/if}

<div class="category-buttons">
  {#each categories as category, index}
    <button class="category-button" on:click={showCategory(category)}
      >{index + 1}</button
    >
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
            <a href={bookmark.url} target="_blank" title={bookmark.description}
              >{bookmark.bookmarkName}</a
            >
            <div class="bookmark-button-section">
              <button
                on:click={deleteBookmark(bookmark.bookmarkName)}
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
  on:click={() => {
    newBookmarkFormOpen = !newBookmarkFormOpen;
  }}
  title="Create new bookmark">➕</button
>
<div class="debug-section">
  <h2 class="debug-section-header">Debug Section</h2>
  <p class="debug-section-text">
    This is a work in progress. Expect some bugs ☺️
  </p>
  <button on:click={getBookmarks}>Retrieve All Bookmarks</button>
  <button on:click={retrieveCategories}>Categorize</button>
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
    z-index: 0;
    transition: background-color 250ms ease;
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
    z-index: -1;
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
    align-items: center;
    justify-content: space-between;
    margin: 20px 0px 10px 0px;
    /* border: 1px solid red; */
  }
  .category-button {
    padding: 5px 15px;
    height: 100%;
    margin: 0px 3px;
  }

  @media screen and (max-width: 480px) {
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
    }
  }
</style>
