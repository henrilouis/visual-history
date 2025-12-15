<script lang="ts">
  import Search from "./lib/components/Search.svelte";
  import Card from "./lib/components/Card.svelte";
  import Calendar from "./lib/components/Calendar.svelte";
  import MomentContent from "./lib/components/MomentContent.svelte";

  import { getHistoryByDay, type HistoryByDay } from "./lib/utils/chrome-api";
  import CardLoading from "./lib/components/CardLoading.svelte";

  import { dateTimeFormatOptions } from "./lib/utils/general";

  let search: string = $state("");
  let selectedMoments: string[] = $state([]);

  // Cache the resolved data in state
  let historyData = $state<HistoryByDay | null>(null);
  let isLoading = $state(false);

  // Fetch data when search changes
  $effect(() => {
    isLoading = true;
    getHistoryByDay(search, true).then((result) => {
      historyData = result;
      isLoading = false;
    });
  });
</script>

<header>
  <h1>Visual history</h1>
  <Search bind:value={search} />
</header>
<main>
  {#if historyData}
    <Calendar data={historyData} bind:selectedMoments {isLoading} />
    <section class="days">
      {#if selectedMoments.length > 0}
        {#each selectedMoments as date}
          {#if historyData[date]?.length > 0}
            <Card>
              <MomentContent {date} items={historyData[date]} />
            </Card>
          {:else}
            <Card>
              <h3>
                {new Date(date).toLocaleDateString(
                  undefined,
                  dateTimeFormatOptions
                )}
              </h3>
              No results for this date
            </Card>
          {/if}
        {/each}
      {:else}
        {#each Object.entries(historyData) as [date, items]}
          {#if items.length > 0}
            <Card>
              <MomentContent {date} {items} />
            </Card>
          {/if}
        {/each}
      {/if}
    </section>
  {:else}
    <div style="margin-block: 3rem; text-align: center;">
      Loading calendar...
    </div>
    <section class="days">
      <CardLoading />
      <CardLoading />
      <CardLoading />
    </section>
  {/if}
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
  .days {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }
</style>
