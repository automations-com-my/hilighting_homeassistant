# HiLighting LEDs Integration for Home Assistant

This repository provides integration support for various BLE RGB LED strips, including those commonly sourced from platforms like AliExpress and Lazada. Different manufacturers tend to use unique controllers, each with its own communication protocol. This project focuses on integrating these controllers seamlessly with Home Assistant, providing users with a unified interface for their LED strips.

## Supported Devices

We have recently added support for an additional BLE RGB LED strip found on Lazada. Initially, this strip wasn't recognized by the existing integration due to a different advertised Bluetooth name. After thorough investigation, we confirmed that the controller uses the same application and command set as previously supported devices, and we extended the integration to include this new variant.

### Lazada version:
- https://www.lazada.com.my/products/i3606344623-s21149872014.html
- Product screenshots shows `L7261` and `L7161`.
- Advertised Bluetooth name: `L7183`
- MAC address prefix: `23:11:11`

### AliExpress version:
- https://s.click.aliexpress.com/e/_DD74ZBn
- Advertised Bluetooth name: `L7161`
- MAC address prefix: `23:01:02`

Both versions are controlled using the `HiLighting` smartphone application, which may require enabling precise location services on certain Android versions to detect and control the LED strips.

## Features

The integration currently provides the following features for the supported LED strips:

- Turn On / Off
- Set RGB Colour
- Adjust Brightness
- Control Limited Effects with a Fixed Speed

The integration works by assuming the current state of the lights based on the last command sent. As a result, if the lights are altered externally (e.g., via the IR remote), the Home Assistant integration will not detect these changes, leading to a potential mismatch between the actual and assumed states.

## Installation in Home Assistant

### Requirements

Ensure that the Bluetooth component and [HACS](https://hacs.xyz/) (Home Assistant Community Store) are properly configured and operational within Home Assistant.

### Installation via HACS

Add this repository to HACS as a custom repository:

- Navigate to HACS -> Integrations -> Top right menu -> Custom Repositories
- Paste the GitHub URL of this repo into the Repository field
- Select `Integration` as the category
- Click Add
- Find the HiLighting integration and click Download to install
- Restart Home Assistant
- Your HiLighting LED devices should now appear in your Integrations page

## Sniffing the BLE Communication

Similar to [ELK-BLEDOB](https://github.com/8none1/elk-bledob) project, an nRF52840 BLE Sniffer was used to capture the communication between the controller and the application, helping to understand the protocol.

### Wireshark Tips

- Use `btle.length != 0` to filter out empty PDU packets
- Use `btatt.handle == 0x0014` to filter for writes to the serial port

## Command Reference

The controller operates via a Bluetooth LE serial port with the identifier `6e400002-b5a3-f393-e0a9-e50e24dcca9e`. This port is write-only, meaning it does not provide feedback or status updates after receiving commands. Therefore, communication with the controller is one-way.

Due to the write-only nature of the UART connection, any changes made directly through the controller (e.g., using an infrared remote control) will not be reflected in Home Assistant.

To test or manually execute commands, you can use an app like [LightBlue](https://punchthrough.com/lightblue/) to connect to the controller. Once connected, you can write the required command hex bytes directly to the serial port. This is particularly useful for debugging or manually controlling the lights outside of the Home Assistant environment.

### Power on & off

Using a BLE UART connection, write the following bytes to the serial port:

- `55 01 02 00` for Off
- `55 01 02 01` for On

### Set RGB Color

```
|------|------------------------ header
|      | ||--------------------- red
|      | || ||------------------ green
|      | || || ||--------------- blue
55 07 01 ff 00 00
55 07 01 00 ff 00
55 07 01 00 00 ff
```
### Adjust Brightness

```
|------|------------------------ header
|      | |---|------------------ brightness
55 03 01 09 03
55 03 01 6c 02
55 03 01 6f 05
```

Minimum brightness is 0x6c 0x02 (27650)
Maximum brightness is 0xff 0x0f (65295)

### Control Effects

The effects are numbered from 0 to 9, and the brightness cannot be adjusted for these effects.

```
|---|--------------------------- header
|   | ||------------------------ select effect
|   | || ||--------------------- effect number
55 04 01 00
55 04 01 01
55 04 01 06
55 04 01 07
```

### Effect Speed

```
|---|--------------------------- header
|   | ||------------------------ select effect speed (0-255)
|   | || ||--------------------- speed
55 04 04 31
55 04 04 59
55 04 04 96
55 04 04 bf
55 04 04 ff
```

### Custom Effects

```

|------| --------------------------------------------------------------------- custom effects header
|      | ||------------------------------------------------------------------- speed probably 0-255
|      | || ||---------------------------------------------------------------- effect type (merge, flash etc) probably 1 -> 5
|      | || || ||------------------------------------------------------------- brightness probably 0-255
|      | || || || |---------------| ------------------------------------------ likely all colour data. 2 bytes per colour? 565?
55 05 01 00 03 00 fa f8 55 f9 b1 ff       -  RGB   merge,  slow,   dim
55 05 01 00 03 ff fa f8 55 f9 b1 ff       -  RGB   merge,  slow,   bright
55 05 01 7f 03 00 fa f8 55 f9 b1 ff       -  rgb   merge,  fast,   dim
55 05 01 7f 03 ff fa f8 55 f9 b1 ff       -  rgb   merge,  fast,   bright
55 05 01 7f 04 ff fa f8 55 f9 b1 ff       -  rgb   flash,  fast,   bright
55 05 01 7f 05 ff fa f8 55 f9 b1 ff       -  rgb,  jump,   fast,   bright
55 05 01 7f 05 ff fa f8 41 f9 fc fd 4d e4 -  rgrg, jump,   fast,   bright

```

## Contributing

This project is continuously evolving to support new devices. If you encounter a similar LED strip that isn't recognized by this integration, feel free to contribute by following the steps outlined in this document or [reach out to us](https://automations.com.my/#contact) for Custom Integration Development Services.

## Possible Enhancements

- Custom Effect with Timing could be added
- Controller Firmware version and other information could be displayed on the detected devices
- Custom Firmware for the controller could be developed to provide bidirectional communications allowing status reads and updates in Home Assistant

## Other projects that might be of interest

- [iDotMatrix](https://github.com/8none1/idotmatrix)
- [Zengge LEDnet WF](https://github.com/8none1/zengge_lednetwf)
- [iDealLED](https://github.com/8none1/idealLED)
- [BJ_LED](https://github.com/8none1/bj_led)
- [ELK BLEDOB](https://github.com/8none1/elk-bledob)
- [BLELED LED Lamp](https://github.com/8none1/ledble-ledlamp)
