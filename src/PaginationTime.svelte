<script>
  import { tick } from "svelte";

  export let periods;
  export let selectedPeriod;
  export let paginationPos;
  export let loading;

  let clientWidth;
  let leftPosition = "0px";
  let tabWidth = "0px";
  let tabHeight = "0px";
  let showCustom = false;
  let id = Math.random().toString(36).substring(2, 15);

  $: setIndicator(clientWidth, selectedPeriod);
  $: aggregation = periods.find((p) => p.value == selectedPeriod)?.aggregation;

  const setIndicator = async (period, clientWidth) => {
    await tick();
    const firstTab = document
      .getElementById(id)
      ?.querySelector(".card-tab.active");
    if (firstTab) {
      leftPosition = firstTab.offsetLeft + "px";
      tabWidth = firstTab.offsetWidth + "px";
      tabHeight = firstTab.offsetHeight + "px";
    } // Logic to set indicator based on period and clientWidth
  };
</script>

<div
  class="paginator-time"
  class:top={paginationPos === "top"}
  bind:clientWidth
  style:--left-position={leftPosition}
  style:--tab-width={tabWidth}
  style:--tab-height={tabHeight}
  style:--paginator-justify-content={aggregation
    ? "space-between"
    : "flex-start"}
>
  <div {id} class="card-tabs" class:loading>
    {#each periods || [] as period}
      <button
        class="card-tab"
        class:active={selectedPeriod === period.value}
        on:click={(e) => {
          selectedPeriod = period.value;
        }}
      >
        {clientWidth > 600 ? period.label : period.labelMob}
      </button>
    {/each}
    <button
      class="card-tab"
      class:active={selectedPeriod === "custom"}
      on:click={(e) => {
        showCustom = !showCustom;
      }}
    >
      {clientWidth > 600 ? "Custom" : "Cust"}
    </button>
    {#if showCustom}
      DADADADA
    {/if}
  </div>

  {#if aggregation}
    <div class="aggregation">{aggregation}</div>
  {/if}
</div>

<style>
  @media (max-width: 600px) {
    .paginator-time {
      flex-direction: column;
      align-items: center !important;
    }

    .aggregation {
      display: none;
    }

    .card-tabs {
      width: 100%;
      justify-content: center;
    }
    .card-tab {
      width: 100%;
      text-align: center;
      padding: 0.25rem 0.5rem;
      font-size: 13px;
    }
  }

  .paginator-time {
    display: flex;
    align-items: center;
    font-size: 11px;
    justify-content: var(--paginator-justify-content, flex-start);
    border: 1px solid var(--spectrum-global-color-gray-300);
    background-color: var(--spectrum-global-color-gray-100);
    border-radius: 0.25rem;
  }

  .aggregation {
    border-radius: 0.25rem;
    padding: 0.25rem 0.5rem;
    margin-right: 0.5rem;
    color: var(--spectrum-global-color-gray-900);
    min-width: 2rem;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 12px;
  }

  .card-tabs {
    position: relative;
    display: flex;
    z-index: 0;
    min-width: 0;
    gap: 0.25rem;
    padding: 0.25rem;
  }

  .card-tabs.loading {
    pointer-events: none;
    opacity: 0.5;
  }

  .card-tabs::before {
    content: "";
    position: absolute;
    top: 0.25rem;
    left: var(--left-position, 0);
    width: var(--tab-width);
    height: var(--tab-height, 2px);
    background-color: var(--spectrum-global-color-gray-50);
    border-radius: 0.25rem;
    transition: all 0.2s ease-in-out;
    z-index: -1;
  }

  .card-tab {
    padding: 0.25rem 0.5rem;
    cursor: pointer;
    transition: color 0.2s ease-in-out;
    color: var(--spectrum-global-color-gray-600);
    text-overflow: ellipsis;
    white-space: nowrap;
    overflow: hidden;
    min-width: 4rem;
    display: flex;
    justify-content: center;
    background: transparent;
    border-color: transparent;
    font-size: 12px;
  }

  .card-tab:hover {
    color: var(--spectrum-global-color-gray-800);
  }

  .card-tab.active {
    color: var(--spectrum-global-color-gray-900);
    border-radius: 0.25rem;
  }
</style>
