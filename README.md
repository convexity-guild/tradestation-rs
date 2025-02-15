# TradeStation Rust Client

An ergonomic Rust client for the [TradeStation API](https://www.tradestation.com/platforms-and-tools/trading-api/). 

This is a fork of the [tradestation crate](https://github.com/antonio-hickey/tradestation-rs), but more opinionated and providing higher level abstractions.

* [crates.io homepage](https://crates.io/crates/tradestation-rs)
* [documentation](https://docs.rs/tradestation-rs/latest/tradestation-rs)

Install
---
Use cargo CLI:
```
cargo install tradestation-rs
```

Or manually add it into your `Cargo.toml`:
```toml
[dependencies]
tradestation-rs = "0.1.2"
```

Usage
---

For more thorough information, read the [docs](https://docs.rs/tradestation-rs/latest/tradestation-rs/).

Simple example for streaming bars of trading activity:
```rust
use tradestation_rs::{
    responses::MarketData::StreamBarsResp,
    ClientBuilder, Error,
    MarketData::{self, BarUnit},
};

#[tokio::main]
async fn main() -> Result<(), Error> {
    let mut client = ClientBuilder::new()?
        .set_credentials("YOUR_CLIENT_ID", "YOUR_CLIENT_SECRET")?
        .authorize("YOUR_AUTHORIZATION_CODE")
        .await?
        .build()
        .await?;
    println!("Your TradeStation API Bearer Token: {:?}", client.token);

    let stream_bars_query = MarketData::StreamBarsQueryBuilder::new()
        .set_symbol("CLX24")
        .set_unit(BarUnit::Minute)
        .set_interval("240")
        .build()?;

    let streamed_bars = client
        .stream_bars(&stream_bars_query, |stream_data| {
            match stream_data {
                StreamBarsResp::Bar(bar) => {
                    // Do something with the bars like making a chart
                    println!("{bar:?}")
                }
                StreamBarsResp::Heartbeat(heartbeat) => {
                    if heartbeat.heartbeat > 10 {
                        return Err(Error::StopStream);
                    }
                }
                StreamBarsResp::Status(status) => {
                    println!("{status:?}");
                }
                StreamBarsResp::Error(err) => {
                    println!("{err:?}");
                }
            }

            Ok(())
        })
        .await?;

    // All the bars collected during the stream
    println!("{streamed_bars:?}");

    Ok(())
}
```
