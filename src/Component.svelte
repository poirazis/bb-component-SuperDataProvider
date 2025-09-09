<script lang="ts">
  import { getContext, onDestroy, setContext } from "svelte";
  import Wizard from "./Wizard.svelte";
  import MetricSelector from "./MetricSelector.svelte";

  import type {
    TableSchema,
    SortOrder,
    SearchFilters,
    UISearchFilter,
    DataFetchDatasource,
    UserDatasource,
    GroupUserDatasource,
    DataFetchOptions,
  } from "@budibase/types";
  import { LogicalOperator, EmptyFilterOption } from "@budibase/types";

  type ProviderDatasource = Exclude<
    DataFetchDatasource,
    UserDatasource | GroupUserDatasource
  >;

  import PaginationTime from "./PaginationTime.svelte";
  import { set } from "zod";

  const {
    styleable,
    Provider,
    ActionTypes,
    API,
    fetchData,
    QueryUtils,
    memo,
    builderStore,
  } = getContext("sdk");
  const component = getContext("component");

  export let text;
  export let showFooter;

  export let dataSource: ProviderDatasource;
  export let filter: UISearchFilter;
  export let sortColumn: string;
  export let sortOrder: SortOrder;
  export let limit: number;
  export let paginate: boolean;
  export let autoRefresh: number | string;
  export let pagination: string;
  export let paginationPos: "top" | "bottom" = "bottom";
  export let tsField: string;
  export let valueField: any;
  export let mode: string = "normal";
  export let flex: boolean = false;

  const periods = [
    {
      label: "Live",
      labelMob: "10m",
      value: "10m",
      aggregation: "1s",
      dataRefresh: 5,
      chartRefresh: 0.5,
    },
    {
      label: "1 Hour",
      labelMob: "1H",
      value: "1h",
      aggregation: "5s",
      dataRefresh: 60,
      chartRefresh: 10,
    },
    {
      label: "12 Hours",
      labelMob: "12H",
      value: "12h",
      aggregation: "1m",
      dataRefresh: 60,
      chartRefresh: 20,
    },
    {
      label: "1 Day",
      labelMob: "1D",
      value: "1d",
      aggregation: "150s",
      dataRefresh: 60,
      chartRefresh: 60,
    },
    {
      label: "1 Week",
      labelMob: "1W",
      value: "1w",
      aggregation: "30m",
      dataRefresh: 60,
      chartRefresh: 60,
    },
    {
      label: "1 Month",
      labelMob: "1M",
      value: "1month",
      aggregation: "1h",
      dataRefresh: 60,
      chartRefresh: 60,
    },
    {
      label: "3 Months",
      labelMob: "3M",
      value: "3months",
      aggregation: "1d",
      dataRefresh: 60,
      chartRefresh: 60,
    },
    {
      label: "6 Months",
      labelMob: "6M",
      value: "6months",
      aggregation: "1d",
      dataRefresh: 60,
      chartRefresh: 60,
    },
    {
      label: "1 Year",
      labelMob: "1Y",
      value: "1y",
      aggregation: "1d",
      dataRefresh: 60,
      chartRefresh: 60,
    },
  ];

  const metrics = [
    { label: "Ping", value: 1, aggregation: "1s" },
    { label: "CPU", value: 2, aggregation: "1s" },
    { label: "Network", value: 3, aggregation: "1s" },
    { label: "Wireless", value: 4, aggregation: "1s" },
  ];

  let selectedPeriod = "10m";
  let selectedMetric = 1;
  let selectedChartRefresh = 0.5;
  let selectedDataRefresh = 5;

  let timeSeriesOptions = memo({
    labelColumn: tsField,
    valueColumn: [valueField],
    period: selectedPeriod,
    aggregation: periods.find((p) => p.value == selectedPeriod)?.aggregation,
  });

  $: periods_reverse = [...periods].reverse();
  $: inBuilder = $builderStore.inBuilder;

  let interval: ReturnType<typeof setInterval>;
  let chartInterval: ReturnType<typeof setInterval>;

  let queryExtensions: Record<string, any> = {};

  let dataSourceStore = memo(dataSource);
  $: dataSourceStore.set(dataSource);

  $: hasPeriodParameter = dataSource?.parameters?.find(
    (param) => param.name == "period"
  );

  $: showWizard = mode == "timeseries" && !hasPeriodParameter;

  let filterStore = memo(filter);
  $: filterStore.set(filter);

  $: age = new Date().toLocaleTimeString([], {
    hour: "2-digit",
    minute: "2-digit",
  });

  $: aggregation = periods.find((p) => p.value == selectedPeriod)?.aggregation;
  $: selectedDataRefresh =
    periods.find((p) => p.value == selectedPeriod)?.dataRefresh || 0;
  $: selectedChartRefresh =
    periods.find((p) => p.value == selectedPeriod)?.chartRefresh || 0;

  $: defaultQuery = QueryUtils.buildQuery(filter);

  // We need to manage our lucene query manually as we want to allow components
  // to extend it
  $: query = extendQuery(defaultQuery, queryExtensions);
  $: fetch = createFetch($dataSourceStore);
  $: fetch.update({
    query,
    sortColumn,
    sortOrder,
    limit,
    paginate,
  });
  $: schema = sanitizeSchema($fetch.schema);
  $: setUpAutoRefresh(autoRefresh, selectedDataRefresh, $dataSourceStore);
  $: setUpAutoChartRefresh(autoRefresh, selectedChartRefresh);

  $: actions = [
    {
      type: ActionTypes.RefreshDatasource,
      callback: () => fetch.refresh(),
      metadata: { dataSource },
    },
    {
      type: ActionTypes.AddDataProviderQueryExtension,
      callback: addQueryExtension,
    },
    {
      type: ActionTypes.RemoveDataProviderQueryExtension,
      callback: removeQueryExtension,
    },
    {
      type: ActionTypes.SetDataProviderSorting,
      callback: ({
        column,
        order,
      }: {
        column: string;
        order: SortOrder | undefined;
      }) => {
        let newOptions: Partial<DataFetchOptions> = {};
        if (column) {
          newOptions.sortColumn = column;
        }
        if (order) {
          newOptions.sortOrder = order;
        }
        if (Object.keys(newOptions)?.length) {
          fetch.update(newOptions);
        }
      },
    },
  ];

  $: dataContext = {
    rows: $fetch.rows,
    info: $fetch.info,
    datasource: dataSource || {},
    schema,
    rowsLength: $fetch.rows.length,
    pageNumber: $fetch.pageNumber + 1,
    // Undocumented properties. These aren't supposed to be used in builder
    // bindings, but are used internally by other components
    id: $component?.id,
    state: {
      query: $fetch.query,
    },
    limit,
    primaryDisplay: ($fetch.definition as any)?.primaryDisplay,
    loaded: $fetch.loaded,
    loading: $fetch.loading,
    timeseries: {
      labelColumn: tsField,
      valueColumn: [valueField],
      period: selectedPeriod,
      aggregation,
    },
  };

  $: timeSeriesOptions.set({
    labelColumn: tsField,
    valueColumn: [valueField],
    period: selectedPeriod,
    aggregation: periods.find((p) => p.value == selectedPeriod)?.aggregation,
    rows: $fetch?.rows || [],
    loaded: $fetch?.loaded,
  });

  const createFetch = (datasource: ProviderDatasource) => {
    return fetchData({
      API,
      datasource,
      options: {
        query,
        sortColumn,
        sortOrder,
        limit,
        paginate,
      },
    });
  };

  const sanitizeSchema = (schema: TableSchema | null) => {
    if (!schema) {
      return schema;
    }
    let cloned = { ...schema };
    Object.entries(cloned).forEach(([field, fieldSchema]) => {
      if (fieldSchema.visible === false) {
        delete cloned[field];
      }
    });
    return cloned;
  };

  const addQueryExtension = (key: string, extension: any) => {
    if (!key || !extension) {
      return;
    }
    queryExtensions = { ...queryExtensions, [key]: extension };
  };

  const removeQueryExtension = (key: string) => {
    if (!key) {
      return;
    }
    const newQueryExtensions = { ...queryExtensions };
    delete newQueryExtensions[key];
    queryExtensions = newQueryExtensions;
  };

  const extendQuery = (
    defaultQuery: SearchFilters,
    extensions: Record<string, any>
  ): SearchFilters => {
    if (!Object.keys(extensions).length) {
      return defaultQuery;
    }
    const extended: SearchFilters = {
      [LogicalOperator.AND]: {
        conditions: [
          ...(defaultQuery ? [defaultQuery] : []),
          ...Object.values(extensions || {}),
        ],
      },
      onEmptyFilter: EmptyFilterOption.RETURN_NONE,
    };

    // If there are no conditions applied at all, clear the request.
    return (extended[LogicalOperator.AND]?.conditions?.length ?? 0) > 0
      ? extended
      : {};
  };

  const setUpAutoRefresh = (autoRefresh: any, selectedDataRefresh: number) => {
    clearInterval(interval);
    if (inBuilder) return;
    if (autoRefresh != "auto") {
      interval = setInterval(
        () => {
          if (!$fetch?.loading) fetch.refresh();
        },
        Math.max(5000, autoRefresh * 1000)
      );
    } else if (autoRefresh == "auto" && selectedDataRefresh) {
      interval = setInterval(
        () => {
          if (!$fetch?.loading) fetch.refresh();
        },
        Math.max(5000, selectedDataRefresh * 1000)
      );
    }
  };

  const setUpAutoChartRefresh = (autoRefresh: any, chartRefresh: number) => {
    clearInterval(chartInterval);
    if (inBuilder) return;

    if (autoRefresh == "auto") {
      chartInterval = setInterval(
        () => {
          timeSeriesOptions.set({
            labelColumn: tsField,
            valueColumn: [valueField],
            period: selectedPeriod,
            aggregation: periods.find((p) => p.value == selectedPeriod)
              ?.aggregation,
            rows: $fetch?.rows || [],
            loaded: true,
            refresh: Date.now(),
          });
        },
        Math.max(500, chartRefresh * 1000)
      );
    }
  };

  onDestroy(() => {
    clearInterval(interval);
    clearInterval(chartInterval); // Clears auto-refresh when navigating away
  });

  $: if ($dataSourceStore?.parameters && selectedPeriod) {
    $dataSourceStore = {
      ...$dataSourceStore,
      queryParams: {
        ...$dataSourceStore.queryParams,
        period: selectedPeriod,
        bucket: aggregation,
        metric_id: selectedMetric,
      },
    };
  }

  $: $component.styles = {
    ...$component.styles,
    normal: {
      ...$component.styles.normal,
      flex: flex ? "auto" : "none",
    },
  };

  setContext("superDataProvider", timeSeriesOptions);
</script>

<div class="super-data-provider" use:styleable={$component.styles}>
  <Provider data={dataContext} {actions} />

  <MetricSelector {metrics} bind:selectedMetric />

  <div class="inner-wrapper">
    <slot />
  </div>

  {#if pagination == "datetime"}
    <PaginationTime
      {autoRefresh}
      {periods}
      {paginationPos}
      bind:selectedPeriod
    />
  {/if}
</div>

<style>
  .super-data-provider {
    display: flex;
    flex-direction: column;
    width: 100%;
    gap: 1rem;
    overflow: hidden;

    & > .inner-wrapper {
      width: 100%;
      height: 100%;
      overflow: hidden;
      padding: var(--super-data-provider-padding);
    }
  }
</style>
