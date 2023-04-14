# Golioth Weather Demo

This is most bare-bones temperature reporting possible. It demonstrates the
ability to report temperature readings to Golioth using a variety of different
hardware without any device-specific C code.

## Setup

When building for a Nordic device, initialize using `west-ncs.yml`:

```
west init -m https://github.com/path/to/repository.git --mf west-ncs.yml
west update
```

For all other devices, initialize using `west-zephyr.yml`

```
west init -m https://github.com/path/to/repository.git --mf west-zephyr.yml
west update
```

Existing installs may change between the two options by editing the value for
`file` in `../.west/config`:

```yaml
[manifest]
path = app
file = west-zephyr.yml

[zephyr]
base = deps/zephyr
```

After changing this file, run `west update`

## Building

Copy `credentials.conf_example` to `credentials.conf` and update with your WiFi
(if needed) and Golioth credentials.

Build and flash the project:

```
# ESP32
west build -b esp32 .
west flash
```
