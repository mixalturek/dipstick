A quick, modular metrics toolkit for Rust applications; similar to popular logging frameworks,
but with counters, markers, gauges and timers.

Dipstick builds on stable Rust with minimal dependencies
and is published as a [crate.](https://crates.io/crates/dipstick)

## Features

  - Send metrics to stdout, log, statsd or graphite (one or many)
  - Synchronous, asynchronous or mixed operation
  - Optional fast random statistical sampling
  - Immediate propagation or local aggregation of metrics (count, sum, average, min/max)
  - Periodic or programmatic publication of aggregated metrics
  - Customizable output statistics and formatting
  - Global or scoped (e.g. per request) metrics
  - Per-application and per-output metric namespaces
  - Predefined or ad-hoc metrics

## Cookbook

Dipstick is easy to add to your code:
```rust
use dipstick::*;
let app_metrics = metrics(to_graphite("host.com:2003"));
app_metrics.counter("my_counter").count(3);
```

Metrics can be sent to multiple outputs at the same time:
```rust
let app_metrics = metrics((to_stdout(), to_statsd("localhost:8125", "app1.host.")));
```
Since instruments are decoupled from the backend, outputs can be swapped easily.

Metrics can be aggregated and scheduled to be published periodically in the background:
```rust
use std::time::Duration;
let (to_aggregate, from_aggregate) = aggregate();
publish_every(Duration::from_secs(10), from_aggregate, to_log("last_ten_secs:"), all_stats);
let app_metrics = metrics(to_aggregate);
```
Aggregation is performed locklessly and is very fast.
Count, sum, min, max and average are tracked where they make sense.
Published statistics can be selected with presets such as `all_stats` (see previous example),
`summary`, `average`.

For more control over published statistics, a custom filter can be provided:
```rust
let (_to_aggregate, from_aggregate) = aggregate();
publish(from_aggregate, to_log("my_custom_stats:"),
    |metric_kind, metric_name, metric_score|
        match metric_score {
            HitCount(hit_count) => Some((Counter, vec![metric_name, ".per_thousand"], hit_count / 1000)),
            _ => None
        });
```

Metrics can be statistically sampled:
```rust
let app_metrics = metrics(sample(0.001, to_statsd("server:8125", "app.sampled.")));
```
A fast random algorithm is used to pick samples.
Outputs can use sample rate to expand or format published data.

Metrics can be recorded asynchronously:
```rust
let app_metrics = metrics(async(48, to_stdout()));
```
The async queue uses a Rust channel and a standalone thread.
The current behavior is to block when full.

Metric definitions can be cached to make using _ad-hoc metrics_ faster:
```rust
let app_metrics = metrics(cache(512, to_log()));
app_metrics.gauge(format!("my_gauge_{}", 34)).value(44);
```

The preferred way is to _predefine metrics_,
possibly in a [lazy_static!](https://crates.io/crates/lazy_static) block:
```rust
#[macro_use] external crate lazy_static;

lazy_static! {
    pub static ref METRICS: GlobalMetrics<String> = metrics(to_stdout());
    pub static ref COUNTER_A: Counter<Aggregate> = METRICS.counter("counter_a");
}
COUNTER_A.count(11);
```

Timers can be used multiple ways:
```rust
let timer =  app_metrics.timer("my_timer");
time!(timer, {/* slow code here */} );
timer.time(|| {/* slow code here */} );

let start = timer.start();
/* slow code here */
timer.stop(start);

timer.interval_us(123_456);
```

Related metrics can share a namespace:
```rust
let db_metrics = app_metrics.with_prefix("database.");
let db_timer = db_metrics.timer("db_timer");
let db_counter = db_metrics.counter("db_counter");
```


## Design
Dipstick's design goals are to:
- support as many metrics backends as possible while favoring none
- support all types of applications, from embedded to servers
- promote metrics conventions that facilitate app monitoring and maintenance
- stay out of the way in the code and at runtime (ergonomic, fast, resilient)

## Performance
Predefined timers use a bit more code but are generally faster because their initialization cost is is only paid once.
Ad-hoc timers are redefined "inline" on each use. They are more flexible, but have more overhead because their init cost is paid on each use.
Defining a metric `cache()` reduces that cost for recurring metrics.

Run benchmarks with `cargo +nightly bench --features bench`.

## TODO
Although already usable, Dipstick is still under heavy development and makes no guarantees
of any kind at this point. See the following list for any potential caveats :
- META turn TODOs into GitHub issues
- generic publisher / sources
- feature flags
- time measurement units in metric kind (us, ms, etc.) for naming & scaling
- heartbeat metric on publish
- logger templates
- configurable aggregation (?)
- non-aggregating buffers
- framework glue (rocket, iron, gotham, indicatif, etc.)
- more tests & benchmarks
- complete doc / inline samples
- more example apps
- A cool logo
- method annotation processors `#[timer("name")]`
- fastsinks (M / &M) vs. safesinks (Arc<M>)
- `static_metric!` macro to replace `lazy_static!` blocks and handle generics boilerplate.

License: MIT/Apache-2.0

_this file was generated using [cargo readme](https://github.com/livioribeiro/cargo-readme)_
