# Sensirion Raspberry Pi I²C SHT4X Driver

The repository provides a driver for setting up a sensor of the SHT4X family to run on a Raspberry Pi over I²C.

<img src="images/SHT4x.png" width="300px">

Click [here](https://sensirion.com/products/catalog/SEK-SHT40/) to learn more about the Sensirion SHT4X sensor family.



## Supported sensor types

| Sensor name   | I²C Addresses  |
| ------------- | -------------- |
|[SHT40](https://sensirion.com/products/catalog/SHT40/)| **0x44**, 0x45|
|[SHT41](https://sensirion.com/products/catalog/SHT41/)| **0x44**, 0x45|
|[SHT45](https://sensirion.com/products/catalog/SHT45/)| **0x44**, 0x45|

The following instructions and examples use a *SHT40*.



## Connect the sensor

Your sensor has 4 different connectors: SDA, GND, SCL, VDD.
Use the following pins to connect your SHT4X:

| *SHT4X* | *Cable Color*  |   *Raspberry Pi*   |
| :----------------: | -------------- | ------------------ |
| SDA | green | Pin 3
| GND | black | Pin 6
| SCL | yellow | Pin 5
| VDD | red | Pin 1


<img src="images/raspi-i2c-pinout-3.3V.png" width="400px">


### Detailed sensor pinout

Please refer to the datasheet for proper circuit setup. There are also 3rd party boards for easy connection of your SHT4x sensor to the Raspberry Pi.

<img src="images/SHT40_pinout.png" width="300px">

| *Pin* | *Cable Color* | *Name* | *Description*  | *Comments* |
|-------|---------------|:------:|----------------|------------|
| 1 | green | SDA | I2C: Serial data input / output |
| 2 | black | GND | Ground |
| 3 | yellow | SCL | I2C: Serial clock input |
| 4 | red | VDD | Supply Voltage | 1.1V to 3.6V



## Quick start example

- [Install the Raspberry Pi OS on to your Raspberry Pi](https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up)
- [Enable the I²C interface in the raspi-config](https://www.raspberrypi.org/documentation/configuration/raspi-config.md)
- Download the SHT4X driver from [Github](https://github.com/Sensirion/raspberry-pi-i2c-sht4x) and extract the `.zip` on your Raspberry Pi
- Connect the SHT4X sensor as explained in the [section above](#connect-the-sensor)
- The provided example is working with a SHT40, I²C address 0x44.
  In order to use the code with another product or I²C address you need to change it in the call sht4x_init(ADDRESS) in
  `sht4x_i2c_example_usage.c`. The list of supported I²C-addresses is found in the header
  `sht4x_i2c.h`.

- Compile the driver
    1. Open a [terminal](https://projects.raspberrypi.org/en/projects/raspberry-pi-using/8)
    2. Navigate to the driver directory. E.g. `cd ~/raspberry-pi-i2c-sht4x`
    3. Navigate to the subdirectory example-usage.
    4. Run the `make` command to compile the driver

       Output:
       ```
       rm -f sht4x_i2c_example_usage
       cc -Os -Wall -fstrict-aliasing -Wstrict-aliasing=1 -Wsign-conversion -fPIC -I. -o sht4x_i2c_example_usage  sht4x_i2c.h sht4x_i2c.c sensirion_i2c_hal.h sensirion_i2c.h sensirion_i2c.c \
           sensirion_i2c_hal.c sensirion_config.h sensirion_common.h sensirion_common.c sht4x_i2c_example_usage.c
       ```
- Test your connected sensor
    - Run `./sht4x_i2c_example_usage` in the same directory you used to
      compile the driver. You should see the measurement values in the console.

## Troubleshooting

### Building driver failed

If the execution of `make` in the compilation step 3 fails with something like

```bash
 make: command not found
```

your RaspberryPi likely does not have the build tools installed. Proceed as follows:

```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install build-essential
```


### Initialization failed

If you run `./sht4x_i2c_example_usage` but do not get sensor readings but something like this instead

```
Error executing soft_reset(): -1
Error executing serial_number(): -1
Error executing measure_lowest_precision_ticks(): -1
...
```
then go through the below troubleshooting steps.


-   Ensure that you connected the sensor correctly: All cables are fully
    plugged in and connected to the correct pin.
-   Ensure that I²C is enabled on the Raspberry Pi. For this redo the steps on
    "Enable the I²C interface in the raspi-config" in the guide above.
-   Ensure that your user account has read and write access to the I²C device.
    If it only works with user root (`sudo ./sht4x_i2c_example_usage`), it's
    typically due to wrong permission settings. See the next chapter how to solve this.

### Missing I²C permissions

If your user is missing access to the I²C interface you should first verfiy
the user belongs to the `i2c` group.

```
$ groups
users input some other groups etc
```
If `i2c` is missing in the list add the user and restart the Raspberry Pi.

```
$ sudo adduser your-user i2c
Adding user `your-user' to group `i2c' ...
Adding user your-user to group i2c
Done.
$ sudo reboot
```

If that did not help you can make globally accessible hardware interfaces
with a udev rule. Only do this if everything else failed and you are
reasoably confident you are the only one having access to your Pi.

Go into the `/etc/udev/rules.d` folder and add a new file named
`local.rules`.
```
$ cd /etc/udev/rules.d/
$ sudo touch local.rules
```
Then add a single line `ACTION=="add", KERNEL=="i2c-[0-1]*", MODE="0666"`
to the file with your favorite editor.
```
$ sudo vi local.rules
```

## Contributing

**Contributions are welcome!**

We develop and test this driver using our company internal tools (version
control, continuous integration, code review etc.) and automatically
synchronize the master branch with GitHub. But this doesn't mean that we don't
respond to issues or don't accept pull requests on GitHub. In fact, you're very
welcome to open issues or create pull requests :)

This Sensirion library uses
[`clang-format`](https://releases.llvm.org/download.html) to standardize the
formatting of all our `.c` and `.h` files. Make sure your contributions are
formatted accordingly:

The `-i` flag will apply the format changes to the files listed.

```bash
clang-format -i *.c *.h
```

Note that differences from this formatting will result in a failed build until
they are fixed.


## License

See [LICENSE](LICENSE).