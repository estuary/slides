---
title: Estuary Flow
separator: <!--s-->
verticalSeparator: <!--v-->
theme: league
scripts: 'lib/mermaid.min.js,lib/reveal-mermaid.js'
---

# Reductions

<!--s-->

```SQL [3-5]
SELECT
    sensor_id,
    MIN(degrees) AS degrees_min,
    MAX(degrees) AS degrees_max,
    SUM(1)       AS n_samples,
    FROM readings
    GROUP BY sensor_id;
```

In SQL, describe reductions as _aggregate functions_.

<!--s-->

```SQL [5-7]
SELECT
    sensor_id,
    site_id,
    utc_date,
    MIN(degrees) AS degrees_min,
    MAX(degrees) AS degrees_max,
    SUM(1)       AS n_samples,
    FROM readings
    GROUP BY sensor_id, site_id, utc_date;
```

We end up repeating ourselves quite a bit.<br>
Most metrics have only one sensible aggregation.

<!--s-->

```yaml [5,7,8,10,11,13]
properties:
    sensor_id:  { type: integer }
    site_id: { type: integer }
    utc_date:   { format: date }
    degrees_min:
      type: number
      reduce: {strategy: minimize}
    degrees_max:
      type: number
      reduce: {strategy: maximize}
    n_samples:
      type: integer
      reduce: {strategy: sum}
```

Flow "hoists" description to the JSON schema.<br>
Use one schema, grouped many ways.

<!--s-->

```yaml []
properties:
  # Roll up a sorted array of site IDs.
  site_metrics:
    items: "#SensorRollup"
    reduce:
      strategy: merge
      key: [/site_id]
   
```

Go beyond fact tables!<br>
Express compose-able, nested reductions.

<!--s-->

```JSON
{ "site_metrics": [{"site_id": 7, "n_samples": 3}] }
{ "site_metrics": [{"site_id": 9, "n_samples": 2}] }
{ "site_metrics": [{"site_id": 7, "n_samples": 9}] }
```

⇩

```JSON
{
  "site_metrics": [
    {"site_id": 7, "n_samples": 12},
    {"site_id": 9, "n_samples": 2}
  ]
}
```

<!--s-->

```yaml []
# Leave, or overwrite a prior value based on `action`.
oneOf:
  - properties:
      action: { const: insert }
    reduce: { strategy: firstWriteWins }

  - properties:
      action: { const: update }
    reduce: { strategy: lastWriteWins }
```

As _annotations_, they react to schema evaluation.<br>
Use conditionals to build more advanced behaviors.

