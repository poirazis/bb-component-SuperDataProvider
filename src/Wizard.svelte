<script>
  export let dataSource;

  $: hasPeriodParameter = dataSource?.parameters?.find(
    (param) => param.name === "period"
  );
  $: hasAggregation = dataSource?.parameters?.find(
    (param) => param.name === "aggregation"
  );
</script>

<div class="super-data-provider-wizard">
  Your dataSource is missing the 'period' parameter, which is required for
  pagination.
</div>
