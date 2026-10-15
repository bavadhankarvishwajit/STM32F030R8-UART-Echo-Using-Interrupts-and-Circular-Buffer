# STM32F030R8 UART Echo Using Interrupts and Circular Buffer

A bare-metal embedded systems project demonstrating UART communication on the **STM32F030R8** microcontroller using register-level programming in STM32CubeIDE. Incoming UART data is received using interrupts, stored in a circular buffer, and echoed back to the terminal.

## Project Overview

This project demonstrates how to build a reliable UART receive pipeline without polling the receive register continuously.

### Key Features

- Register-level UART configuration for STM32F030R8
- USART2 serial communication
- Interrupt-driven UART reception
- Circular buffer implementation in C
- Producer-consumer architecture
- UART data echo using RealTerm
- Buffer overflow detection and error counters
- Modular code organization using header and source files

## Hardware Requirements

- STM32F030R8 microcontroller / development board
- USB-to-UART converter, if required
- USB cable for programming and debugging
- Computer running Windows
- Optional: breadboard and jumper wires

## Software Requirements

- STM32CubeIDE
- RealTerm Serial Capture Program
- ARM GCC toolchain (included with STM32CubeIDE)

## Hardware Connections

| STM32F030R8 | USB-to-UART Adapter |
|---|---|
| PA2 — USART2_TX | RX |
| PA3 — USART2_RX | TX |
| GND | GND |

**Important:** Connect TX to RX and RX to TX. Use compatible 3.3 V UART logic levels. Do not connect an RS-232 voltage-level interface directly to the STM32 pins.

## UART Configuration

| Parameter | Configuration |
|---|---|
| Peripheral | USART2 |
| TX pin | PA2 |
| RX pin | PA3 |
| Baud rate | 9600 |
| Data bits | 8 |
| Parity | None |
| Stop bits | 1 |
| Flow control | None |
| Reception method | Interrupt-driven |
| Buffer type | Circular buffer |

The baud-rate configuration assumes an 8 MHz USART2 peripheral clock. The clock configuration must match the actual board setup.

## Project Structure

```text
1_CAN_2-7_STM32_UART_Circular_Buffer_Echo/
├── Inc/
│   ├── main.h
│   ├── uart.h
│   ├── ring_buffer.h
│   ├── gpio.h
│   ├── events.h
│   ├── FreeRTOSConfig.h
│   └── system.h
├── Src/
│   ├── main.c
│   ├── uart.c
│   ├── ring_buffer.c
│   ├── gpio.c
│   ├── events.c
│   ├── hooks.c
│   ├── tasks.c
│   ├── syscalls.c
│   ├── sysmem.c
│   └── system.c
├── Startup/
├── FreeRTOS/
├── Debug/
├── STM32F030R8TX_FLASH.ld
└── README.md
```

The FreeRTOS files are present in the project structure. The UART echo test described here uses a main-loop consumer and does not start the FreeRTOS scheduler.

## How It Works

1. **Initialization:** The MCU configures GPIO alternate functions, enables USART2, and sets the UART baud rate.
2. **UART reception:** When a byte arrives, the USART2 interrupt handler reads the received data register.
3. **Circular buffer:** The interrupt handler stores the byte in the ring buffer.
4. **Main-loop processing:** The main loop removes available bytes from the buffer.
5. **Echo transmission:** Each consumed byte is transmitted back to the terminal.

### Data Flow

```text
Computer / RealTerm
        |
        | UART TX
        v
STM32 PA3 / USART2_RX
        |
        v
USART2 Receive Interrupt
        |
        v
Circular Buffer in RAM
        |
        v
Main Application Loop
        |
        v
USART2 Transmit / PA2
        |
        v
Computer / RealTerm
```

## Circular Buffer Concept

The circular buffer temporarily stores incoming data while the main application processes it.

- `head` identifies where the next byte will be inserted.
- `tail` identifies the next byte to be consumed.
- When an index reaches the buffer boundary, it wraps around to the beginning.
- One array position is reserved to distinguish a full buffer from an empty buffer.

For a 128-byte array, the usable capacity is 127 bytes.

## How to Build and Run

### 1. Import the project

Open STM32CubeIDE and import the existing project into your workspace.

### 2. Build the firmware

1. Select the project.
2. Choose **Project → Clean**.
3. Choose **Project → Build Project**.
4. Resolve any configuration errors before flashing.

### 3. Flash the STM32

Connect your board using a supported programmer/debugger and flash the generated firmware.

### 4. Configure RealTerm

Set the serial port to the COM port associated with your USB-to-UART adapter.

| Setting | Value |
|---|---|
| Baud | 9600 |
| Data bits | 8 |
| Parity | None |
| Stop bits | 1 |
| Flow control | None |

Open the port and enter text in the terminal's transmit interface.

## Expected Output

```text
STM32F030R8 UART Echo
USART2 - 9600 baud, 8-N-1
Type text in RealTerm:
```

When you transmit:

```text
Hello Vishwajit
```

The MCU echoes the received characters back to RealTerm.

## Error Handling

The firmware includes counters for monitoring receive problems:

- `uart_rx_overflow_count`: counts bytes rejected because the software circular buffer is full.
- `uart_rx_error_count`: counts detected UART hardware error conditions.

These counters can be inspected using the debugger. A nonzero overflow count indicates that the application is not consuming data quickly enough or the incoming data rate exceeds the buffer's capacity over the relevant interval.

## Learning Outcomes

After completing this project, you should understand:

- UART peripheral configuration at register level
- USART interrupts on STM32
- Interrupt service routines (ISRs)
- Circular buffer implementation
- Producer-consumer data processing
- Serial communication testing and debugging
- Buffer overflow and hardware receive error handling

## Future Improvements

- Add command-based UART parsing.
- Implement packet framing and message validation.
- Add CRC or checksum verification.
- Integrate UART processing into a FreeRTOS task.
- Connect the UART packet parser to CAN or Modbus communication modules.

## Author

**Vishwajit Mahesh Bavadhankar**

Embedded Systems | STM32 | UART | CAN | Modbus | IoT

## License

This project is intended for educational and embedded systems learning purposes. Add an appropriate open-source license if you plan to distribute or reuse the code publicly.
