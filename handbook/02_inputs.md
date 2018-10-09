# input

## namespace

Related metrics can share a namespace:
```rust,skt-run
let app_metrics = metric_scope(to_stdout());
let db_metrics = app_metrics.add_prefix("database");
let _db_timer = db_metrics.timer("db_timer");
let _db_counter = db_metrics.counter("db_counter");
```

## proxy

## counter

## marker

## timer

Timers can be used multiple ways:
```rust,skt-run
let app_metrics = metric_scope(to_stdout());
let timer =  app_metrics.timer("my_timer");
time!(timer, {/* slow code here */} );
timer.time(|| {/* slow code here */} );

let start = timer.start();
/* slow code here */
timer.stop(start);

timer.interval_us(123_456);
```

## gauge


## ad-hoc metrics

Where necessary, metrics can also be defined _ad-hoc_ (or "inline"):

```rust,skt-run
let user_name = "john_day";
let app_metrics = metric_scope(to_log()).with_cache(512);
app_metrics.gauge(format!("gauge_for_user_{}", user_name)).value(44);
```

## ad-hoc metrics cache 

Defining a cache is optional but will speed up re-definition of common ad-hoc metrics.


## local vs global scopes
