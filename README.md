<p align="center">
  <img src="https://github.com/Kasperr169/homeassistant-app-bambuddy/blob/main/logo.png?raw=true" alt="Bambuddy Logo" width="300">
</p>

# Bambuddy App

[Bambuddy](https://github.com/maziggy/bambuddy) wrapped inside a Homeassistant App.

A self-hosted print archive and management system for Bambu Lab 3D printers.

![Supports aarch64 Architecture][aarch64-shield] ![Supports amd64 Architecture][amd64-shield] ![Supports armhf Architecture][armhf-shield] ![Supports armv7 Architecture][armv7-shield] ![Supports i386 Architecture][i386-shield]

[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[armhf-shield]: https://img.shields.io/badge/armhf-yes-green.svg
[armv7-shield]: https://img.shields.io/badge/armv7-yes-green.svg
[i386-shield]: https://img.shields.io/badge/i386-yes-green.svg

## Features

- 🖨️ Complete print archive and management for Bambu Lab printers
- 🎥 Real-time printer monitoring and camera streaming  
- 📊 Statistics, analytics, and failure detection
- 📤 Print queue management and auto-dispatch 
- 🖥️ **Virtual Printer Support** - Send prints directly from Bambu Studio/OrcaSlicer to Bambuddy
- 🔐 Secure local or remote access
- 🔄 Full Home Assistant integration

## Virtual Printer Support

This add-on includes optional support for Bambuddy's Virtual Printer feature, allowing you to:

- Send prints directly from Bambu Studio or OrcaSlicer to Bambuddy
- Create multiple virtual printers (e.g., Archive, Review Queue, Print Queue)
- Use Proxy Mode for remote printing to real printers
- No physical printer required for archiving and slicing

**⚠️ Important:** Enabling Virtual Printer adds 12+ network ports to the container. See [Virtual Printer Setup Guide](./VIRTUAL_PRINTER_SETUP.md) for details.

### Quick Start

1. Install and start Bambuddy
2. Go to Add-on settings → Configuration → Set `virtual_printer_enabled` to `true`
3. Follow the [Virtual Printer Setup Guide](./VIRTUAL_PRINTER_SETUP.md) to configure

For complete documentation: https://wiki.bambuddy.cool/features/virtual-printer/
