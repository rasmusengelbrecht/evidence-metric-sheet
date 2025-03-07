---
title: Metric Sheet
---

```sql metrics_query
SELECT
    name,
    month + interval '2 hours' AS month, -- Correcting for timezone differences
    market,
    metric_value,
    1.00 * (metric_value - LAG(metric_value) OVER (PARTITION BY name, market ORDER BY month)) / nullif(LAG(metric_value) OVER (PARTITION BY name, market ORDER BY month), 0) as mom_pct,
    1.00 * (metric_value - LAG(metric_value, 12) OVER (PARTITION BY name, market ORDER BY month)) / nullif(LAG(metric_value) OVER (PARTITION BY name, market ORDER BY month), 0) as yoy_pct,
    1.00 *((metric_value / nullif(LAG(metric_value, 3) OVER (PARTITION BY name, market ORDER BY month), 0))^(1/3)-1) as cgr_3_month_new_pct,
    1.00 *((metric_value / nullif(LAG(metric_value, 12) OVER (PARTITION BY name, market ORDER BY month), 0))^(1/12)-1) as cgr_12_month_new_pct,
    ARRAY_AGG({'month': month, 'metric_value': metric_value}) OVER (
        PARTITION BY name, market
        ORDER BY month
        ROWS BETWEEN 11 PRECEDING AND CURRENT ROW
    ) AS metric_value_last_12_months
FROM
    csv_files.metric_sheet_dummy_data
```


## Filters

```sql months_dropdown
SELECT distinct left(month,7) as month
FROM ${metrics_query}
order by month desc
```


<Dropdown
    data={months_dropdown}
    name=selected_month
    value=month
    order="month asc"
    defaultValue={months_dropdown[0].month}
/>


```sql market_dropdown
SELECT DISTINCT market
FROM ${metrics_query}
```


<Dropdown
    data={market_dropdown}
    name=selected_market
    value=market
    order="market asc"
    multiple=true
    selectAllByDefault=true
/>

```sql metric_dropdown
SELECT DISTINCT name
FROM ${metrics_query}
```


<Dropdown
    data={metric_dropdown}
    name=selected_metric
    value=name
    order="name asc"
    multiple=true
    selectAllByDefault=true
/>

<br>
<br>

---

<br>

## Metric Sheet

```sql selected_month_rows
SELECT
  name,
  market,
  metric_value,
  mom_pct,
  cgr_3_month_new_pct,
  yoy_pct,
  cgr_12_month_new_pct,
  metric_value_last_12_months
FROM
    ${metrics_query}
WHERE
  left(month,7) = '${inputs.selected_month.value}'
  and market in  ${inputs.selected_market.value}
  and name in  ${inputs.selected_metric.value}
order by market asc
```

<DataTable data={selected_month_rows} groupBy=name groupType=section sort="metric_value desc" >
	<Column id=name />
	<Column id=market height=35px/>
  <Column id=metric_value_last_12_months title="Past 12m" contentType=sparkbar sparkX=month sparkY=metric_value sparkColor=#556b2f/>
	<Column id=metric_value title='This Month'/>
  <Column id=mom_pct contentType=delta fmt=pct0 title="MoM"/>
  <Column id=cgr_3_month_new_pct contentType=delta fmt=pct0 title="3m CGR"/>
  <Column id=yoy_pct contentType=delta fmt=pct0 title="YoY"/>
</DataTable>

<br>

---

<br>

## Monthly Development

```sql names
  select distinct name
  from ${metrics_query}
  where name in ${inputs.selected_metric.value}
  order by metric_value desc
```

<ButtonGroup
    data={names}
    name=selected_name
    value=name
    display=tabs
    defaultValue={names[0].name}
/>

```sql name_filter
SELECT
  month,
  market,
  metric_value,
  mom_pct,
  cgr_3_month_new_pct
FROM
    ${metrics_query}
WHERE name = '${inputs.selected_name}'
  and market in  ${inputs.selected_market.value}
order by month desc
```


<LineChart
    data={name_filter}
    x=month
    y=metric_value
    title="{inputs.selected_name} per Month by Market"
    yAxisTitle="{inputs.selected_name}"
    series=market
    chartAreaHeight=480
    markers=true
    labels=true
/>
