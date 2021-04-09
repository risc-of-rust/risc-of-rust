# Sipeed Longan Nano

## Examples

### I2C

```rust
let pb = peripherals.GPIOB.split(&mut rcu);

    let mut pb6_scl = pb
        .pb6
        .into_open_drain_output_with_state(hal::gpio::State::High);
    let pb7_sda = pb
        .pb7
        .into_open_drain_output_with_state(hal::gpio::State::High);
    // Attempt to unstuck the i2c bus.
    for _ in 0..16 {
        delay.delay_us(5);
        let _ = pb6_scl.set_low();
        delay.delay_us(5);
        let _ = pb6_scl.set_high();
    }
    let pb6_scl = pb6_scl.into_alternate_open_drain();
    let pb7_sda = pb7_sda.into_alternate_open_drain();

    // Set up i2c.
    let mut i2c0 = hal::i2c::BlockingI2c::i2c0(
        peripherals.I2C0,
        (pb6_scl, pb7_sda),
        &mut afio,
        hal::i2c::Mode::Fast {
            frequency: 400_000.hz(),
            duty_cycle: hal::i2c::DutyCycle::Ratio2to1,
        },
        &mut rcu,
        1000,
        10,
        1000,
        1000,
    );

    // Write register 0x20: 0b1010 - set to open drain active low for INT1 and INT2 to prevent blocking JTAG operation
    const BMA223_ADDR: u8 = 0x18;
    const BMO223_CHIP_ID: u8 = 0b11111000;
    const BMA223_REG_BGW_CHIPID: u8 = 0x00;
    const BMA223_REG_INT_OUT_CTRL: u8 = 0x20;
    const BMA223_REG_ACCD_X_LSB: u8 = 0x02;

    // This frees up the JTAG pins (see `notes/01-JTAG.md`).
    let _ = nb::block!(i2c0.write(BMA223_ADDR, &[BMA223_REG_INT_OUT_CTRL, 0b1010])).unwrap_or_else(
        |e| {
            write!(
                uart1_tx,
                "Error writing INT_OUT_CTRL to BMA223: {:?}\r\n",
                e
            )
            .unwrap()
        },
    );

    {
        let mut read = [0; 1];
        match nb::block!(i2c0.write_read(BMA223_ADDR, &[BMA223_REG_BGW_CHIPID], &mut read)) {
            Ok(()) => write!(uart1_tx, "Read BMA223 chip id: {:#010b}\r\n", read[0]).unwrap(),
            Err(e) => write!(
                uart1_tx,
                "Error writing to and reading from BMA223: {:?}\r\n",
                e
            )
            .unwrap(),
        }
    }
```
> from (https://github.com/alvinhochun/gd32vf103-pinecil-demo-rs) under MIT

## Links

(https://pramode.net/2019/10/07/rust-on-riscv-board-sipeed-longan-nano/)

(https://github.com/alvinhochun/gd32vf103-pinecil-demo-rs/blob/master/07-bma223/src/main.rs)

(https://wiki.segger.com/SiPeed_Longan_Nano)

# BL602

## Examples

### I2C

```rust

```


## Links

[https://github.com/sipeed/bl602-rust-guide](https://github.com/sipeed/bl602-rust-guide)

[https://github.com/sipeed/bl602-hal](https://github.com/sipeed/bl602-hal)
