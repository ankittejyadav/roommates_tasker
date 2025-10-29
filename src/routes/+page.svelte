<script lang="ts">
  // Data Structures
  interface CleaningEntry {
    id: number;
    roommate: string;
    month: string;
    isComplete: boolean;
    completionDate: string | null;
  }

  // Initial Data
  let cleaningData: CleaningEntry[] = [
    {
      id: 1,
      roommate: "Ahmed",
      month: "November 2025",
      isComplete: false,
      completionDate: null,
    },
    {
      id: 2,
      roommate: "Ankit",
      month: "December 2025",
      isComplete: false,
      completionDate: null,
    },
    {
      id: 3,
      roommate: "Mohammed",
      month: "January 2026",
      isComplete: false,
      completionDate: null,
    },
    {
      id: 4,
      roommate: "AbdulAziz",
      month: "February 2026",
      isComplete: false,
      completionDate: null,
    },
    {
      id: 5,
      roommate: "Ahmed",
      month: "March 2026",
      isComplete: false,
      completionDate: null,
    },
    {
      id: 6,
      roommate: "Ankit",
      month: "April 2026",
      isComplete: false,
      completionDate: null,
    },
    {
      id: 7,
      roommate: "Mohammed",
      month: "May 2026",
      isComplete: false,
      completionDate: null,
    },
    {
      id: 8,
      roommate: "AbdulAziz",
      month: "June 2026",
      isComplete: false,
      completionDate: null,
    },
    {
      id: 9,
      roommate: "Ahmed",
      month: "July 2026",
      isComplete: false,
      completionDate: null,
    },
    {
      id: 10,
      roommate: "Ankit",
      month: "August 2026",
      isComplete: false,
      completionDate: null,
    },
  ];

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

  .cleaning table th,
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
