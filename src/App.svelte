<script lang="ts">
  import Search from "./lib/components/Search.svelte";
  import Card from "./lib/components/Card.svelte";
  import DayCalendar from "./lib/components/DayCalendar.svelte";
  import HourCalendar from "./lib/components/HourCalendar.svelte";
  import MomentContent from "./lib/components/MomentContent.svelte";
  import { historyStore } from "./lib/stores/history.svelte";
  import { dateTimeFormatOptions } from "./lib/utils/general";

  // Initialize data fetch
  historyStore.fetch();

  // Local binding for search input
  let searchValue = $state("");

  // Sync search input to store (debounced would be nice for performance)
  $effect(() => {
    historyStore.setSearch(searchValue);
  });

  // Format moment key for display (handles both day and hour formats)
  function formatMomentLabel(key: string): string {
    if (key.includes("T")) {
      const [date, hour] = key.split("T");
      const d = new Date(date);
      return `${d.toLocaleDateString(undefined, dateTimeFormatOptions)} at ${hour}:00`;
    }
    return new Date(key).toLocaleDateString(undefined, dateTimeFormatOptions);
  }
</script>

<header>
  <h1>Visual history</h1>
  <Search bind:value={searchValue} />
</header>
<main>
  <div class="calendar-controls">
    <button
      class={historyStore.calendarMode === "day" ? "active" : ""}
      onclick={() => historyStore.setCalendarMode("day")}
    >
      Days
    </button>
    <button
      class={historyStore.calendarMode === "hour" ? "active" : ""}
      onclick={() => historyStore.setCalendarMode("hour")}
    >
      Hours
    </button>
  </div>

  {#if historyStore.calendarMode === "day"}
    <DayCalendar
      data={historyStore.byDayWithEmpty}
      selectedMoments={historyStore.selectedMoments}
      onToggleMoment={historyStore.toggleMoment}
    />
  {:else}
    <HourCalendar
      data={historyStore.byDayAndHourWithEmpty}
      selectedMoments={historyStore.selectedMoments}
      onToggleMoment={historyStore.toggleMoment}
    />
  {/if}

  <section class="days">
    {#if historyStore.selectedMoments.length > 0}
      {#each historyStore.selectedMoments as momentKey}
        {@const items = historyStore.getItemsForMoment(momentKey)}
        {#if items.length > 0}
          <Card>
            <MomentContent
              date={momentKey}
              {items}
              deleteHistoryUrl={historyStore.removeUrl}
            />
          </Card>
        {:else}
          <Card>
            <h3>{formatMomentLabel(momentKey)}</h3>
            No results for this time
          </Card>
        {/if}
      {/each}
    {:else if historyStore.isLoading}
      <Card loading={true} />
      <Card loading={true} />
      <Card loading={true} />
    {:else if Object.keys(historyStore.filtered).length === 0}
      <Card>
        <h3>No results found</h3>
      </Card>
    {:else}
      {#each Object.entries(historyStore.byDay) as [date, items]}
        <Card>
          <MomentContent {date} {items} deleteHistoryUrl={historyStore.removeUrl} />
        </Card>
      {/each}
    {/if}
  </section>
</main>

<style>
  header {
    position: sticky;
    top: 0;
    background-color: var(--bg-secondary);
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 1rem;
  }
  h1 {
    margin: 0;
    font-size: 1.375rem;
    position: absolute;
    display: none;
  }
  @media (min-width: 840px) {
    h1 {
      display: block;
    }
  }
  main {
    padding: 0 1rem 1rem 1rem;
    margin-inline: auto;
    max-width: 60rem;
  }
  .calendar-controls {
    display: flex;
    justify-content: center;
    gap: 0.5rem;
    margin-top: 1rem;
  }
  .calendar-controls button {
    padding: 0.5rem 1rem;
    border: 1px solid var(--el-border-color-default, #333);
    background: var(--el-bg-default, transparent);
    color: var(--text-primary, inherit);
    border-radius: var(--el-border-radius, 4px);
    cursor: pointer;
    transition: all 0.15s ease;
  }
  .calendar-controls button:hover {
    background: var(--el-bg-hover, #222);
  }
  .calendar-controls button.active {
    background: var(--heatmap-color-2, #39d353);
    color: var(--bg-primary, #000);
    border-color: transparent;
  }
  .days {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }
</style>
