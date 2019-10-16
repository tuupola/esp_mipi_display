# MIPI DCS Display Driver

Low level driver for displays supporting the [MIPI Display Command Set](https://www.mipi.org/specifications/display-command-set). Currently tested with ST7735S, ST7789V and ILI9341.

[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE.md)

## Usage

Display and SPI parameters all the pins can be changed with `menuconfig`. You can find predefined values for some popular board in the [sdkconfig](https://github.com/tuupola/esp_mipi/tree/master/sdkconfig) folder. To use them copy one of the files to your project root before running menuconfig.

```
$ cp components/esp_mipi/sdkconfig/m5stack.defaults sdkconfig.defaults
$ make menuconfig
```

If you have already run `menuconfig` before, do a `defconfig` first.

```
$ cp components/esp_mipi/sdkconfig/m5stack.defaults sdkconfig.defaults
$ make defconfig
$ make menuconfig
```

Note that this is a low level driver. It provides only `init`, `write`, `ioctl` and `close` functions. It is meant to be used as a building block for a graphics library such as [copepod](https://github.com/tuupola/copepod). For basic usage see also [MIPI driver speedtest](https://github.com/tuupola/esp-examples/tree/master/016-mipi-speedtest).

```c
#include <driver/spi_master.h>
#include <stdlib.h>

#include "mipi_display.h"

spi_device_handle_t spi;
mipi_display_init(&spi);

/* Draw a random pixel on the screen. */
uint16_t color = rand() % 0xffff;
int16_t x0 = rand() % DISPLAY_WIDTH;
int16_t y0 = rand() % DISPLAY_HEIGHT;

mipi_display_write(spi, x0, y0, 1, 1, (uint8_t *) &color);

/* Draw a random 32x32 rectangle on the screen. */
uint16_t w = rand() % 32;
uint16_t h = rand() % 32;
int16_t x1 = rand() % (DISPLAY_HEIGHT - 32);
int16_t y1 = rand() % (DISPLAY_HEIGHT - 32);

uint8_t bitmap[32 * 32 * sizeof(uint16_t)];
memset16(bitmap, color, 32 * 32 * sizeof(uint16_t));

mipi_display_write(spi, x1, y1, w, h, bitmap)
```

You can also issue any command defined in [mipi_dcs.h](mipi_dcs.h).

```c
#include <driver/spi_master.h>
#include <stdlib.h>

#include "mipi_dcs.h"
#include "mipi_display.h"

spi_device_handle_t spi;
mipi_display_init(&spi);

/* Turn display inversion on. */
mipi_display_ioctl(spi, MIPI_DCS_ENTER_INVERT_MODE, 0, 0);

/* Read display address mode setting. */
uint8_t buffer[2];
mipi_display_ioctl(spi, MIPI_DCS_GET_ADDRESS_MODE, buffer, 2);
```

For clean shutdown

```c
mipi_display_close();
```

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.

Based on Espressif provided SPI Master example which was released to Public Domain: https://goo.gl/ksC2Ln
