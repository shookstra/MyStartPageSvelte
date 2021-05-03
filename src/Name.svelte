<script>
  import { onMount } from "svelte";

  let name = "";

  const db = new Dexie("settings");
  db.version(1).stores({
    settings: "settingName,settingValue",
  });

  onMount(async () => {
    await getName();
  });

  const getName = async () => {
    await db.settings
      .get("name")
      .then((setting) => {
        name = setting.settingValue;
        // console.log(setting.settingValue);
      })
      .catch((err) => {
        console.log(err);
      });
  };
</script>

{#if name}
  <h3>Hi, {name}</h3>
{:else}
  <h3>Let's Get Started</h3>
{/if}
