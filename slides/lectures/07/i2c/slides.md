---
layout: section
---
# I2C
Inter-Integrated Circuit

---
---

# Bibliography
for this section

1. **Raspberry Pi Ltd**, *[RP2040 Datasheet](https://datasheets.raspberrypi.com/rp2040/rp2040-datasheet.pdf)*
   - Chapter 4 - *Peripherals*
     - Chapter 4.3 - *I2C*

2. **Paul Denisowski**, *[Understanding I2C](https://www.youtube.com/watch?v=CAvawEcxoPU)*

---
---
# I2C
a.k.a *I square C*

- Used for communication between integrated circuits
- Sensors usually expose an *SPI* and an *I2C* interface
- Two device types:
  - *controller* (master) - initiates the communication (usually MCU)
  - *target* (slave) - receive and transmit data when the *controller* requests (usually the sensor)


<div align="center">
<img src="/i2c/i2c_network.svg" class="rounded w-120">
</div>

---
---
# Wires & Addresses

- *SDA* - **S**erial **DA**ta line - carries data from the **controller** to the **target** or from the **target** to the **controller**
- *SCL* - **S**erial **CL**ock line - the clock signal generated by the **controller**, **targets** 
  - *sample* data when the clock is *low*
  - *write* data to the bus only when the clock is *high*
- each *target* has a unique address of **7 bits** or **10 bits**
- wires are never driven with `LOW` or `HIGH`
  - are always *pull-up*, which is `HIGH`
  - devices *pull down* the lines to *write* `LOW`

<div grid="~ cols-2 gap-5">

<div align="center">
<br>
<img src="/i2c/i2c_example.svg" class="rounded w-12f0">
</div>

<div align="center">
<img src="/i2c/i2c_network.svg" class="rounded w-120">
</div>

</div>

---
---
# Transmission Example
7 bit address

<div grid="~ cols-2 gap-5">

<v-clicks>

1. **controller** issues a `START` condition
    - pulls the `SDA` line `LOW`
    - waits for ~ 1/2 clock periods and starts the clock
2. **controller** sends the address of the **target**
3. **controller** sends the command bit (`R/W`)
4. **target** sends `ACK` / `NACK` to **controller**

</v-clicks>


<div>

<v-clicks>

5. **controller** or **target** sends data (depends on `R/W`)
    - receives `ACK` / `NACK` after every byte
6. **controller** issues a `STOP` condition 
   - stops the clock
   - pulls the `SDA` line `HIGH` while `CLK` is `HIGH`

</v-clicks>

Address Format

<div align="center">
<img src="/i2c/i2c_7bit_address_format.svg" class="rounded w-100">
</div>

</div>

</div>

Transmission

<div align="center">
<img src="/i2c/i2c_7bit_address_transmission.svg" class="rounded w-155">
</div>

**controller** writes each bit when `CLK` is `LOW`, **target** samples every bit when `CLK` is `HIGH`


---
---
# Transmission Example
10 bit address

<div grid="~ cols-2 gap-5">

<v-clicks>

1. **controller** issues a `START` condition
2. **controller** sends `11110` followed by the *upper address* of the **target**
3. **controller** sends the command bit (`R/W`)
4. **target** sends `ACK` / `NACK` to **controller**
5. **controller** sends the *lower address* of the **target**
6. **target** sends `ACK` / `NACK` to **controller**

</v-clicks>

<div>

<v-clicks>

7. **controller** or **target** sends data (depends on `R/W`)
    - receives `ACK` / `NACK` after every byte
8. **controller** issues a `STOP` condition 

</v-clicks>

Address Format
<div align="center">
<img src="/i2c/i2c_10bit_address_format.svg" class="rounded w-120">
</div>

</div>

</div>

Transmission

<div align="center">
<img src="/i2c/i2c_10bit_address_transmission.svg" class="rounded">
</div>

**controller** writes each bit when `CLK` is `LOW`, **target** samples every bit when `CLK` is `HIGH`

---
---
# I2C Modes
| Mode | Speed | Capacity | Drive | Direction | 
|-|-|-|-|-|
| Standard mode (Sm) | 100 kbit/s | 400 pF | Open drain | Bidirectional |
| Fast mode (Fm) |400 kbit/s | 400 pF | Open drain | Bidirectional |
| Fast mode plus (Fm+) | 1 Mbit/s | 550 pF 	| Open drain | Bidirectional |
| High-speed mode (Hs) | 1.7 Mbit/s | 400 pF | Open drain | Bidirectional |
| High-speed mode (Hs) | 3.4 Mbit/s | 100 pF | Open drain | Bidirectional |
| Ultra-fast mode (UFm) | 5 Mbit/s | ? | Push–pull | Unidirectional |

---
---
# Facts

| | | |
|-|-|-|
| Transmission | *half duplex* | data must be sent in one direction at one time |
| Clock | *synchronized* | the **controller** and **target** use the same clock, there is no need for clock synchronization |
| Wires | *SDA* / *SCL* | the same read and write wire and a clock wire |
| Devices | *1 controller* <br> *several targets* | a receiver and a transmitter |
| Speed | *5 Mbit/s* | usually 100 Kbit/s, 400 Kbit/s and 1 Mbit/s |

---
---
# Usage

- sensors
- small displays
- RP2040 has two I2C devices

<div align="center">
<img src="/i2c/raspberry_pi_pico_pins.jpg" class="rounded m-5 w-120">
</div>

---
---
# Embassy API
for RP2040, synchronous

<div grid="~ cols-3 gap-5">

```rust {lines: false}
pub struct Config {
    /// Frequency.
    pub frequency: u32,
}
```

```rust {lines: false}
pub enum ConfigError {
    /// Max i2c speed is 1MHz
    FrequencyTooHigh,
    ClockTooSlow,
    ClockTooFast,
}
```

```rust {lines: false}
pub enum Error {
    Abort(AbortReason),
    InvalidReadBufferLength,
    InvalidWriteBufferLength,
    AddressOutOfRange(u16),
    AddressReserved(u16),
}
```

</div>

```rust{all|1|3,4|6|8,9|11,12}
use embassy_rp::i2c::Config as I2cConfig;

let sda = p.PIN_14;
let scl = p.PIN_15;

let mut i2c = i2c::I2c::new_blocking(p.I2C1, scl, sda, I2cConfig::default());

let tx_buf = [0x90];
i2c.write(0x5e, &tx_buf).unwrap();

let mut rx_buf = [0x00u8; 7];
i2c.read(0x5e, &mut rx_buf).unwrap();
```

---
---
# Embassy API
for RP2040, asynchronous

```rust{all|1|3-5|7,8|10|12,13|15,16}
use embassy_rp::i2c::Config as I2cConfig;

bind_interrupts!(struct Irqs {
    I2C1_IRQ => InterruptHandler<I2C1>;
});

let sda = p.PIN_14;
let scl = p.PIN_15;

let mut i2c = i2c::I2c::new_async(p.I2C1, scl, sda, Irqs, I2cConfig::default());

let tx_buf = [0x90];
i2c.write(0x5e, &tx_buf).await.unwrap();

let mut rx_buf = [0x00u8; 7];
i2c.read(0x5e, &mut rx_buf).await.unwrap();
```