# smo_sepa_kiwis

A pure-Ruby client for the SEPA Time Series KiWIS API at
`https://timeseries.sepa.org.uk/KiWIS/KiWIS`. Fetch rainfall stations,
15-minute timeseries metadata, and timeseries values. Export everything to CSV.

## Why this gem

Built specifically for use inside InfoWorks ICM's embedded Ruby interpreter,
where no `gem install` step is possible. Uses stdlib only (`net/http`, `uri`,
`json`, `csv`, `date`, `time`), no native extensions, and no Bundler runtime
dependencies. Load it with a single `require_relative`.

## Installation

**Option A. Clone and require directly (recommended for ICM):**

```ruby
require_relative "/path/to/smo_sepa_kiwis/lib/smo_sepa_kiwis"
```

**Option B. Install from RubyGems (when published):**

```sh
gem install smo_sepa_kiwis
```

```ruby
require "smo_sepa_kiwis"
```

## Quick start

```ruby
require "smo_sepa_kiwis"

client = SmoSepaKiwis::Client.new
# Optional kwargs: base_url:, timeout: (default 60), user_agent:

# 1. List all rainfall stations.
stations = client.rainfall_stations
# => Array<SmoSepaKiwis::Station>
# Station fields: no, name, lat, lon, catchment, river

# 2. Find 15-minute timeseries for a specific station.
series = client.rainfall_15min_timeseries(station_no: "14964")
# => Array<SmoSepaKiwis::Timeseries>
# Timeseries fields: ts_id, ts_path, ts_name, station_no, coverage_from, coverage_to

# 3. Full inventory: all 15-min series with station details in one call.
inventory = client.rainfall_15min_inventory
# => Array<Hash> keys: :station_no, :station_name, :lat, :lon,
#    :catchment, :river, :ts_id, :ts_path, :coverage_from, :coverage_to

# 4. Download timeseries values. from/to accept String, Date, Time, or DateTime.
values = client.timeseries_values(
  ts_id: 55570010,
  from: "2021-10-22",
  to: "2021-10-25"
)
# => Array<SmoSepaKiwis::Value>
# Value fields: timestamp (Time UTC), value (Float or nil), quality_code (Integer or nil)

# For long date ranges, use chunk_days to avoid server timeouts.
values = client.timeseries_values(
  ts_id: 55570010,
  from: "2020-01-01",
  to: "2021-01-01",
  chunk_days: 30
)

# 5. Write inventory to CSV.
client.rainfall_15min_inventory_to_csv("inventory.csv")

# 6. Write timeseries values to CSV.
client.timeseries_values_to_csv(
  ts_id: 55570010,
  from: "2021-10-22",
  to: "2021-10-25",
  path: "rainfall.csv"
)
```

## Data fidelity note

This gem returns SEPA fields verbatim. String fields that are blank in the API
response are set to `nil`. Numeric fields (`lat`, `lon`, `easting`, `northing`,
`ts_id`) are coerced from the string representation SEPA provides. No values
are derived or synthesised.

## Support

If this gem saves you time on a project, you can buy me a coffee.

[![Buy Me a Coffee](https://github.com/Sebasmadridmx/SMO-WGS84-TO-BNG/blob/main/temp_png/buymecoffeeqr.png)](https://buymeacoffee.com/smadrid)

## License

MIT. Copyright (c) 2024 Sebastian Madrid Ontiveros.
