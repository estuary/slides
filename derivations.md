---
title: Estuary Flow
separator: <!--s-->
verticalSeparator: <!--v-->
theme: league
scripts: 'lib/mermaid.min.js,lib/reveal-mermaid.js'
---

# Derivations

<!--s-->

```SQL
CREATE VIEW sensor_id_date_rollup AS
SELECT
    sensor_id,
    utc_date,
    MIN(degrees) AS degrees_min,
    SUM(1)       AS n_samples,
    FROM sensor_readings
    GROUP BY sensor_id, utc_date;
```

In SQL, you model views as queries.

<!--s-->

```yaml
collections:
  - name: sensor/id_date_rollup
    key: [/sensor_id, /utc_date]
    schema: schema.yaml#SensorRollup

    derivation:
      transform:
        fromReadings:
          source: { name: sensor/reading }
```

With Flow, use a [derived collection](https://estuary.readthedocs.io/en/latest/docs/concepts.html#derivations).
* _key_ defines group-by.
* _schema_ defines reductions.

<!--s-->
<!-- .slide: data-auto-animate -->

```JavaScript
// TypeScript lambda which maps from a `source` reading.
return [{
  sensor_id:   source.sensor_id,
  utc_date:    moment(source.timestamp).format('YYYY-MM-DD'),
  degrees_min: source.degrees,
  n_samples:   1
}];
```

[λ's](https://estuary.readthedocs.io/en/latest/docs/concepts.html#lambdas) map from source collection documents.<br>
Use familiar languages & libraries.

<!--s-->
<!-- .slide: data-auto-animate -->

```JavaScript
// TypeScript lambda which maps from a `source` reading.
return [{
  sensor_id:   source.sensor_id,
  utc_date:    moment(source.timestamp).format('YYYY-MM-DD'),
  degrees_min: source.degrees,
  n_samples:   1
}];
```

```yaml
$anchor: SensorRollup
properties:
  sensor_id:   { type: integer }
  utc_date:    { format: date }
  degrees_min: { type: number }
  n_samples:   { type: integer }
```

Must match collection schema.<br>
Checked at both build & runtime.

<!--s-->

Flow executes via continuous map & combine.<br>
[Materialize](https://estuary.readthedocs.io/en/latest/docs/concepts.html#materializations) into a database to fully reduce the view.

<!--s-->

λ's are scale-out, pure functions.<br>
[Registers](https://estuary.readthedocs.io/en/latest/docs/concepts.html#registers) are fast, durable scratch spaces for state:

* Arbitrary JSON documents.
* Declare schema & ``reduce`` annotations.

<!--s-->
<!-- .slide: data-auto-animate -->

Example: find zero-crossings of a running sum.

```YAML
# Register schema
type: number
reduce: { strategy: sum }
```

"Update" lambda runs first:<br>
- Maps a `source` document to register updates.<br>
- Flow reduces these into its value.<br>

<!--s-->
<!-- .slide: data-auto-animate -->

Example: find zero-crossings of a running sum.

```YAML
# Register schema
type: number
reduce: { strategy: sum }
```

```JavaScript
// Update lambda: accumulate into running sum.
return [source.NumberToSum];
```

<!--s-->
<!-- .slide: data-auto-animate -->

Example: find zero-crossings of a running sum.

```JavaScript
// Update lambda: accumulate into running sum.
return [source.NumberToSum];
```

"Publish" lambda runs second:<br>
- Takes: _source_ document, current _register_, and _previous_ register.
- Maps to new documents of the derived collection.<br>

<!--s-->
<!-- .slide: data-auto-animate -->

Example: find zero-crossings of a running sum.

```JavaScript
// Update lambda: accumulate into running sum.
return [source.NumberToSum];
```

```JavaScript
// Publish lambda: did our update cross over zero?
if (register > 0 != previous > 0) {
  return [source];
}
return [];
```

<!--s-->

Read multiple sources within one derivation.

```yaml [7,8,11,12]
collections:
  - name: sensor/reading_with_calibration
    key: [/sensor_id, /timestamp]

    derivation:
      transform:
        fromReadings:
          source: { name: sensor/readings }
          shuffle: [/sensor_id]

        fromCalibrations:
          source: { name: sensor/calibrations }
          shuffle: [/calibrated_sensor_id]
```

Each has its own "publish" and "update" lambdas.

<!--s-->

Registers are shared, keyed by _shuffle_.<br>
Use them to [communicate](https://estuary.readthedocs.io/en/latest/derive-patterns/README.html) between lambdas.

<!--s-->

### What's next?

- [How Flow Helps: History](hfh-history.html)

