# OBI Slave SPI Master
This repository provides a simple SPI master with an OBI interface. The **number of slaves** and the **output SPI frequency** is parametrizable.

# Parameters
| **Parameter name**         | **Type** | **Description**
| -------------------------- | -------- | ---------------
| `NUM_SLAVES`               | uint     | Number of slaves.
| `SCLK_COUNTER_RESET_VALUE` | uint     | Reset counter value.

# Ports
| **Signal Name** | **Type**         | **Direction**  |**Description**
| --------------- | --------         | -------------  | --------------
| `clk_i`         | logic            | *input*        | Primary input clock. The control interface is suposed to be synchronous to this clock.
| `rstn_i`        | logic            | *input*        | Synchronous active-low reset.
| `obi*`          | obi interface    | *input/output* | [OBI interface](https://github.com/pulp-platform/obi).
| `spi_ss_o`      | logic            | *output*       | SPI slave select.
| `spi_sclk_o`    | logic            | *output*       | SPI serial clock.
| `spi_mosi_o`    | logic            | *output*       | SPI master out slave in.
| `spi_miso_i`    | logic            | *input*        | SPI master in slave out.
| `complete_o`    | logic            | *output*       | Complete flag output.

# Register map

## `Tx` Register (offset 0x00, rw)
Set data to be transferred over SPI (1 byte).

|   7 : 0    |
| ---------- |
| data byte  |

## `Rx` Register (offset 0x04, read-only)
Read data transferred back from SPI (1 byte).

|   7 : 0    |
| ---------- |
| data byte  |

## `SpiDivClk` Register (offset 0x08, rw)
Set the value on which the module inverts serial clock output (half a period of output serial clock).

| 31 : 0      |
| ----------- |
| reset value |

## `Ss` Register (offset 0x0C, rw)
Set the selected slave, one at a time.

| `NUM_SLAVES-1` : 0 |
| ------------------ |
| slave select       |

Selecting the slave means setting the corresponding bit in this register.

## `Ctrl` Register (offset 0x10, read_only)

# Operation
The module counts to the reset value and then switches the output serial clock, which effectively splits the input clock by an integer ratio.

**Example:**

 The processor runs at 200 MHz and I want my SPI controller to run at 10 MHz. `SCLK_COUNTER_RESET_VALUE = 200 MHz / (10 MHz * 2) - 1 = 9`.