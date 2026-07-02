<script lang="ts">
  import { invoke } from "@tauri-apps/api/core";
  import { onMount } from "svelte";

  type Flip = {
    market_hash_name: string;
    lowest_price: string;
    median_price: string;
    volume: number | null;
    margin_pct: number;
  };

  let flips = $state<Flip[]>([]);
  let loading = $state(true);

  async function load() {
    loading = true;
    try {
      flips = await invoke<Flip[]>("flips_get");
    } finally {
      loading = false;
    }
  }
  onMount(load);
</script>

<div class="search">
  <span class="hint">recent sales above the lowest ask, after the ~15% fee</span>
  <button class="btn" onclick={load} disabled={loading}>{loading ? "…" : "refresh"}</button>
</div>

{#if loading}
  <p class="empty">scanning the market…</p>
{:else if flips.length === 0}
  <p class="empty">No opportunities right now — check back after the next market refresh.</p>
{:else}
  <div class="wl-head">
    <span>opportunity</span>
    <span class="flip-cols"><span>ask → sold</span><span>margin</span></span>
  </div>
  <ul class="list">
    {#each flips as f (f.market_hash_name)}
      <li class="row">
        <span class="name">{f.market_hash_name}</span>
        <span class="muted-price">{f.lowest_price}</span>
        <span class="arrow">→</span>
        <span class="price">{f.median_price}</span>
        <span class="margin">+{f.margin_pct}%</span>
      </li>
    {/each}
  </ul>
  <p class="disclaimer">
    A signal, not a guarantee: it flags items whose recent sale price beats the current lowest ask after
    fees. Thin markets can mislead — always check the item before buying.
  </p>
{/if}

<style>
  .flip-cols {
    display: flex;
    gap: 14px;
  }
  .arrow {
    color: var(--muted);
    font-size: 10px;
  }
  .margin {
    color: var(--accent);
    font-variant-numeric: tabular-nums;
    min-width: 48px;
    text-align: right;
  }
  .disclaimer {
    font-size: 9px;
    color: var(--muted);
    line-height: 1.5;
    margin: 10px 2px 0;
  }
</style>
