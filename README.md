# SuperDataProvider

A sophisticated data management component that serves as a centralized data source for child components, providing advanced data handling, caching, pagination, and context management for Budibase applications.

## 🚀 Features

### Data Management Modes

- **Normal Mode**: Standard data collection with filtering and sorting
- **Single Row Mode**: Optimized for individual record operations
- **Timeseries Mode**: Specialized handling for time-based data with aggregation

### Advanced Data Options

- **Flexible Filtering**: Comprehensive filter and search capabilities
- **Multi-Column Sorting**: Sort by any field in ascending/descending order
- **Pagination Control**: Page-based or datetime-based pagination
- **Auto-refresh**: Scheduled data updates at configurable intervals
- **Row Limits**: Configurable data size limits for performance

### Context Integration

- **Rich Context Provision**: Exposes data, schema, and metadata to child components
- **Aggregation Support**: Timeseries data aggregation with multiple time windows
- **Data Transformation**: Template-based data manipulation capabilities
- **State Management**: Centralized data state for component synchronization

### Child Component Support

- **Nested Components**: Provides data context to all child components
- **Shared Data**: Single data source for multiple consuming components
- **Optimization**: Efficient data sharing without duplicate queries
- **Real-time Updates**: Coordinated refresh across all dependent components

## 🎯 Usage Instructions

### Basic Setup

1. Add the SuperDataProvider component to your screen
2. Select your data source (table, view, external datasource)
3. Configure data filtering, sorting, and pagination options
4. Add child components that will consume the data
5. Set up auto-refresh if needed for real-time data

### Data Source Configuration

#### Simple Data Source

- **Data Source**: Select from available Budibase datasources
- **Mode**: Choose Normal (default), Single Row, or Timeseries
- **Limit**: Set maximum rows to retrieve
- **Refresh**: Configure auto-refresh interval

#### Advanced Filtering & Sorting

- **Filter Panel**: Apply complex filters based on data types
- **Sort Column**: Select column for sorting
- **Sort Order**: Choose ascending or descending
- **Pagination**: Enable/disable pagination controls

### Timeseries Configuration

#### Time Series Setup

- **Mode**: Set to Timeseries
- **Timestamp Field**: Select timestamp column
- **Value Field**: Specify value column for metrics
- **Aggregation Views**: Enable aggregates at different intervals

#### Aggregation Levels

- **5-minute Aggregates**: Enable for detailed time windows
- **15/30-minute Aggregates**: Medium-term aggregation
- **1-hour Aggregates**: Hourly summaries
- **1-day Aggregates**: Daily rollups
- **Data Sources**: Separate datasources for each aggregation level

### Child Component Integration

#### Data Consumption

- **Context Access**: Child components access `rows`, `schema`, `pagination`
- **Template Binding**: Use `{{ rows }}` and `{{ rowsLength }}` in bindings
- **Dynamic Updates**: All child components update when data refreshes
- **Performance**: Shared query avoids duplicate data fetching

#### Responsive Data Display

- **Automatic Updates**: Components respond to data changes
- **State Synchronization**: Coordinate data state across components
- **Error Handling**: Graceful handling of data loading errors

## 🔧 Configuration Properties

### Core Data Settings

- **Mode**: Normal, Single Row, or Timeseries data processing
- **Data Source**: Budibase datasource selection
- **Limit**: Maximum number of rows to retrieve (default: 50)
- **Auto Refresh**: Never, Auto, or specified intervals (5s to 10m)

### Data Processing Options

- **Filter**: Comprehensive filter configuration panel
- **Sort Column**: Field selection for sorting
- **Sort Order**: Ascending or descending sort direction
- **Pagination**: None, Page-based, or datetime-based pagination
- **Pagination Position**: Top or bottom placement of controls

### Timeseries Configuration

- **Timestamp Field**: Time field for timeseries processing
- **Value Field**: Metric value field selection
- **Aggregation Views**: Multiple time-window aggregations
- **Separate Datasources**: Individual datasources for each aggregation

### Layout Settings

- **Flex Mode**: Shrink or grow container sizing

## 📋 Usage Examples

### Basic Data Provider - Standard Use

Configure fundamental data access:

- Mode: Normal
- Data Source: `User Accounts` table
- Filter: Active users only
- Sort: By name, ascending
- Limit: 100 rows

### Single Record Provider - Individual Items

Configure for specific record operations:

- Mode: Single Row
- Data Source: `Product Details` table
- Filter: By product ID
- Refresh: Every 30 seconds

### Timeseries Data Provider - Metrics Tracking

Configure timeseries data management:

- Mode: Timeseries
- Data Source: `Performance Metrics` table
- Timestamp Field: `measured_at`
- Value Field: `response_time`
- 5m/15m/30m aggregation enabled

### Paginated Data Provider - Large Datasets

Configure pagination for large datasets:

