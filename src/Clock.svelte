<script>
  import { onMount } from "svelte";

  let time;
  let timeFormat;

  const db = new Dexie("settings");
  db.version(1).stores({
    settings: "settingName,settingValue",
  });

  onMount(async () => {
    await getTimeFormat();
    if (timeFormat === "24-hour") {
      startTime();
    } else {
      formatAMPM();
    }
  });

  const getTimeFormat = async () => {
    await db.settings
      .get("timeFormat")
      .then((setting) => {
        timeFormat = setting.settingValue;
        // console.log(setting.settingValue);
      })
      .catch((err) => {
        console.log(err);
      });
  };

  // 24-hour time functions from
  // https://www.w3schools.com/js/tryit.asp?filename=tryjs_timing_clock
  const startTime = () => {
    var today = new Date();
    var h = today.getHours();
    var m = today.getMinutes();
    var s = today.getSeconds();
    m = checkTime(m);
    s = checkTime(s);
    time = h + ":" + m + ":" + s;
    setTimeout(startTime, 500);
  };

  const checkTime = (i) => {
    if (i < 10) {
      i = "0" + i;
    } // add zero in front of numbers < 10
    return i;
  };

  // 12-hour time formatter here:
  // https://stackoverflow.com/questions/8888491/how-do-you-display-javascript-datetime-in-12-hour-am-pm-format
  const formatAMPM = () => {
    var today = new Date();
    var hours = today.getHours();
    var minutes = today.getMinutes();
    var seconds = today.getSeconds();
    var ampm = hours >= 12 ? "pm" : "am";
    hours = hours % 12;
    hours = hours ? hours : 12; // the hour '0' should be '12'
    minutes = checkTime(minutes);
    seconds = checkTime(seconds);
    time = hours + ":" + minutes + ":" + seconds + " " + ampm;
    setTimeout(formatAMPM, 500);
  };
</script>

<h3>{time}</h3>

<style>
  h3 {
    margin-bottom: 10px;
  }

  :global(body.dark h3) {
    color: #eee;
  }

  :global(body h3) {
    color: black;
  }
</style>
