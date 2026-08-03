# OBI Slave SPI Master
This repository provides a simple SPI master with an OBI interface. The **number of slaves** is parametrizable and the **output SPI frequency** is configurable.

# Parameters
| **Parameter name**         | **Type** | **Description**
| -------------------------- | -------- | ---------------
| `NUM_SLAVES`               | uint     | Number of slaves.

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
Set data to be transferred over SPI.

|   7 : 0    |
| ---------- |
| data byte  |

## `Rx` Register (offset 0x04, read-only)
Read data transferred back from SPI.

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

## `Ctrl` Register (offset 0x10, rw)
Controls the operation of the module.

| 3             | 2             | 1             | 0             |
| ------------- | ------------- | ------------- | ------------- |
| complete      |          busy | start_reading | start_writing |

Setting the **start_writing** bit starts an SPI transaction, with the contents of `Tx` Register being sent.

Setting the **start_reading** bit starts an SPI transaction, receiving data on SPI and storing it to `Rx` Register.

**Busy** bit is set whenever a read or write transaction is in progress.

**Complete** bit is set when an SPI transaction completes and **needs to be cleared** before initiating another SPI transaction. Output port `complete_o` is synonymous with this bit.

# Operation
This module counts to the set reset value and then switches the output serial clock, which effectively splits the input clock frequency by an integer ratio.

**Example:**

I want to send 0x20 over SPI to the second slave, where the processor speed is 200 MHz and SPI runs at 10 MHz.

Then I want to read from the same slave.

`SCLK_COUNTER_RESET_VALUE = 200 MHz / (10 MHz * 2) - 1 = 9`.
 
 0) Set `NUM_SLAVES = 2` (to number of slaves you are using) when instantiating this module.
 1) Set `SCLK_COUNTER_RESET_VALUE = 9` (to the calculated value).
 2) Set `Tx = 0x20` register (if sending over SPI).
 3) Set `Ss = 0x2` register (to select the slave you are sending/reading to/from).
 4) Set `start_writing` in `Ctrl` register.
 5) Wait for `complete` bit in `Ctrl` register to set.
 6) Clear `complete` bit in `Ctrl` register.
 7) Set `start_reading` bit in `Ctrl` register.
 8) Wait for `complete` bit in `Ctrl` register to set.
 9) Read from `Rx` register.
 10) Clear `complete` bit in `Ctrl` register.
 11) Clear `Ss = 0x0` register to unselect the slave.
