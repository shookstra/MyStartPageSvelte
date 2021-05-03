<script>
  import { slide } from "svelte/transition";
  import { onMount } from "svelte";

  let showSettings = false;
  let name = "";
  let twelveHour = false;
  let darkMode;
  let verboseTabs;

  const db = new Dexie("settings");
  db.version(1).stores({
    settings: "settingName,settingValue",
  });

  onMount(async () => {
    await getName();
    await getTimeFormat();
    await getTheme();
    await getTabSettings();

    if (darkMode) {
      toggleDarkMode();
    }
  });

  const toggleDarkMode = () => {
    window.document.body.classList.toggle("dark");
  };

  const saveName = async () => {
    if (name) {
      await db.settings
        .put({
          settingName: "name",
          settingValue: name,
        })
        .then((setting) => {
          console.log(setting);
          location.reload();
        })
        .catch((err) => {
          alert("Oops: " + err);
        });
    }
  };

  const getName = async () => {
    await db.settings
      .get("name")
      .then((setting) => {
        name = setting.settingValue;
      })
      .catch((err) => {
        console.log(err);
      });
  };

  const getTimeFormat = async () => {
    await db.settings
      .get("timeFormat")
      .then((setting) => {
        if (setting.settingValue === "24-hour") {
          twelveHour = false;
        } else {
          twelveHour = true;
        }
      })
      .catch((err) => {
        console.log(err);
      });
  };

  const saveTimeFormat = async () => {
    const timeFormat = twelveHour === true ? "12-hour" : "24-hour";

    if (twelveHour !== null) {
      await db.settings
        .put({
          settingName: "timeFormat",
          settingValue: timeFormat,
        })
        .then((setting) => {
          location.reload();
        })
        .catch((err) => {
          alert("Oops: " + err);
        });
    }
  };

  const saveTheme = async () => {
    await db.settings
      .put({
        settingName: "darkMode",
        settingValue: darkMode,
      })
      .then((setting) => {
        console.log(setting);
        location.reload();
      })
      .catch((err) => {
        alert("Oops: " + err);
      });
  };

  const getTheme = async () => {
    await db.settings
      .get("darkMode")
      .then((setting) => {
        darkMode = setting.settingValue;
      })
      .catch((err) => {
        console.log(err);
      });
  };

  const saveTabSettings = async () => {
    await db.settings
      .put({
        settingName: "verboseTabs",
        settingValue: verboseTabs,
      })
      .then((setting) => {
        console.log(setting);
        location.reload();
      })
      .catch((err) => {
        alert("Oops: " + err);
      });
  };

  const getTabSettings = async () => {
    await db.settings
      .get("verboseTabs")
      .then((setting) => {
        console.log(setting.settingValue);
        verboseTabs = setting.settingValue;
      })
      .catch((err) => {
        console.log(err);
      });
  };

  const saveSettings = () => {
    saveName();
    saveTimeFormat();
    saveTheme();
    saveTabSettings();
  };
</script>

<button
  class="hamburger-button"
  on:click={() => {
    showSettings = !showSettings;
  }}
>
  <span>---</span>
  <span>---</span>
  <span>---</span>
</button>

{#if showSettings}
  <div transition:slide={{ y: 0 }} class="settings">
    <h1>Settings</h1>
    <button
      on:click={() => {
        showSettings = !showSettings;
      }}
      class="sidebar-close"
      >×
    </button>

    <!-- name settings -->
    <div class="settings-field">
      <label for="name">Name</label>
      <input
        type="text"
        bind:value={name}
        name="name"
        class="settings-text"
        placeholder="Enter your name (optional)"
      />
    </div>

    <!-- clock settings -->
    <div class="settings-field checkbox-field">
      <label for="12-hour-clock">12-Hour Clock</label>
      <input type="checkbox" name="12-hour-clock" bind:checked={twelveHour} />
    </div>

    <!-- dark mode -->
    <div class="settings-field checkbox-field">
      <label for="dark-mode">Dark Mode</label>
      <input
        type="checkbox"
        name="dark-mode"
        bind:checked={darkMode}
        on:change={toggleDarkMode}
      />
    </div>

    <!-- dark mode -->
    <div class="settings-field checkbox-field">
      <label for="verboseCategories">Verbose Tab Formatting</label>
      <input
        type="checkbox"
        name="verboseCategories"
        bind:checked={verboseTabs}
      />
    </div>

    <button on:click={saveSettings}>Save</button>
  </div>
{/if}

<style>
  h1 {
    margin: 10px 0px;
  }

  .settings {
    position: absolute;
    left: 10px;
    top: 10px;
    display: flex;
    flex-direction: column;
    text-align: left;
    background-color: var(--bg-alt);
    padding: 10px;
    color: var(--text-color);
    width: 250px;
    border-radius: 3px;
    box-shadow: 0px 0px 10px 0px rgba(0, 0, 0, 0.35);
    z-index: 1;
  }

  .settings-field {
    width: 100%;
    padding: 0px 0px;
    /* border: 1px solid red; */
  }

  .settings-field label {
    padding: 5px 0px;
  }

  .hamburger-button {
    position: absolute;
    top: 5px;
    left: 5px;
    display: flex;
    flex-direction: column;
    justify-content: start;
    align-items: center;
    padding: 0px 10px;
    margin: 0px;
    height: 50px;
    background-color: transparent;
    /* border: 1px solid red; */
  }

  span {
    color: var(--text-color);
    margin: 0px;
    padding: 0px;
    height: 8px;
    display: inline-flex;
  }

  button {
    padding: 5px;
  }

  .sidebar-close {
    position: absolute;
    top: 10px;
    right: 10px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    padding: 0px 15px 5px 15px;
    font-size: 28px;
    /* background-color: transparent; */
  }

  .checkbox-field {
    display: flex;
    justify-content: space-between;
  }
</style>
