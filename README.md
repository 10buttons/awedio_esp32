# Awedio ESP32 &emsp; [![Latest Version]][crates.io]

ESP32 I2S backend for the [awedio] audio playback library using ESP-IDF.
Requires std and ESP-IDF v5.

mp3 is supported but may not work well on ESPs without native floating point
support.

## Setup

The caller is responsible for setting up the I2S driver before calling start on
the backend. For example:


```rust no_run
use esp_idf_svc::hal;
use hal::i2s::config;

const SAMPLE_RATE: u32 = 44100;
const CHANNEL_COUNT: u16 = 1;

let i2s_config = config::StdConfig::new(
    config::Config::default(),
    config::StdClkConfig::from_sample_rate_hz(SAMPLE_RATE),
    config::StdSlotConfig::philips_slot_default(
        config::DataBitWidth::Bits16,
        config::SlotMode::Mono,
    ),
    config::StdGpioConfig::default(),
);

let peripherals = hal::peripherals::Peripherals::take().unwrap();
let i2s = peripherals.i2s0;
let blk = peripherals.pins.gpio44;
let dout = peripherals.pins.gpio42;
let mclk: Option<hal::gpio::AnyIOPin> = None;
let ws = peripherals.pins.gpio43;
let driver = hal::i2s::I2sDriver::new_std_tx(i2s, &i2s_config, bclk, dout, mclk, ws).unwrap();

let backend = awedio_esp32::Esp32Backend::with_defaults(
    driver,
    CHANNEL_COUNT,
    SAMPLE_RATE,
    128,
);
let manager = backend.start()
```

In order to get the `rmp3` native dependency to compile for xtensa chips
(if the rmp3-mp3 feature is enabled) you may need to export the following
variables (adjust for your target):
`export CROSS_COMPILE=xtensa-esp32s3-elf; export CFLAGS=-mlongcalls`

## Motivation

Built for creating activities for [10 Buttons](https://www.10Buttons.com), a
screen-less tablet for kids. Purposefully kept generic to be usable in other
contexts.

## Features

* report-render-time: Print to stdout stats about rendering time.

## License

This project is licensed under either of
[Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0) or
[MIT license](https://opensource.org/licenses/MIT) at your option.

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.

[Latest Version]: https://img.shields.io/crates/v/awedio_esp32.svg
[crates.io]: https://crates.io/crates/awedio_esp32
[awedio]: https://docs.rs/awedio