- Mode: Normal
- Data Source: `Transaction History` table
- Pagination: Page-based
- Page Size: 50 rows
- Position: Bottom

### Auto-refresh Data Provider - Real-time Data

Configure live data updates:

- Mode: Normal
- Data Source: `Live Sensors` table
- Auto Refresh: Every 10 seconds
- Limit: 25 most recent readings

## 🎨 Context & Binding

### Exposed Context Variables

The SuperDataProvider exposes rich context data:

- **`rows`**: Full array of data rows (`{{ rows }}`)
- **`rowsLength`**: Total number of rows (`{{ rowsLength }}`)
- **`schema`**: Data structure definition (`{{ schema }}`)
- **`pageNumber`**: Current page number (`{{ pageNumber }}`)
- **`period`**: Selected time period for timeseries
- **`aggregation`**: Current aggregation level
- **`info`**: Additional metadata string

### Child Component Binding

Child components can access data through these bindings:

#### In SuperTable

```json
{
  "datasource": "{{ SuperDataProvider.rows }}",
  "schema": "{{ SuperDataProvider.schema }}"
}
```

#### In SuperCharts

```json
{
  "dataProvider": "{{ SuperDataProvider.rows }}",
  "timestampColumn": "{{ SuperDataProvider.timestamp }}"
}
```

#### Conditional Display

```json
{
  "hidden": "{{ SuperDataProvider.rowsLength === 0 }}",
  "text": "{{ SuperDataProvider.rowsLength }} items loaded"
}
```

## 🔄 Data Flow Architecture

### Provider-Consumer Pattern

- **Central Data Management**: Single data source authority
- **Child Component Consumption**: Automatic data availability
- **Consistent Context**: Standardized data access patterns
- **Performance Optimization**: Shared data queries and caching

### Data State Management

- **Reactive Updates**: Automatic child component updates
- **Refresh Coordination**: Synchronized across all consumers
- **Error Propagation**: Consistent error handling patterns
- **Loading States**: Coordinated loading and ready states

## 📊 Advanced Features

### Aggregation Layer

- **Multiple Time Windows**: 5m, 15m, 30m, 1h, 1d aggregations
- **Separate Data Sources**: Dedicated datasources for each window
- **Performance Optimization**: Pre-aggregated data for large datasets
- **Query Optimization**: Reduced data processing overhead

### Pagination Strategies

- **Page-based**: Standard page number navigation
- **Datetime-based**: Time-range navigation for timeseries
- **Cursor-based**: Efficient navigation for large datasets
- **Flexible Positioning**: Top or bottom placement options

### Auto-refresh Capabilities

- **Flexible Scheduling**: From 5 seconds to 10 minutes
- **Background Updates**: Automated without user interaction
- **Event-driven Refresh**: Trigger updates from other components
- **Smart Refresh**: Conditional updates based on data changes

## 🔗 Integration Patterns

### Table Integration

- **Data Binding**: Direct binding to table datasources
- **Column Mapping**: Automatic field mapping and validation
- **CRUD Operations**: Create, read, update, delete integration
- **Bulk Operations**: Multi-record operations support

### Chart Integration

- **Data Visualization**: Chart components connected to provider data
- **Real-time Updates**: Charts automatically refresh with new data
- **Context Sharing**: Chart filters connected to provider filters
- **Performance**: Optimized data passing for smooth rendering

### Form Integration

- **Data Population**: Forms pre-filled from provider data
- **Validation**: Schema-based validation integration
- **Save Operations**: Direct save actions to provider datasource
- **State Sync**: Form data synchronized with provider context

## 📱 Datascience & Analytics

### Timeseries Analysis

- **Timeline Playback**: Navigate through historical data
- **Aggregation Views**: Multiple detail levels in same interface
- **Trend Analysis**: Visual trend identification
- **Performance Metrics**: Response time and throughput tracking

### Data Quality Management

- **Schema Validation**: Consistent data structure enforcement
- **Quality Checks**: Data validation and completeness monitoring
- **Error Handling**: Graceful degradation on data issues
- **Audit Trails**: Data access and modification tracking

## 🐛 Troubleshooting

### Data Loading Issues

- **Datasource Connection**: Verify data source configuration
- **Authentication**: Check datasource access permissions
- **Query Errors**: Review filter and sort configurations
- **Network Issues**: Confirm external datasource connectivity

### Child Component Issues

- **Context Access**: Ensure correct binding syntax
- **Update Delay**: Check refresh interval settings
- **Component State**: Verify component lifecycle states
- **Memory Usage**: Monitor for excessive data retention

### Performance Problems

- **Query Efficiency**: Optimize filter and sorting selections
- **Data Size**: Adjust row limits for memory optimization
- **Refresh Frequency**: Balance update needs with performance
- **Aggregation Settings**: Ensure efficient data aggregation

Find out more about developing [Custom Plugins](https://docs.budibase.com/docs/custom-plugin) and [Budibase](https://github.com/Budibase/budibase).
