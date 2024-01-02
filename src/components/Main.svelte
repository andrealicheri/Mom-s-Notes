<script>
  import Note from "../components/Note.svelte";
  let currentYear = new Date().getFullYear();
  import { swipeable } from "@react2svelte/swipeable";

  function getDaysInMonth(year, month) {
    const date = new Date(year, month + 1, 0);
    const daysInMonth = date.getDate();
    return Array.from({ length: daysInMonth }, (_, i) => i + 1);
  }

  let showNote = false;
  let noteID = null;

  function setNote(date) {
    showNote = true;
    noteID = date;
  }
  function decrementDateByOneDay() {
    let parts = noteID.split("/");
    let day = parseInt(parts[0], 10);
    let month = parseInt(parts[1], 10) - 1; // Months are zero-based
    let year = parseInt(parts[2], 10);
    let currentDate = new Date(year, month, day);
    currentDate.setDate(currentDate.getDate() - 1);
    let newDay = currentDate.getDate();
    let newMonth = currentDate.getMonth() + 1; // Months are zero-based
    let newYear = currentDate.getFullYear();
    newDay = newDay < 10 ? "0" + newDay : newDay;
    newMonth = newMonth < 10 ? "0" + newMonth : newMonth;
    noteID = `${newDay}/${newMonth}/${newYear}`;
  }

  function incrementDateByOneDay() {
    let parts = noteID.split("/");
    let day = parseInt(parts[0], 10);
    let month = parseInt(parts[1], 10) - 1;
    let year = parseInt(parts[2], 10);
    let currentDate = new Date(year, month, day);
    currentDate.setDate(currentDate.getDate() + 1);
    let newDay = currentDate.getDate();
    let newMonth = currentDate.getMonth() + 1;
    let newYear = currentDate.getFullYear();
    newDay = newDay < 10 ? "0" + newDay : newDay;
    newMonth = newMonth < 10 ? "0" + newMonth : newMonth;
    noteID = `${newDay}/${newMonth}/${newYear}`;
  }

  function zerofy(input) {
    if (input <= 9) {
      return "0"
    } else {
      console.log(input)
      return ""
    }
  }
  import { onMount } from "svelte";
  let localStorageEntries = [];

  function updateEntries() {
      const entries = Object.entries(localStorage).map(([key, value]) => ({
          key,
          value,
          timestamp: new Date(localStorage.getItem(key)).getTime(),
      }));

      // Filter out entries with empty values
      const nonEmptyEntries = entries.filter(
          (entry) =>
              entry.value !== null &&
              entry.value !== undefined &&
              entry.value !== "" &&
              entry.value !== " ",
      );

      // Sort non-empty entries by timestamp in descending order
      localStorageEntries = nonEmptyEntries.sort(
          (a, b) => b.timestamp - a.timestamp,
      );

      setTimeout(updateEntries, 1000);
  }
  onMount(() => {
      updateEntries();
  });

  let hasNoteKey = false;
  // Check if any key in localStorage starts with "note"
  Object.keys(localStorage).forEach((key) => {
      if (key.startsWith("note")) {
          hasNoteKey = true;
      }
  });

  function truncateAndAddDots(variable) {
      if (variable.length > 10) {
          return variable.substring(0, 7) + "...";
      } else {
          return variable;
      }
  }
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
<div style="display: flex;">
  <div style="flex: 1;" id="sidebar">
    <p style="font-size: 1.7rem">Ciao, Paola!</p>
    <div>
      {#if !hasNoteKey}
          <p>Non hai ancora nessuna nota. Clicca su un giorno per crearne una.</p>
      {:else}
          <p style="font-size: 1.4rem;">Ecco le tue note più recenti:</p>
          <ul>
              {#each localStorageEntries as { key, value }}
                  {#if key.startsWith("note_")}
                      <li style="font-size: 1.2rem;"{key} on:click={() => setNote(key.substring(5))}>
                          <strong>{key.substring(5)}:</strong>
                          {truncateAndAddDots(value)}
                      </li>
                  {/if}
              {/each}
          </ul>
      {/if}
    </div>
  </div>
  <div style="flex: 2;">
    <div id="header">
      <p on:click={() => (showNote = false)}>Calendario</p>
    </div>
    {#if !showNote}
      <div id="main">
        <div
          style="display: flex; width: 100%; justify-content: center; align-items: center"
        >
          <button on:click={() => (currentYear -= 1)}>Anno Precedente</button>
          <button on:click={() => (currentYear += 1)}>Anno Successivo</button>
        </div>
        {#each Array(12) as _, month}
          <p>
            {new Date(currentYear, month, 1).toLocaleDateString("default", {
              year: "numeric",
              month: "long",
            })}
          </p>
          <div class="calendar">
            <!-- svelte-ignore a11y-no-static-element-interactions -->
            {#each getDaysInMonth(currentYear, month) as day}
              <!-- svelte-ignore a11y-click-events-have-key-events -->
              <div
                class="day"
                on:click={() =>
                  setNote(
                     zerofy(day) + day + "/" + zerofy(month) + String(Number(month) + 1) + "/" + currentYear,
                  )}
              >
                {zerofy(day) + day}
              </div>
            {/each}
          </div>
        {/each}
        <div
          style="display: flex; width: 100%; justify-content: center; align-items: center"
        >
          <button on:click={() => (currentYear -= 1)}>Anno Precedente</button>
          <button on:click={() => (currentYear += 1)}>Anno Successivo</button>
        </div>
      </div>
    {:else}
      {#key noteID}
        <div
          use:swipeable
          on:swipedleft={incrementDateByOneDay}
          on:swipedright={decrementDateByOneDay}
        >
          <Note noteId={noteID} />
        </div>
      {/key}
    {/if}
  </div>
</div>

<style>
  .calendar {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 10px;
  }
  .day {
    border-radius: 1000px;
    background-color: rgb(214, 251, 251);
    width: 1rem;
    padding: 10px;
    border: solid rgb(41, 41, 41);
  }

  button {
    background-color: rgb(214, 251, 251);
    border-radius: 25px;
    font-size: 1.5rem;
    color: rgb(0, 0, 0);
    border: solid rgb(41, 41, 41);
  }

  #header {
    width: 100%;
    font-size: 1.7rem;
    text-align: center;
  }

  /* Apply styles only in portrait mode */
  @media (orientation: portrait) {
    #sidebar {
      display: none;
      visibility:hidden;
    }
  }


  #sidebar {
    padding: 5px;
  }
</style>


