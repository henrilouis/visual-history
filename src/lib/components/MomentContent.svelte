<script lang="ts">
  let {
    date,
    items,
    deleteHistoryUrl,
  }: {
    date: string;
    items: chrome.history.HistoryItem[];
    deleteHistoryUrl: (url: string) => void;
  } = $props();

  import { dateTimeFormatOptions } from "../utils/general";
  import { blur } from "svelte/transition";
  import { flip } from "svelte/animate";
  // these animations destroy performance on the large render, need to fix that
  // before addingh them back in
  import { getFaviconURL } from "../utils/chrome-api";

  // Format header based on key type (day: "2024-01-15" or hour: "2024-01-15T14")
  function formatHeader(key: string): string {
    if (items[0]?.lastVisitTime) {
      const d = new Date(items[0].lastVisitTime);
      const dateStr = d.toLocaleDateString(undefined, dateTimeFormatOptions);
      if (key.includes("T")) {
        const hour = key.split("T")[1];
        return `${dateStr} at ${hour}:00`;
      }
      return dateStr;
    }
    // Fallback to parsing the key
    if (key.includes("T")) {
      const [dateKey, hour] = key.split("T");
      return `${new Date(dateKey).toLocaleDateString(undefined, dateTimeFormatOptions)} at ${hour}:00`;
    }
    return new Date(key).toLocaleDateString(undefined, dateTimeFormatOptions);
  }
</script>

<header>
  <h3>{formatHeader(date)}</h3>
</header>
<ol>
  {#each items as item, index (item.id)}
    <!-- <li out:blur={{ duration: 150 }} animate:flip={{ delay: 150,duration: 150 }}> -->
    <li>
      <time
        >{item.lastVisitTime
          ? new Date(item.lastVisitTime).toLocaleTimeString([], {
              hour: "numeric",
              minute: "numeric",
              hour12: false,
            })
          : ""}</time
      >
      <img
        src={item.url ? getFaviconURL(new URL(item.url).hostname) : ""}
        alt={`Favicon for ${item.url ? new URL(item.url).hostname : ""}`}
      />
      <div>
        <a href={item.url}>{item.title}</a>
        <span class="text-secondary"
          >{item.url ? new URL(item.url).hostname : ""}</span
        >
      </div>
      <button
        class="quiet"
        onclick={() => {
          item.url ? deleteHistoryUrl(item.url) : null;
        }}>Delete</button
      >
    </li>
  {/each}
</ol>

<style>
  ol {
    list-style: none;
    padding: 0;
    margin: 0;
  }
  li {
    font-size: 0.75rem;
    display: grid;
    grid-template-columns: 4rem 16px 1fr auto;
    align-items: center;
    gap: 0.5rem;
    padding-block: 0.25rem;
    &:not(:last-child) {
      border-bottom: var(--el-border-width) solid var(--el-border-color-default);
    }
  }
</style>
