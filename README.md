# Golioth Weather Demo

This repository demonstrates a very bare-bones temperature reporting
application. It highlights how to use [Zephyr
RTOS](https://www.zephyrproject.org/) to apply a single firmware project to
multiple different architectures and sensors without the need to specialize the
C code.

![Several chip architectures running the same Zephyr code](img/many_weather_devices_microcontrollers_scaled.jpg)

You can build this project for the Nordic nRF9160, NXP i.MX1062, or ESP32 and
your choice of an Infineon DPS310 or Bosch BME280 temperature sensor. The result
is a fleet of diverse hardware all sending temperature data to Golioth.

This demo was put together for the *Building a modular codebase with Zephyr RTOS
and Devicetree* talk during the [2023 Embedded Online
Conference](https://embeddedonlineconference.com/).

**Note:** for brevity, some Zephyr best practices have not been used. Please see
the section at the bottom of this readme for more on this topic.

## Golioth

[Golioth](https://golioth.io/) is an instant IoT cloud for firmware devices.
With Golioth, your project will have OTA, Time Series and Stateful data handling,
command and control, remote logging, and many other useful services from day
one.

[Try Golioth now](https://console.golioth.io/), your first 50 devices are free
with our Dev Tier.

## Setup

Do not clone this repo. Instead, use `west init` to set up a workspace with the
correct dependencies.

### Initial Setup

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

### Changing between Zephyr and the nRF Connect SDK (NCS)

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

Build and flash the project:

```
# SparkFun Thing Plus nRF9160 (uses west-ncs.yml)
west build -b sparkfun_thing_plus_nrf9160_ns .
west flash

# NXP i.MX RT1060 EVKB (uses west-zephyr.yml)
west build -b mimxrt1060_evkb .
west flash

# ESP32 (uses west-zephyr.yml)
west build -b esp32 .
west flash
```

## Save Device Credentials to Non-Volatile Storage

Each device needs to be assigned its own PSK-ID and PSK to authenticate with
Golioth. Devices that use WiFi also need an SSID and PSK. These are assigned
(once) via the serial console after the firmware has been flashed and will
persist across future firmware upgrades.

```
uart:~$ settings set wifi/ssid <my-wifi-ap-ssid>
uart:~$ settings set wifi/psk <my-wifi-ap-password>
uart:~$ settings set golioth/psk-id <my-psk-id@my-project>
uart:~$ settings set golioth/psk <my-psk>
uart:~$ kernel reboot cold
```

## A note on Zephyr best practices

The `main.c` file in this project has been condensed to the bare-minimum for
readability. However, this removed several best practices that should be learned
and used on production projects.

### Checking return codes

It is highly recommended that you test function return codes against [the Zephyr
error numbers
list](https://docs.zephyrproject.org/apidoc/latest/group__system__errno.html).
For instance, checking Golioth API call return codes is well demonstrated in
this snippet from [our LightDB Stream
sample](https://github.com/golioth/golioth-zephyr-sdk/blob/main/samples/lightdb_stream/src/main.c)
application:

```c
int err;

err = golioth_stream_push_cb(client, "temp",
			     GOLIOTH_CONTENT_FORMAT_APP_JSON,
			     sbuf, strlen(sbuf),
			     temperature_push_handler, NULL);

if (err) {
	LOG_WRN("Failed to push temperature: %d", err);
	return;
}
```

### Checking that a device is ready

When getting a device such as a sensor from Devicetree, it is recommended that
you use `device_is_ready()` to verify. This is well demonstrated in this
function from [the Zephyr BME280 sample
code](https://github.com/zephyrproject-rtos/zephyr/blob/main/samples/sensor/bme280/src/main.c):

```c
static const struct device *get_bme280_device(void)
{
	const struct device *const dev = DEVICE_DT_GET_ANY(bosch_bme280);

	if (dev == NULL) {
		/* No such node, or the node does not have status "okay". */
		printk("\nError: no device found.\n");
		return NULL;
	}

	if (!device_is_ready(dev)) {
		printk("\nError: Device \"%s\" is not ready; "
		       "check the driver initialization logs for errors.\n",
		       dev->name);
		return NULL;
	}

	printk("Found device \"%s\", getting sensor data\n", dev->name);
	return dev;
}
```
