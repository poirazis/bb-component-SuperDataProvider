<script>
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();
  export let limit = 50;
  export let currentOffset = 0;
  export let dataSource = null;
  export let fetch = null;

  $: currentPage = Math.floor(currentOffset / limit) + 1;
  $: hasMorePages = $fetch?.rows?.length >= limit;

  function goToPrevious() {
    if (currentOffset > 0) {
      currentOffset = Math.max(0, currentOffset - limit);
      updateDataSource();
    }
  }

  function goToNext() {
    if (hasMorePages) {
      currentOffset += limit;
      updateDataSource();
    }
  }

  function goToFirst() {
    currentOffset = 0;
    updateDataSource();
  }

  function updateDataSource() {
    if (dataSource) {
      dataSource.set({
        ...dataSource.value,
        queryParams: {
          ...dataSource.value.queryParams,
          offset: currentOffset,
        },
      });
    }
  }
</script>

<div class="pagination-limit-offset">
  <div class="pagination-controls">
    <button
      class="pagination-btn"
      on:click={goToFirst}
      disabled={currentPage <= 1}
      title="First Page"
    >
      <i class="ph ph-caret-double-left"></i>
    </button>

    <button
      class="pagination-btn"
      on:click={goToPrevious}
      disabled={currentPage <= 1}
      title="Previous Page"
    >
      <i class="ph ph-caret-left"></i>
    </button>

    <span class="pagination-info">
      Page {currentPage}
      (Offset: {currentOffset}, Limit: {limit})
    </span>

    <button
      class="pagination-btn"
      on:click={goToNext}
      disabled={!hasMorePages}
      title="Next Page"
    >
      <i class="ph ph-caret-right"></i>
    </button>
  </div>
</div>

<style>
  .pagination-limit-offset {
    display: flex;
    justify-content: center;
    padding: 0.5rem;
    background: var(--spectrum-global-color-gray-50);
    border-radius: var(--spectrum-border-radius);
  }

  .pagination-controls {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .pagination-btn {
    aspect-ratio: 1 / 1;
    border: 1px solid var(--spectrum-global-color-gray-300);
    background: var(--spectrum-global-color-gray-100);
    border-radius: 0.25rem;
    color: var(--spectrum-global-color-gray-700);
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .pagination-btn:hover:not(:disabled) {
    background: var(--spectrum-global-color-gray-200);
    border-color: var(--spectrum-global-color-gray-400);
  }

  .pagination-btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  .pagination-info {
    font-size: 0.875rem;
    color: var(--spectrum-global-color-gray-700);
    white-space: nowrap;
  }
</style>
