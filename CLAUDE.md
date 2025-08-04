# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is `jbdtool`, a Linux-based utility for interfacing with JBD (Jiabaida) Battery Management Systems (BMS). The tool communicates with BMS units through multiple transport protocols to read/write parameters, monitor battery status, and manage battery protection settings.

## Build System

The project uses a Makefile-based build system with configurable features:

```bash
# Basic build
make

# Debug build (default enabled)
make DEBUG=yes

# Enable optional features
make BLUETOOTH=yes MQTT=yes

# Cross-compilation targets
make TARGET=win32    # Windows 32-bit
make TARGET=win64    # Windows 64-bit  
make TARGET=pi       # Raspberry Pi ARM

# macOS build (disable Linux-specific features)
make MQTT=no BLUETOOTH=no    # Minimal build for macOS

# Static linking
make STATIC=yes

# Install to system
make install

# Debug with GDB
make debug

# Clean build artifacts
make clean
```

## Architecture

### Core Components

- **jbd.c/jbd.h**: Core JBD BMS protocol implementation and session management
- **jbd_info.c/jbd_info.h**: BMS information structures and parameter definitions
- **main.c**: Command-line interface and application entry point
- **cfg.c/cfg.h**: Configuration file parsing and management

### Transport Layer
The tool supports multiple transport protocols through a modular design:

- **serial.c**: RS232/USB serial communication
- **bt.c**: Bluetooth Low Energy (requires glib-2.0, gio-2.0)
- **can.c**: CAN bus communication
- **ip.c**: TCP/IP and ESP-link support
- **mqtt.c**: MQTT messaging (requires paho.mqtt.c library)

### Utility Modules

- **parson.c/parson.h**: JSON parsing library for configuration and output
- **list.c/list.h**: Generic linked list implementation
- **utils.c/utils.h**: Common utility functions
- **module.c/module.h**: Dynamic module loading system

## Usage Patterns

### Transport Specification
All operations require specifying a transport using `-t transport:target,options`:

```bash
# Serial communication
jbdtool -t serial:/dev/ttyS0,9600

# Bluetooth (MAC address + descriptor)
jbdtool -t bt:01:02:03:04:05:06,ff01

# CAN bus
jbdtool -t can:can0,500000

# TCP/IP
jbdtool -t ip:10.0.0.1,23

# CAN-over-IP
jbdtool -t can_ip:10.0.0.1,3930,can0,500000
```

### Common Operations

```bash
# Read all parameters
jbdtool -t [transport] -r -a

# Read specific parameters
jbdtool -t [transport] -r BalanceStartVoltage BalanceWindow

# Write parameters
jbdtool -t [transport] -w BalanceStartVoltage 4050 BalanceWindow 20

# Output to JSON file
jbdtool -t [transport] -j -o output.json

# List supported parameters
jbdtool -l
```

## Dependencies

### Required
- Standard C library and POSIX-compliant system
- pthread library

### Optional Features
- **MQTT**: paho.mqtt.c library (https://github.com/eclipse/paho.mqtt.c)
- **Bluetooth**: glib-2.0 and gio-2.0 development packages
- **Cross-compilation**: MinGW for Windows targets, ARM toolchain for Pi

## Development Notes

- The codebase uses a modular transport abstraction allowing easy addition of new communication methods
- Configuration is handled through both command-line parameters and configuration files
- JSON output format is supported throughout for integration with other tools
- Debug builds include extensive logging controlled by the `debug` global variable
- The BMS protocol implementation handles packet framing, checksums, and error recovery
- Parameter definitions are centralized in jbd_info.c for easy maintenance

## Platform Support

### macOS
- CAN bus support is disabled (Linux-specific)
- Some high-speed serial baud rates are not supported (460800, 500000, 576000, 921600)
- MQTT and Bluetooth require additional libraries - use `make MQTT=no BLUETOOTH=no` for basic build
- Serial and IP transports work correctly

### Linux
- Full feature support including CAN bus, Bluetooth, and MQTT
- All transport protocols supported

### Windows  
- Cross-compilation supported via MinGW
- CAN bus support disabled
- Special library linking required for MQTT

## Important Constraints

- CAN bus transport cannot read/write parameters (read-only for basic info)
- CAN bus is only available on Linux systems
- macOS builds exclude CAN and may have limited baud rate support
- Windows builds require specific library linking for MQTT and networking
- Static builds on some platforms require additional system libraries