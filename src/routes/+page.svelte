<script lang="ts">
  //load data on mount
  $effect(() => {
    const saved = localStorage.getItem("cleaningData");
    if (saved) {
      cleaningData = JSON.parse(saved);
    }
  });

  // Data Structures
  interface CleaningEntry {
    id: number;
    roommate: string;
    month: string;
    isComplete: boolean;
    completionDate: string | null;
  }

  //save whenever data changes
  $effect(() => {
    localStorage.setItem("cleaningData", JSON.stringify(cleaningData));
  });

  function generateSchedule(
    startMonth: Date,
    roommates: string[],
    months: number = 12
  ) {
    const schedule: CleaningEntry[] = [];

    for (let i = 0; i < months; i++) {
      const date = new Date(startMonth);
      date.setMonth(date.getMonth() + i);

      schedule.push({
        id: i + 1,
        roommate: roommates[i % roommates.length],
        month: date.toLocaleString("en-US", { month: "long", year: "numeric" }),
        isComplete: false,
        completionDate: null,
      });
    }
    return schedule;
  }

  //usage
  const roommates = ["Ankit", "Abdulaziz", "Ahmed", "Mohammed"];
  let cleaningData = generateSchedule(new Date(2025, 10), roommates);

  // Logic Function
  function handleToggle(entry: CleaningEntry) {
    if (entry.isComplete) {
      // if checked set a date
      entry.completionDate = new Date().toLocaleString("en-US", {
        year: "numeric",
        month: "short",
        day: "numeric",
      });
    } else {
      // if unchecked set to null
      entry.completionDate = null;
    }

    // svelte reavtivity
    cleaningData = cleaningData;
  }
</script>

<div class="schedule-container">
  <h2>Bathroom Cleaning Schedule</h2>
  <table class="cleaning-table">
    <thead>
      <tr>
        <th>Name</th>
        <th>Month</th>
        <th>Status</th>
        <th>Date</th>
      </tr>
    </thead>
    <tbody>
      {#each cleaningData as entry (entry.id)}
        <tr class={entry.isComplete ? "completed-row" : ""}>
          <td>{entry.roommate}</td>
          <td>{entry.month}</td>
          <td>
            <input
              type="checkbox"
              bind:checked={entry.isComplete}
              on:change={() => handleToggle(entry)}
            />
            <span class="status-text"
              >{#if entry.isComplete}
                Done
              {:else}
                Pending
              {/if}</span
            ></td
          >
          <td class="log-column">
            {#if entry.completionDate}
              {entry.completionDate}
            {/if}
          </td>
        </tr>
      {/each}
    </tbody>
  </table>
</div>

<style>
  .schedule-container {
    max-width: 800px;
    margin: 2rem auto;
    padding: 1rem;
    font-family: Arial, sans-serif;
  }

  .cleaning-table {
    width: 100%;
    border-collapse: collapse;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }

  .cleaning-table th,
  .cleaning-table td {
    padding: 12px 15px;
    border: 1px solid #ddd;
    text-align: left;
  }
  .cleaning-table th {
    background-color: #f4f4f4;
    font-weight: bold;
    color: #333;
  }
  .completed-row {
    background-color: #e6ffe6;
  }
  .log-column {
    font-size: 0.9em;
    color: #555;
  }
  .status-text {
    margin-left: 10px;
    font-weight: 500;
  }
</style>
