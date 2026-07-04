# MonitoringMexico Data Feed

Machine-readable export of the indicators tracked by the MonitoringMexico
pipeline: Banxico/INEGI/FRED/SHF economic data plus derived model output
(nowcasts, forecasts, indices).

## Files

- `latest.json` -- schema below. One entry per topic; each series entry
  points at the CSV containing its full history.
- `<topic>.csv` -- one file per topic, long-format history of that
  topic's series of interest (see CSV format below). Topic keys match the
  directory names used internally by the pipeline's state store (e.g.
  `cpi`, `employment`, `banxicospf`).

## JSON schema (`latest.json`)

| Field | Type | Description |
|---|---|---|
| `schema_version` | int | Feed schema version. Bumped on breaking changes. |
| `generated_at` | string | ISO 8601 timestamp (with UTC offset) of when this feed was built. |
| `run_id` | string or null | Identifier of the pipeline run that produced this feed, if available. |
| `topics` | object | Map of topic key -> topic object (below). Keys sorted alphabetically. |

Each **topic object**:

| Field | Type | Description |
|---|---|---|
| `series` | array | Series of interest for this topic (the data shown on the public site/dashboard). Sorted alphabetically by `name`. |
| `predictions` | array | Forecasts/nowcasts for this topic, tracked but not displayed. Sorted alphabetically by `name`. |
| `noticeable_today` | bool | Whether this topic had an editorially "noticeable" data update in the run that produced this feed. `false` when that information wasn't supplied to the feed builder. |
| `n_series` | int | Number of entries in `series`. |

Each entry in `series`:

| Field | Type | Description |
|---|---|---|
| `name` | string | Series identifier (matches the `series` column in the topic's CSV). |
| `latest_date` | string | ISO date (`YYYY-MM-DD`) of the most recent observation. |
| `latest_value` | number or null | Value at `latest_date`. `null` if the latest observation is missing/NaN. |
| `frequency` | string | Inferred nominal frequency: `daily`, `weekly`, `biweekly`, `monthly`, or `quarterly`. Inferred from the median day-gap of the last 8 observations. |
| `history_csv` | string | Filename (within this feed directory) of the CSV containing this series' full history. |

Each entry in `predictions`:

| Field | Type | Description |
|---|---|---|
| `name` | string | Forecast identifier. |
| `reference_date` | string or null | Date the forecast was generated from (the last actual observation used as the model's anchor). `null` if not recorded. |
| `latest_date` | string | ISO date of the last point in the stored forecast series (typically the far end of the projection horizon, not the near-term forecast). |
| `latest_value` | number or null | Value at `latest_date`. |

All numeric values are plain JSON numbers -- never NaN/Infinity; missing
values are `null`. All dates are ISO 8601 date strings.

## CSV format (`<topic>.csv`)

Long format, one row per (series, date) observation, full history for
every series of interest in that topic:

```
series,date,value
cpi_headline,2020-01-01,3.24
cpi_headline,2020-01-16,3.31
```

Columns:
- `series` -- matches `name` in the corresponding `latest.json` series entry.
- `date` -- ISO date (`YYYY-MM-DD`).
- `value` -- observation value (blank if missing).

Rows are sorted by `(series, date)`. Predictions/forecasts are not
included in the CSV -- only their latest value + reference date appear in
`latest.json`.

## Update cadence

Regenerated on each pipeline run; series update when sources release data.

## Attribution

Underlying data: Banco de México (SIE), INEGI, FRED, SHF and public news
sources. Derived indicators and nowcasts are model output of the
MonitoringMexico project. Verify licensing with the original sources
before redistribution.
