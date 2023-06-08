# Awedio ESP32 &emsp; [![Latest Version]][crates.io]

ESP32 I2S backend for the [awedio] audio playback library using ESP-IDF.
Requires std.

mp3 is supported but may not work well on ESPs without native floating point.

## Setup

The caller is responsible for setting up the I2S port before calling start on
the backend. For example:


```rust no_run
const I2S_PORT_NUM: u32 = 0;
const SAMPLE_RATE: u32 = 44100;
const CHANNEL_COUNT: u16 = 1;

let config = esp_idf_sys::i2s_driver_config_t {
    mode: esp_idf_sys::i2s_mode_t_I2S_MODE_MASTER | esp_idf_sys::i2s_mode_t_I2S_MODE_TX,
    sample_rate: SAMPLE_RATE,
    bits_per_sample: esp_idf_sys::i2s_bits_per_chan_t_I2S_BITS_PER_CHAN_16BIT,
    channel_format: esp_idf_sys::i2s_channel_fmt_t_I2S_CHANNEL_FMT_ONLY_RIGHT,
    communication_format: esp_idf_sys::i2s_comm_format_t_I2S_COMM_FORMAT_STAND_I2S,
    intr_alloc_flags: esp_idf_sys::ESP_INTR_FLAG_LEVEL1 as i32, // Interrupt level 1, default 0
    dma_buf_count: 8,
    dma_buf_len: 64,
    use_apll: false,
    tx_desc_auto_clear: false,
    fixed_mclk: 0,
    mclk_multiple: esp_idf_sys::i2s_mclk_multiple_t_I2S_MCLK_MULTIPLE_DEFAULT,
    bits_per_chan: 0,
    bit_order_msb: false,
    big_edin: false,
    left_align: false,
    chan_mask: esp_idf_sys::i2s_channel_t_I2S_CHANNEL_MONO,
    total_chan: 0,
    skip_msk: false,
};

let result =
    unsafe { esp_idf_sys::i2s_driver_install(I2S_PORT_NUM, &config, 0, std::ptr::null_mut()) };
if result != esp_idf_sys::ESP_OK {
    panic!("error installing i2s driver");
}

let pin_config = esp_idf_sys::i2s_pin_config_t {
    mck_io_num: esp_idf_sys::I2S_PIN_NO_CHANGE, // unused
    bck_io_num: esp_idf_sys::gpio_num_t_GPIO_NUM_44,
    ws_io_num: esp_idf_sys::gpio_num_t_GPIO_NUM_43, // LR clock
    data_out_num: esp_idf_sys::gpio_num_t_GPIO_NUM_38,
    data_in_num: esp_idf_sys::I2S_PIN_NO_CHANGE,
};

let result = unsafe { esp_idf_sys::i2s_set_pin(I2S_PORT_NUM, &pin_config) };
if result != esp_idf_sys::ESP_OK {
    panic!("error setting i2s pins");
}

let mut backend = awedio_esp32::Esp32Backend::with_defaults(CHANNEL_COUNT, SAMPLE_RATE);
let manager = backend.start();
```

In order to get the `rmp3` native dependency to compile for xtensa chips
(if the mp3 feature is enabled) you may need to export the following variables
(adjust for your target):
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
