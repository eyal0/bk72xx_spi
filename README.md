# BK72xx SPI for ESPHome

Hardware SPI bus for the Beken BK7238 and BK7231N. ESPHome's `spi:` component does not support LibreTiny, and the Beken SDK leaves its SPI driver unlinked, so this component programs the controller directly.

There is one controller. SCK, MOSI, and MISO are fixed. Each device supplies its own GPIO chip-select, clock, and SPI mode.

| Signal | Pin |
| --- | --- |
| SCK | P14 |
| MOSI | P16 |
| MISO | P17 |
| CS | GPIO chosen by the device |

The clock is 13 MHz divided by 2 through 255, about 51 kHz to 6.5 MHz, and it never runs faster than the device requests. BK7231T uses a different SPI register map and is rejected.

## Use the bus

```yaml
external_components:
  - source: github://eyal0/bk72xx_spi@main
    components: [bk72xx_spi]

bk72xx_spi:
  id: spi_bus
```

A second device on the same controller uses the same `spi_id` and a different `cs_pin`. List this source before a component that depends on it. ESPHome caches the clone for a day; set `refresh: 0s` on that entry to pick up a push immediately.

## Write a device

Depend on `bk72xx_spi`, take its id with `spi_id`, and call `transfer()` with that device's chip-select, clock, and mode. Mode is SPI mode 0 through 3. A null transmit buffer clocks out zeros, and a null receive buffer discards the incoming bytes. A transfer is 1 to 4095 bytes.

```cpp
this->parent_->transfer(this->cs_pin_, frequency_hz, mode, tx, rx, length);
```

The [HLW8112 meter](https://github.com/eyal0/hlw8112_spi) is one device on this bus.
